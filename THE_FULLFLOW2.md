# The Full Flow

A step-by-step account of how this project works, in simple words. Each section was checked against the code before it was written down.

---

## Intro

This project is a Layer 4 TCP reverse proxy. There is no HTTP router on the public port. There is no cluster of workers. One Linux process, one thread, one event loop.

Clients talk to one address. That process picks a backend, opens a second TCP connection, and copies bytes both ways. It does not read URLs, cookies, or JSON in that traffic. A backend that speaks HTTP, TLS, or postgres gets the same treatment.

The aim is this: if a client sends bytes, those same bytes reach a backend, and the backend’s bytes come back, without this process blocking on any one guest, leaking a file descriptor, or drowning in memory when someone reads slowly.

---

## The kitchens start first

The demo has three dummy backends. They are ordinary Python HTTP servers, not part of the C++ program. Someone starts them before flowlb:

`scripts/backends.sh start`

That launches **b1** on `127.0.0.1:9001`, **b2** on `:9002`, **b3** on `:9003`. Each one answers `GET /` with a line like `backend=b1 port=9001`. They speak HTTP/1.0 on purpose: one TCP connection, one request, then close. That makes round-robin visible. Keep-alive would hide it.

They are already listening. flowlb has not started. Nobody has been load-balanced yet.

---

## You start flowlb

You type:

`./build/release/flowlb config/flowlb.json`

Or you omit the path, and it uses that same file.

`main` is thin. First it ignores `SIGPIPE` process-wide. A write to a reset peer must return `EPIPE`, not kill the process. Then it starts the logger.

Then it loads the config. Missing file, unknown key, empty backend list, `listen` equal to `stats_port`, a host it cannot resolve: print a sentence, exit 1. It does not start half-configured.

The file says: listen on **8080**, stats on **8081**, algorithm **round-robin**, log level **info**, three backends b1/b2/b3, health every 1000 ms, probe timeout 500 ms, three fails to mark down, two successes to mark up, type **tcp**.

Each backend `host` is turned into an IPv4 address **now**. `127.0.0.1` is a number, not a lookup. If it were a hostname, `getaddrinfo` runs here, once. The later loop will never call DNS.

`FLOWLB_LOG_LEVEL` wins over the file if it is set. Load tests use `warn` so per-connection INFO is not part of the number.

Then it prints pid, fd limit, listen port, algorithm, how many backends. Then `Proxy::open`. If bind fails (port in use), that is a startup exception, not an I/O `expected`. The process is not a server yet.

---

## Building the waiter

`Proxy` is the shared object. It is not a thread. Inside it, in construction order:

- **EventLoop** — `epoll_create1`. This is the bell board. It also blocks `SIGINT`/`SIGTERM` and turns them into a `signalfd`, registered level-triggered. Ctrl-C will be a readable fd, not a surprise in the middle of `read`.
- **BackendPool** — three `Backend` rows from the config: id, host, port, the already-resolved address. Every one starts **healthy**. `active` is 0. The round-robin cursor is 0. Nobody has been picked yet.
- **Stats** — counters at zero, a monotonic start time.
- **Listener** — not built in the constructor. `open` binds it next.
- **StatsServer**, **HealthChecker**, the connection map — also empty until `open` finishes.

`open` binds the public door first. `Listener::bind`:

A TCP socket, already non-blocking and `CLOEXEC`. `SO_REUSEADDR` so a restart can take 8080 even if TIME_WAIT from the last run is still sitting there. Bind `0.0.0.0:8080`, listen with backlog 1024. Open a spare fd to `/dev/null`. That spare is for later, if the process runs out of fds. Then the listen fd goes onto the epoll board as **level-triggered** `EPOLLIN`. Not edge-triggered. The callback is `Proxy::on_accept`.

Then `StatsServer::open` on 8081. A second `Listener`, same kind of bind, same spare-fd trick, same LT accept. A second `timerfd` that will fire every 5 seconds for a human-readable summary line. That timer is **not** the health timer. Two clocks, two jobs.

Then `HealthChecker` is constructed in place. `timerfd_create(CLOCK_MONOTONIC)`. First fire in 1 millisecond. After that, the interval on the fd is `timeout_ms` (500 ms in this config), so a stuck probe can be expired on the same fd that launches new ones. `interval_ms` (1000) still gates how often a **new** probe round is due. The timer fd goes on the same epoll, **level-triggered**. One empty probe slot per backend.

`run()` logs `listening port=8080 ... backends=b1:127.0.0.1:9001,...` and then the thread sits in `epoll_wait` forever (timeout `-1`).

That is all you can do by yourself. Traffic needs a client. Health will poke the kitchens on its own.

---

## The idle wait

There is one thread. It is not “the Proxy running as a thread.” The Proxy is the object. The thread is asleep in `epoll_wait` until some fd is interesting.

Right now those fds include:

- The **epoll** fd itself (the board).
- The **signalfd** (Ctrl-C).
- The **listen** socket on 8080.
- The **stats listen** socket on 8081.
- The **health timerfd**.
- The **stats timerfd**.
- Two spare `/dev/null` fds, one per listener.

No client sockets. No backend sockets. The connection map is empty. `conns_` is a hash map of id → Connection. The id is a counter starting at 1, not an fd. Fds get reused by the kernel. Ids do not, in any lifetime this program cares about.

This is not several goroutines. The poker engine could listen on two gossip rooms and a TCP port as separate workers. Here, one worker. When the kernel wakes it, `poll_once` gets up to 64 ready fds, looks each up in a phone book (`fd → Handler*`), calls `on_event`, then runs deferred destructors. Then it sleeps again.

The phone book does not own the handlers. `Connection` lives in `Proxy::conns_`. Listener, HealthChecker, StatsServer live on Proxy. The map is “who do I call,” not “who do I delete.”

Until Alice connects, the only work that will happen is health ticks and, every 5 seconds, one INFO summary line.

---

## The first probe

About a millisecond after `run()`, the health timer is readable. Level-triggered, so it stays readable until the 8-byte expiration count is `read`. Missed ticks still mean **one** probe round, not a catch-up storm.

`on_tick`: expire any in-flight probe older than 500 ms (none yet). Then, because `next_probe_due_` was set to “now” at construction, start a probe for every backend that has no probe in flight. That is all three.

For each: a non-blocking TCP socket, `connect` to that backend’s already-resolved address. On loopback this often returns **connected immediately**. Type is `tcp`, so success is “the handshake finished.” No HTTP GET. `record_probe(ok)`: `consecutive_ok` goes to 1, fail streak to 0. Already healthy, so no “backend up” log. The probe fd is not kept. The slot is empty again.

If connect had returned `EINPROGRESS`, the probe fd would sit on epoll as edge-triggered `EPOLLOUT`. When writable, `getsockopt(SO_ERROR)`: 0 means connected, anything else is a fail. Same helpers the real connections use.

Then `next_probe_due_` is now plus 1000 ms. The timer will still wake every 500 ms so a black-hole backend (SYN accepted, never answered) can time out between rounds. A new connect is not launched until the interval says so, and not if a probe is already in flight for that slot. At most one probe per backend at a time.

b1, b2, and b3 are up. The pool still says healthy, healthy, healthy. Alice has not arrived.

---

## Alice’s request

Alice types:

`curl -sS http://127.0.0.1:8080/`

Her machine does a TCP handshake with **8080**. The kernel queues the completed handshake. The listen fd becomes ready. `epoll_wait` returns. `Listener::on_event` runs.

It `accept4`s in a loop, `SOCK_NONBLOCK | SOCK_CLOEXEC`, up to **64** times this tick. One SYN flood must not starve existing connections. The listen fd is level-triggered on purpose: if it stopped at 64 with more still queued, the next `epoll_wait` will say “still readable.” Edge-triggered would have consumed the edge and left the backlog silent until a *new* SYN after the queue went idle.

One accept succeeds. New client fd, Alice’s address. That fd is handed to `Proxy::on_accept`.

`BackendPool::pick()`. Algorithm is round-robin. Cursor is 0. b1 is healthy. Pick is index 0, address `127.0.0.1:9001`. Cursor advances to 1.

If every backend were down, pick would be empty. `rejected++`, log `reason=no_healthy_backend`, return. The `Fd` destructor closes Alice immediately. No hang. No queue in userspace.

Alice is not empty. `next_id_` is 1, then becomes 2. Log `conn=1 accept`. `acquire(0)`: b1’s `active` is 1, `total_acquired` is 1. `accepted++`. Log `pick backend=b1 ... algo=round_robin active=1`. Then construct a `Connection` and store it in the map under id 1.

`active` moved in exactly one place. It will decrement in exactly one place, when this connection is destroyed. Least-connections will read that same field later. Stats will too. There is no shadow counter.

---

## Connecting to b1

The Connection constructor owns Alice’s fd now. `TCP_NODELAY` on it, so tiny HTTP writes are not delayed by Nagle.

It makes a second non-blocking TCP socket and `connect`s to b1. Two outcomes:

- **0** — already connected (common on loopback). Jump straight to proxying.
- **`EINPROGRESS`** — handshake in flight. Register the backend fd edge-triggered for `EPOLLOUT | EPOLLRDHUP`. State stays `CONNECTING`. Do not assume success when it becomes writable. `complete_connect` reads `SO_ERROR`. Zero means connected. Non-zero means b1 refused. Then close the pair.

Until connected, the object does not treat Alice as a byte pump. When it *does* enter `PROXYING`, it sets `need_speculative_client_`. Alice may already have sent `GET / ...` while the handshake was in flight. Restoring or adding `EPOLLIN` under edge-triggered **does not** invent an edge for data that is already sitting in the kernel. If we only waited for the next `EPOLLIN`, and she had finished sending, we would stall forever. So we drain her immediately.

`TCP_NODELAY` goes on the backend fd too. State is `PROXYING`. Log `conn=1 connected`. Two buffers are still empty: **c2b** (client-to-backend) and **b2c** (backend-to-client).

This is two TCP connections, not one. Alice talks to flowlb. flowlb talks to b1. Bytes will be copied in userspace. This is a proxy, not NAT.

---

## Copying bytes

`finish_io` speculative-drains Alice. `drain_read` on the client fd:

Read up to 64 KiB into a stack chunk. Loop until `EAGAIN`, EOF, a real error, or the destination buffer hits the high watermark. Each success appends to **c2b** and adds to `bytes_c2b`. Then `flush_write` toward b1: `write` until c2b is empty or `WouldBlock`. Leftover stays in the buffer. `EPOLLOUT` stays armed until empty.

Alice’s bytes are HTTP. This program does not parse them. It copies them. b1’s Python server reads a request, thinks a client spoke HTTP, writes a response. Those bytes arrive on the backend fd. `drain_read` from b1 appends **b2c**, counts `bytes_b2c`, `flush_write` to Alice.

`WouldBlock` is not an error. It is “stop looping, wait for epoll.” `Eof` is `read` returned 0: that side will send no more. A real errno closes the pair.

`apply_interest` turns state + buffer fullness + pause flags into the exact epoll mask, separately for each fd. Empty c2b → do not ask for `EPOLLOUT` on the backend. Asking for events you do not want is how you burn CPU. It caches the last mask and skips `epoll_ctl` if nothing changed.

Both connection sockets are **edge-triggered**. Every read and write loops until `EAGAIN`. If you read 4 KiB of a 64 KiB burst and go back to sleep, there is no second doorbell until new data arrives.

---

## Alice is done sending

curl wrote the request and shut its write side. That is a TCP FIN, not a process exit. `read` on Alice’s fd returns 0.

`on_read_eof(Client)`: `client_read_eof_ = true`, state `HALF_CLOSED_CLIENT`, `pending_shutdown_backend_ = true`. Flush leftover c2b to b1, then `shutdown(SHUT_WR)` on the **backend** fd. That tells b1 “we will not send more.” It does **not** `close` Alice. Closing would tear down receive as well, and b1’s response would be thrown away.

HTTP/1.0 clients do this all day: send, shut write, read until EOF.

b1 finishes the body and FINs. `read` on the backend returns 0. `backend_read_eof_`, maybe `HALF_CLOSED_BACKEND` if we were still in `PROXYING`. Flush leftover b2c to Alice, `shutdown_wr` on the client if needed. When both sides have EOFed and both buffers are empty, `close_now`.

`EPOLLRDHUP` is in the mask so a peer FIN is visible even when we are mostly waiting to write.

---

## The pair closes

`close_now` is allowed to run inside `on_event`. Destroying the Connection is not.

It logs `conn=1 close state=... bytes_c2b=... bytes_b2c=...`, sets state `CLOSING`, **unregisters** both fds from epoll, and queues `on_done` with `EventLoop::defer`. If the same `epoll_wait` batch still has an event for the other fd of this pair, `on_event` sees `CLOSING` and returns. The object still exists. That is the use-after-free this class of program is famous for, and deferred reclaim is how it is avoided.

After every event in the batch has been dispatched, `drain_deferred` runs. `Proxy::on_connection_done`: add those byte counts to the **closed** totals (so `/stats` does not lose them), `release(0)` so b1’s `active` is 0, erase id 1 from the map. Erase destroys the `unique_ptr`, which destroys the Connection, which destroys the two `Fd`s, which `close`. Never retry `close` on `EINTR` on Linux: the number is already gone, and retrying can close a stranger.

If the listen fd had been paused for `EMFILE`, this is also when `resume_accept` may run: an fd was reclaimed, so there is a slot again.

Alice got `backend=b1 port=9001`. The waiter did not cook. It carried a plate.

---

## Bob and Carol

Bob curls the same URL. Same accept path. `pick()`: cursor is 1, b2 is healthy, he gets b2, cursor becomes 2.

Carol curls. Cursor 2, b3, cursor wraps to 0.

Thirty sequential `GET /` in the measurements: 11 / 10 / 10. The extra is an identity curl the gate does first. Sequence in the log is `b1, b2, b3` repeating. That is round-robin working.

They still skip unhealthy backends. The cursor still advances past a down slot so it does not stick. They still fail closed: nothing healthy → close the client now.

Least-connections is the other algorithm, same `pick()`, same `acquire`/`release`. It walks from the cursor, takes the healthy backend with smallest `active`, then sets the cursor just after the winner. An idle pool then looks round-robin-shaped. That is on purpose. A same-tick burst against an idle pool also looks round-robin-shaped, because no `release` has run yet — `pick` only sees what `acquire` already did. Latency that has not happened yet cannot change the count.

The dummy backends are HTTP/1.0, so one TCP pairing is one request, and `total_acquired` in `/stats` is also “requests routed” for the demo. In a keep-alive world those numbers diverge. The count is still connections, because connections are what occupy backend threads and RAM.

---

## The operator checks /stats

Someone — not Alice — types:

`curl -sS --http1.0 http://127.0.0.1:8081/stats`

That hits **8081**, not 8080. A different listen fd, a different `Listener`, a tiny HTTP speaker that is **not** the byte pump.

`StatsServer::on_accept`: at most 8 sessions. A ninth gets a 503 and the fd is parked for deferred close. Cap 8 KiB of request. Five-second expiry on the stats timer, so a scraper cannot sit forever.

The session fd is edge-triggered. Read until `\r\n\r\n` or `\n\n`. Look at the first line. `GET /stats` → call the render callback, which walks live connections for in-flight bytes, adds closed totals, reads `active` from the **same** pool fields `pick` uses, and writes compact JSON: integers, bools, strings, `null`. No floats, so the config parser we wrote can round-trip it. Anything else: 404. Then write, close.

This is the one HTTP parser in the process. It is a side door with a clipboard. It does not peek at Alice’s traffic. Port 8080 still does not know what HTTP is.

Every 5 seconds the stats `timerfd` also prints one INFO line for a human watching the terminal. At `FLOWLB_LOG_LEVEL=warn` that line is silent. `/stats` still moves.

---

## Dave reads slowly

Dave fetches a large body through 8080, but reads it slowly (`scripts/slow_reader.py`, or a phone on a bad link). The backend is a firehose. `drain_read` from the backend appends **b2c**.

If we never stop reading, b2c grows until the process is killed. TCP’s own window will eventually stop the backend, but between “kernel buffer full” and “our `vector` is 2 GB” there is a lot of heap.

At **256 KiB** (high watermark): set `b2c_paused_`, clear `EPOLLIN` on the **backend** fd. Log `backpressure on`. Stop pulling into userspace. The kernel will then apply TCP backpressure to the backend.

As Dave reads, `flush_write` to him drains b2c. At **64 KiB** (low watermark): clear the pause, restore `EPOLLIN`, log `backpressure off`. Two thresholds, not one, so the interest bits do not flap on every packet around 256 KiB.

Then the edge-triggered trap again: data may already be readable from while we were paused. `epoll_ctl` to put `EPOLLIN` back does **not** generate an edge. `need_speculative_backend_ = true`, and `finish_io` drains immediately. A guard of 8 iterations so a pathological readable/writable loop cannot run forever in one handler.

The same story runs the other way: fast client, slow backend, **c2b**, pause `EPOLLIN` on Alice.

Dave does not take down the waiter. RSS stays in the same band as a fast curl of the same payload.

---

## b2 dies

The operator (or `scripts/demo.sh`) does:

`curl -sS http://127.0.0.1:9002/kill`

That hits b2 **directly**, not through 8080. The dummy process answers `dying` and `_exit`s. Nothing is listening on 9002.

flowlb does not get a “b2 left” packet. In-flight connections to b2 fail on their own when the next `read`/`write` errors. Other connections are not RST. Mark-down is “no new tickets,” not “evacuate the dining room by setting it on fire.”

Health still ticks. `connect` to 9002 fails (`ECONNREFUSED`) or a probe times out. `record_probe(fail)`: ok streak 0, fail streak 1, 2, 3. On the third consecutive fail, `set_healthy(false)`. Log at WARN: `backend down id=b2 ... fails=3`. One lost SYN would not have been enough. Thresholds exist because networks drop packets.

`pick()` now skips index 1. Alice’s next curl is b1 or b3. `/stats` shows `"healthy": 2` and b2 `"healthy": false`. 8080 still serves.

If Bob was already connected to b2 when it died, his Connection hits a read/write error, `close_now`, deferred erase, `release`. His curl fails. Carol on b3 does not notice.

---

## b2 comes back

`scripts/backends.sh start-one b2` listens on 9002 again.

Probes start succeeding. Fail streak 0, ok streak 1, then 2. On the second consecutive success, `set_healthy(true)`. Log at INFO: `backend up   id=b2 ... successes=2`. Optimistic start meant it was healthy at process boot before any probe; this rejoin is the pessimistic path, earned.

Cursor and least-connections see it as a candidate again. A 3-second loadgen after rejoin in the demo: `fail=0`.

Probe fds use the same deferred-close parking lot as connections. Health checking is historically where fd leaks hide: a timer that opens sockets forever. The Phase 5 gate asserts the process fd count is flat across kill / rejoin / all-down.

---

## Every kitchen is dark

If b1, b2, and b3 are all down, `pick()` returns empty. `on_accept` increments `rejected`, logs, returns. Alice’s fd closes. curl’s wall time is about 10 ms. The process stays up. Listeners stay up. Health keeps probing. A load balancer that hangs when the world is on fire makes the outage worse: clients pile up, fds pile up, the balancer dies too.

There is no userspace queue of waiting guests. Two queues exist, both in the kernel: the listen backlog, and each socket’s receive buffer. We do not `malloc` a Connection until `accept`. All-down is fail-closed, not “wait for a probe.”

---

## Out of file descriptors

`accept` can fail with `EMFILE` (this process) or `ENFILE` (the system). Under level-triggered listen, if you leave `EPOLLIN` on, `epoll_wait` returns immediately forever: 100% CPU, accepting nobody.

The spare `/dev/null` fd exists for this. Close the spare (one slot frees), accept one incoming connection and drop it, reopen the spare, **clear `EPOLLIN`** on the listen fd, set `paused_`. Log at ERROR.

When any connection (or stats session) is reclaimed, `resume_if_paused` puts `EPOLLIN` back. The 5-second stats tick also tries, as a backstop for probe closes. Until then the listen fd is silent on purpose.

---

## You press Ctrl-C

`SIGINT` is blocked. It arrives as a readable `signalfd`. `handle_signal` reads the info, logs `signal=2, shutting down`, sets `stop_`. `run()`’s `while` ends.

Then Proxy tears down in order: drop StatsServer (close sessions, timer, stats listen), log `shutdown, dropping N connections`, `release` each remaining connection’s backend index, clear the map (close the pairs), drop HealthChecker (timer and in-flight probes), drop the public Listener.

ASan at exit is meaningful only if we actually close things. `kill -9` skips all of this; do not use it to judge leaks.

Clients in the middle of a transfer see a reset or a short read. That is shutdown, not a later “drain until idle” product.

---

## What is running, and what is not

One OS thread. Highly concurrent: many Connections can be alive, each waiting on I/O. Not parallel on this core: one `on_event` at a time. No mutex on `BackendPool::active`. You still need deferred destroy, because the same thread can see the same connection twice in one batch.

The dummy backends *are* parallel: Python `ThreadingHTTPServer`. The kernel is parallel. flowlb’s waiter is not.

There is no second health thread. A thread that `sleep`s and `connect`s would be the first mutex in the process, or a data race, or a second event loop pretending not to be one. The timer is an fd. Probes are fds. Ctrl-C is an fd. Everything interesting waits on the same board.

Port 8080 does not parse HTTP. Alice’s `GET /` works because both ends speak it and we did not mangle the bytes. Port 8081 parses a first line because we *are* the HTTP server there.

The Phase 1 blocking proxy under `spike/` is not this flow. It accepted one client, connected to one backend, and spawned two threads that blocked in `read`/`write`. It proved forwarding. It is not the architecture.

---

## Next request

There is no “hand 2.” The seat list in the poker engine was four people until you restarted the table. Here the backend list is the config file until you restart the process. Health flags live in memory. Counters live in memory. Kill flowlb and they come back from config plus the first probe round.

Alice can curl again. Cursor is wherever `pick` left it. Stacks of bytes from the last connection are gone. The Keyring equivalent — identity, sockets, buffers — is created per Connection and destroyed with it. The pool, the loop, and the listeners stay.

If you want a different algorithm, a different port, or a different kitchen, change the JSON and start again. There is no control plane.
