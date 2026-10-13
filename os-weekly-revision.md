# Operating Systems Interview — Weekly Revision Guide

Synthesized from `os_notes.db`. You already read the long notes. Use this **every week** as a timed recitation pass. Every critical fact is here; examples are compressed, not dropped.

**How to use (one full pass per week, ~40 min/day):**

| Day | Cover | Recite until you can say it without looking |
|-----|--------|---------------------------------------------|
| **Mon** | Processes, threads, PCB, fork/exec | States, context-switch steps, zombie vs orphan, why threads cheaper |
| **Tue** | CPU scheduling + MLFQ / RT / inversion | Metrics, Gantt algos, quantum rule, Pathfinder |
| **Wed** | Synchronization + classic problems | CS 3 properties, mutex vs sem, P-C order, dining 4 fixes |
| **Thu** | Deadlocks | Coffman, RAG, Banker's full example, prevent vs avoid vs detect |
| **Fri** | Memory + virtual memory | VA→PA, TLB, page fault, FIFO/LRU/OPT, Belady, thrashing |
| **Sat** | Syscalls, IPC, file systems | Mode switch, 5 IPC, inode walk, hard vs symlink |
| **Sun** | Disk / I/O / advanced FS / perf + cheat sheet | SCAN family, DMA, journaling, CFS, full recitation |

Each day: **Recite** the boxed facts → **Scan** the section → **Answer** the self-quiz at the end of that day. Sunday: run the full cheat sheet out loud.

---

# DAY 1 — Processes & Threads

## Program vs Process vs Thread

A **program** is bytes on disk. Passive. A recipe on paper.

A **process** is a running program. OS loads it, gives CPU/memory/fds, tracks execution. The recipe being cooked.

A **thread** is a unit of execution *inside* a process. Threads share **code, data, heap**. Each has its **own stack and registers**.

Process = factory. Threads = workers. Same building, same tools; each has its own to-do list (stack).

## Process Control Block (PCB)

OS ID card for every process. Lives in **kernel memory**. Why context switching works: snapshot CPU into PCB, restore later, process never knows it was paused.

| Field | What |
|---|---|
| **PID** | Unique identifier |
| **State** | Running, Ready, Waiting, … |
| **PC** | Next instruction address |
| **Registers** | GPRs, SP, flags |
| **Scheduling** | Priority, queue pointers |
| **Memory** | Page-table base, segments |
| **Accounting** | CPU time, limits, job # |
| **I/O** | Open files, devices in use |

## Process states

```
New → Ready ⇄ Running → Terminated
               ↕
             Waiting
```

| State | Meaning |
|---|---|
| **New** | Being created, PCB allocated |
| **Ready** | In memory, waiting for CPU |
| **Running** | On a core (one process per core) |
| **Waiting** | Blocked on I/O, signal, child |
| **Terminated** | Done; PCB hangs until parent `wait()` |

**Transitions (say the verb):**
- Ready → Running: **dispatched**
- Running → Ready: **preempted** (timer or higher priority)
- Running → Waiting: **blocking syscall**
- Waiting → Ready: **event completed**
- Running → Terminated: **`exit()`**

## Context switch — 8 steps, recite in order

1. Timer interrupt (or blocking syscall)
2. CPU saves minimal state, jumps to interrupt handler
3. OS saves **all** of A's registers into A's PCB
4. A's state → Ready or Waiting
5. Scheduler picks B
6. OS restores B's registers from B's PCB
7. B's state → Running
8. CPU jumps to B's saved PC

**Pure overhead.** ~1–10 μs of CPU work; the **real cost is cache/TLB pollution** — A's working set is gone, B starts cold.

Triggers: quantum expired, blocking syscall, higher-priority ready, voluntary yield.

## fork / exec / wait

Type `ls` in bash:

1. bash **`fork()`** — exact copy of bash
2. Child **`exec("ls")`** — replace memory with `ls`
3. `ls` **`exit()`**
4. bash **`wait()`** — collect exit code, **reap the PCB**

Why this design: between fork and exec the child can set fds, cwd, env **before** becoming the target.

**Zombie:** child exited, parent has not `wait()`ed. PCB holds the exit code. Servers that never wait pile up zombies.

**Orphan:** parent dies first. Child re-parented to **PID 1** (`init`/`systemd`), which periodically `wait()`s.

**COW (preview, Day 5):** `fork()` does **not** copy pages. Both PTEs point at the same frames, marked copy-on-write. First write → fault → copy that frame. `fork()` + `exec()` copies almost nothing.

## User threads vs Kernel threads

| | User | Kernel |
|---|---|---|
| Who schedules | Userspace library | OS |
| Kernel sees | One process | Each thread |
| Create/switch | No syscall — fast | Syscall — slower |
| Blocking I/O | **Whole process** blocks | Other threads keep running |
| Multi-core | No true parallelism | Yes |

Linux: **one-to-one**. Every `pthread` is a kernel task (`clone()` + shared address space). “Lightweight processes.”

## Concurrency vs Parallelism

- **Concurrency:** multiple tasks *make progress* by interleaving. Works on **one core**.
- **Parallelism:** actually execute at the same time on **multiple cores**.

Concurrency without parallelism: single-core many threads. Parallelism without concurrency: SIMD. Usual interview case is both.

### Day 1 self-quiz
1. Program vs process vs thread in one sentence each. What is shared vs private in a thread?
2. Recite the PCB fields. Why does the PCB make context switches possible?
3. Draw the 5-state diagram. Name every transition verb.
4. Recite the 8 context-switch steps. What is the *real* cost?
5. `ls` via fork/exec/wait. Zombie vs orphan. Why the gap between fork and exec?
6. User vs kernel threads. What mapping does Linux use? Concurrency vs parallelism.

---

# DAY 2 — CPU Scheduling

Scheduler runs when a process **terminates, blocks, or (preemptive) a timer fires**.

## Metrics

| Metric | Formula / meaning | Who cares |
|---|---|---|
| **Waiting** | Time in the **ready queue** (summed across preemptions) | Batch |
| **Turnaround** | Completion − Arrival | Batch |
| **Response** | Arrival → **first** CPU touch | Interactive |
| **Throughput** | Completions per unit time | System |

Response ≠ waiting. Response is first touch. Waiting is *all* ready-queue time.

## FCFS — First Come First Served

Non-preemptive. Arrival order.

**Convoy effect:** one long job blocks everyone behind it.

```
P1 AT=0 B=24,  P2 AT=1 B=3,  P3 AT=2 B=3
Gantt: [P1 0-24][P2 24-27][P3 27-30]
Wait: 0, 23, 25  → avg 16
```

## SJF — Shortest Job First

Non-preemptive. Free CPU → shortest **next burst**. **Provably optimal** for average waiting time.

We don't know bursts. Predict with **exponential averaging**:

```
τ(n+1) = α · t(n) + (1−α) · τ(n)
```

`t(n)` = actual last burst, `τ(n)` = last prediction. Typical **α = 0.5**.

## SRTF — Shortest Remaining Time First

**Preemptive SJF.** New arrival with burst < current remaining → preempt.

```
P1 0/7,  P2 2/4,  P3 4/1,  P4 5/4

t=0  P1 runs (rem 7)
t=2  P2=4 < P1 rem 5 → P2
t=4  P3=1 < P2 rem 2 → P3
t=5  P3 done. P4=4, P2 rem=2 → P2
t=7  P2 done. P4=4 < P1 rem 5 → P4
t=11 P4 done. P1
t=16 P1 done

Gantt: [P1 0-2][P2 2-4][P3 4-5][P2 5-7][P4 7-11][P1 11-16]
CT: 16, 7, 5, 11
TAT = CT−AT: 16, 5, 1, 6  → avg 7
Wait = TAT−Burst: 9, 1, 0, 2  → avg 3
```

## Round Robin

Fixed quantum **q**. Expire → preempt, go to **back** of ready queue. Fair, good response.

- q too small → context-switch overhead dominates
- q too large → degenerates to FCFS
- Rule of thumb: q large enough that **~80% of bursts finish in one quantum**

```
P1=5, P2=3, P3=4, q=4, all AT=0
Gantt: [P1 0-4][P2 4-7][P3 7-11][P1 11-12]
Wait: 6, 4, 7  → avg 5.67
```

## Priority

Highest priority first. Preemptive or not.

**Starvation:** low-priority waits forever if highs keep arriving.

**Aging:** bump waiting processes' priority over time. Kills starvation.

## Preemptive vs Non-preemptive

- **Non-preemptive:** keeps CPU until it blocks or exits. Simpler.
- **Preemptive:** timer can steal the CPU. Required for interactive and real-time.

## Multilevel Queue

Fixed partitions: system / interactive / batch, each with its own algo. Higher queues preferred. Lower queues **starve** if higher never empty.

## Multilevel Feedback Queue (MLFQ)

Processes **migrate**. Typical rules:

1. New process → highest queue
2. Uses full quantum → **demote** (CPU-bound)
3. Blocks before quantum → **stay** (I/O-bound)
4. **Aging:** periodically boost everyone to the top

Self-tunes without knowing burst times. Interactive stays high; batch sinks. What general-purpose OS schedulers approximate. Linux historically MLFQ-like; modern Linux is **CFS** (Day 7).

## Real-time

| | Hard | Soft |
|---|---|---|
| Miss a deadline | **System failure** (fly-by-wire, pacemaker) | Quality drop (video, audio, UI) |

**RMS (Rate Monotonic):** static priority. Higher frequency → higher priority. Optimal among static-priority preemptive. *n* periodic tasks schedulable if utilization ≤ **n(2^(1/n) − 1)**.

**EDF (Earliest Deadline First):** dynamic. Nearest absolute deadline wins. Optimal — **100% utilization** if the set is feasible. Harder to implement.

## Priority inversion

L holds a mutex. H wants it, blocks. M (medium) preempts L. L cannot release. **M blocks H.**

**Mars Pathfinder (1997):** low-priority weather task held a mutex the high-priority bus task needed. Medium tasks ran; watchdog reset the spacecraft.

**Priority Inheritance:** L temporarily runs at **H's** priority until it releases. M cannot preempt L.

**Priority Ceiling:** each resource's ceiling = highest priority of any task that might use it. Acquirer is raised to that ceiling.

### Day 2 self-quiz
1. Four metrics. Response vs waiting in one sentence.
2. Convoy effect. Recite the FCFS 24/3/3 Gantt and avg wait.
3. SJF optimality + the τ formula. Recite the SRTF Gantt and TAT/wait averages.
4. RR quantum rule of thumb. Recite the q=4 example.
5. Starvation + aging. MLFQ four rules.
6. Hard vs soft RT. RMS vs EDF. Pathfinder story + inheritance vs ceiling.

---

# DAY 3 — Synchronization

## Race condition

Outcome depends on **interleaving**, and at least one access is a write. Classic: read-modify-write without a lock.

```
balance = balance + 100   is  LOAD, ADD, STORE

T1 LOAD 1000
T1 ADD  1100
        ← switch →
T2 LOAD 1000
T2 SUB   950
T2 STORE 950
        ← switch →
T1 STORE 1100   ← wrong (should be 1050)
```

## Critical section — three properties (all required)

A CS accesses shared data and must run on **at most one** thread at a time.

1. **Mutual Exclusion** — only one in its CS
2. **Progress** — if CS is empty and someone wants in, the decision cannot be postponed. Threads *outside* the CS cannot block others.
3. **Bounded Waiting** — after you request entry, a bound on how many others enter first. No starvation.

## Mutex

Binary: locked / unlocked. `lock()` acquires or **sleeps**. `unlock()` releases and wakes one.

`lock`/`unlock` themselves are **atomic** (hardware).

**Ownership:** only the locker can unlock. That is the difference from a semaphore.

## Semaphore

Integer + two atomic ops:

```
wait(S) / P:   if S > 0: S--  else block on S's queue
signal(S) / V: S++ ; if waiters: wake one
```

- **Binary** (0/1): mutex-like, **no ownership** — any thread can signal
- **Counting** (0..N): N identical resources. Init to N. Three printers → S=3

**Use mutex for exclusion. Use semaphore for signaling / resource counts.**

## Spinlock

Busy-wait: `while (test_and_set(&lock));`

**Good:** very short CS on **multi-core** (spin cheaper than sleep+wake).

**Bad:** long waits (burns CPU); **single-core** (holder cannot run while you spin).

Linux kernel uses spinlocks everywhere — kernel/interrupt context often **cannot sleep**.

## Monitor & condition variables

Monitor = class whose methods are mutually exclusive. Java `synchronized`.

CVs inside a monitor:

- `wait(cv)`: **atomically** release the monitor lock and sleep. On wake, re-acquire before return
- `signal(cv)`: wake one
- `broadcast(cv)`: wake all

Atomic release-and-sleep is mandatory. Release then sleep as two steps → another thread can signal **between** them → you sleep forever (lost wakeup).

## Producer-Consumer (bounded buffer)

```
mutex = 1,  empty = N,  full = 0

Producer:  wait(empty); wait(mutex); add; signal(mutex); signal(full)
Consumer:  wait(full);  wait(mutex); take; signal(mutex); signal(empty)
```

**`wait(empty)` / `wait(full)` BEFORE `wait(mutex)`.** Reverse = deadlock: producer holds mutex, waits for empty slots; consumer needs mutex to free a slot.

Rule: acquire **resource semaphores before the mutex**.

## Readers-Writers

Many readers together. Writer exclusive.

```
read_count = 0;  mutex = 1;  db = 1

Reader:
  wait(mutex); read_count++; if == 1: wait(db); signal(mutex)
  READ
  wait(mutex); read_count--; if == 0: signal(db); signal(mutex)

Writer: wait(db); WRITE; signal(db)
```

First reader locks writers out; last reader lets them back. **Readers-preference → writers starve** if readers keep arriving. Writers-preference can starve readers. Fair variant needs extra semaphores.

## Dining Philosophers

5 philosophers, 5 chopsticks. Each needs two. Naive “left then right” **deadlocks**: all pick left, all wait for right — circular wait.

Four fixes:
1. **Asymmetric:** odd pick left first, even pick right first
2. **At most 4** philosophers try at once
3. **Pick both atomically** (mutex around the pair)
4. **Resource ordering:** chopsticks 1–5, always lower number first

## Peterson's algorithm (2 processes, software only)

```
flag[i] = true;
turn = j;                          // "you go first"
while (flag[j] && turn == j) ;     // wait
// CS
flag[i] = false;
```

Satisfies ME + progress + bounded waiting. **Broken on modern CPUs** without memory fences — OoO + store reordering.

## TAS and CAS

**Test-and-Set:** atomically read and set true. Spinlock = `while (TAS(&lock));` … `lock = false`. **No bounded waiting.**

**Compare-and-Swap:**

```
CAS(ptr, expected, new):
  if *ptr == expected: *ptr = new; return expected
  else return *ptr   // no swap
```

Foundation of **lock-free** structures. Optimistic: try update; CAS fail → retry.

Lock-free: no deadlock, no priority inversion, scales on multi-core. Java `ConcurrentHashMap`, Linux RCU.

### Day 3 self-quiz
1. Recite the bank LOAD/ADD/STORE interleaving. When is it a race?
2. Three CS properties, in order, in your own words.
3. Mutex vs semaphore (ownership, range, use). Spinlock vs mutex (when each wins; kernel?).
4. Why `wait(empty)` before `wait(mutex)`? Recite both P-C sequences.
5. Recite readers-writers. Who starves? Four dining-philosophers fixes.
6. Peterson in 4 lines. Why fences? TAS vs CAS. What does lock-free buy you?

---

# DAY 4 — Deadlocks

## Definition

A set of processes, each waiting for an event **only another in the set** can cause. Nothing proceeds.

## Coffman — all four required

Prevention = remove **at least one**.

1. **Mutual Exclusion** — at least one resource is non-shareable
2. **Hold and Wait** — holds one, waits for another
3. **No Preemption** — cannot forcibly take; holder releases voluntarily
4. **Circular Wait** — P1 waits P2 waits … Pn waits P1

## Resource Allocation Graph

Circles = processes. Squares = resources (dots = instances).

- Request: **P → R**
- Assignment: **R → P**

**Single-instance:** cycle ⟺ deadlock (necessary **and** sufficient).

**Multi-instance:** cycle is necessary, **not** sufficient. Need Banker's (or detection) to decide.

## Prevention — attack one condition

| Attack | How | Cost |
|---|---|---|
| Mutual exclusion | Make shareable | Often impossible (mutex, printer) |
| Hold and wait | Request **all** at start, or release all before asking more | Low utilization, starvation |
| No preemption | Can't get next → release what you hold | Works for CPU/memory; fails for printers / held mutexes |
| Circular wait | **Total order** on resource types; request only in increasing order | The practical one. Same as “lock `min(id)` first” |

## Avoidance — Banker's Algorithm

Need each process's **maximum claim** up front. Grant a request only if the system **stays safe**.

**Safe state:** there exists a **safe sequence** P1…Pn such that each Pi can be satisfied by **Available + resources of all Pj, j < i**.

Safe ⇒ all can finish ⇒ **cannot deadlock**. Unsafe **may** deadlock (depends on future requests) — not a guarantee of deadlock.

```
Available[m]        free instances
Max[n][m]           max demand
Allocation[n][m]    currently held
Need[n][m]          Max − Allocation
```

**Safety algorithm:**

```
Work = Available;  Finish[*] = false
loop:
  find i: !Finish[i] AND Need[i] ≤ Work
  if found: Work += Allocation[i]; Finish[i] = true
  else break
all Finish? SAFE : UNSAFE
```

**Resource-request:**

```
Request > Need     → error
Request > Available → wait
tentatively: Available -= Req; Alloc += Req; Need -= Req
run Safety
  SAFE → grant
  UNSAFE → rollback, Pi waits
```

### Worked example — recite the sequence

Resources A=10, B=5, C=7. Available = **[3, 3, 2]**.

```
     Alloc     Max      Need
     A B C    A B C    A B C
P0   0 1 0    7 5 3    7 4 3
P1   2 0 0    3 2 2    1 2 2
P2   3 0 2    9 0 2    6 0 0
P3   2 1 1    2 2 2    0 1 1
P4   0 0 2    4 3 3    4 3 1
```

Work starts [3,3,2]:

```
P1 Need[1,2,2] ≤ [3,3,2] → Work [5,3,2]
P3 Need[0,1,1] ≤ [5,3,2] → Work [7,4,3]
P4 Need[4,3,1] ≤ [7,4,3] → Work [7,4,5]
P0 Need[7,4,3] ≤ [7,4,5] → Work [7,5,5]
P2 Need[6,0,0] ≤ [7,5,5] → done
```

Safe sequence: **P1 → P3 → P4 → P0 → P2**.

## Detection & Recovery

Don't prevent/avoid → detect later.

**Detection (multi-instance):** safety algorithm but use **Request** (current waits) instead of Need. Processes the algo cannot finish are deadlocked.

**Recovery:**
- Kill all deadlocked, or kill one-by-one (victim: priority, CPU used, resources held)
- Preempt resources and roll back. Problems: victim choice, rollback mechanism, **starvation** of the same victim

Production OS (Linux): **detection-ish** (mutex deadlock detector in debug) + app-level lock ordering. Banker's is unused in general-purpose OS — you cannot know max claims.

**Prevention vs Avoidance vs Detection:**
- **Prevention:** statically make one Coffman condition impossible (resource order)
- **Avoidance:** dynamically grant only if still safe (Banker's)
- **Detection:** allow, find cycles, recover

### Day 4 self-quiz
1. Four Coffman conditions. How does lock ordering remove circular wait?
2. RAG: when is a cycle sufficient? When only necessary?
3. Recite the four prevention attacks and their costs.
4. Safe state in one sentence. Can a safe system deadlock? Can an unsafe system *not* deadlock?
5. Recite safety + request algorithms. Run the [3,3,2] example to P1→P3→P4→P0→P2.
6. Detection vs Banker's (Need vs Request). Why Banker's is unused in real OS.

---

# DAY 5 — Memory + Virtual Memory

## Logical vs Physical

**Logical/virtual:** what the CPU generates. Process thinks it owns 0…max.

**Physical:** real RAM.

**MMU** translates logical → physical on **every** access. Ten processes can each think code starts at 0.

## Contiguous allocation (old)

One contiguous chunk per process.

- **Internal fragmentation:** allocated 50KB, used 35KB. Waste **inside** the block. Nobody else can use it.
- **External fragmentation:** 60KB free total, three 20KB holes. 50KB request fails.
- **Compaction:** slide everything together. Extremely expensive.

## Paging

Physical → fixed **frames**. Logical → same-size **pages**. Map arbitrarily.

Page = frame size. Typical **4KB**.

```
logical = [ page number p | offset d ]
32-bit, 4KB pages:  20-bit p + 12-bit d
physical = f × page_size + d     (f from page table[p])
```

**Kills external fragmentation.** Any free frame works.

**May have internal:** last page partially empty. Max waste = page_size − 1.

## PTE bits

| Bit | Meaning |
|---|---|
| **Valid / Present** | In RAM? 0 → page fault |
| **Dirty** | Written? Write-back only if dirty |
| **Reference** | Recently accessed (Clock / LRU approx) |
| **Protection** | R/W/X |
| **User/Supervisor** | User-mode allowed? |

## Multi-level page tables

32-bit, 4KB, 4-byte PTE → 2^20 entries = **4MB per process**, even if the process uses 1MB.

Page the page table.

**Two-level (32-bit):** `[10-bit outer | 10-bit inner | 12-bit offset]`. Inner tables exist only for used regions.

**x86-64:** 48-bit VA → **four** 9-bit indexes + 12-bit offset. 4-level page tables.

## TLB

Without TLB: **≥2 memory accesses** per load (walk table, then data). Fatal.

TLB = hardware cache of PTEs. 64–1024 entries, L1-speed.

```
(p, d) → TLB hit (~98–99%): f immediately
       → miss: walk page table, fill TLB
physical = f × page_size + d
```

**Context switch:** old process's mappings are wrong.

- **Flush** the TLB (expensive), or
- **ASID** — tag each entry with a process ID so they coexist. Linux uses ASIDs when hardware has them.

## Segmentation

Variable-size **logical** pieces: code, data, stack, heap.

```
[ segment s | offset d ]
physical = base[s] + d    after d < limit[s]
```

Paging = OS-centric, fixed, no external frag. Segmentation = programmer-centric, **has** external frag, no internal. Modern: **paged segmentation** or pure paging with OS-invented “segments.”

## VA → PA walk (interview — do mechanically)

1. CPU emits VA → split `(p, d)`
2. TLB lookup on `p`
3. Hit → `f`. Miss → walk page table (maybe 4 levels), fill TLB
4. If PTE valid=0 → **page fault** (below)
5. PA = `f × page_size + d`

## Demand paging

Load a page **only when first accessed**. Untouched pages never enter RAM. Processes can be larger than RAM. Faster startup. Higher multiprogramming.

## Page fault — 8 steps

PTE valid = 0:

1. MMU raises **page-fault exception**
2. OS handler takes control
3. **Validate** the address (legal range? else SIGSEGV)
4. Find the page on disk (swap or executable)
5. Find a free frame, or **evict** via replacement
6. **Read from disk** into the frame (milliseconds)
7. Update PTE: frame #, valid=1
8. **Restart the faulting instruction**

Normal access: nanoseconds. Faults must be rare.

## Replacement

No free frame → evict.

**FIFO** — oldest page. Simple. **Belady's anomaly:** more frames can **increase** faults. Unique to non-stack algorithms.

**OPT** — evict the page used farthest in the **future**. Impossible (needs future). Gold-standard lower bound.

**LRU** — evict farthest in the **past**. Temporal locality. **No Belady** (stack algorithm). True LRU is expensive on every access:

- Counter per PTE
- Stack: move accessed to top, bottom = LRU

**Clock (Second Chance)** — practical LRU approx. Circular list + **reference bit** (hardware sets 1 on access). Hand: if bit=1, clear and skip (second chance); if 0, evict.

### Worked example — 3 frames, `1 2 3 4 1 2 5 1 2 3 4 5`

**FIFO — 9 faults:**

```
1 → [1]          F
2 → [1,2]        F
3 → [1,2,3]      F
4 → [4,2,3]      F evict 1
1 → [4,1,3]      F evict 2
2 → [4,1,2]      F evict 3
5 → [5,1,2]      F evict 4
1 → [5,1,2]      H
2 → [5,1,2]      H
3 → [3,1,2]      F evict 5
4 → [3,4,2]      F evict 1
5 → [3,4,5]      F evict 2
```

**OPT — ~7 faults** (lower bound). At 4: evict 3 (used latest). At 5: evict 4.

**Belady:** FIFO + more frames can **raise** faults. LRU and OPT never do (stack algorithms: a page in a k-frame set is also in every larger set).

## Thrashing

More time **paging** than executing.

1. Process has fewer frames than its **working set**
2. Bring A, evict B that you need next
3. CPU util collapses
4. Scheduler thinks “idle” → **admits more processes**
5. Even fewer frames each → death spiral

**Working set** WS(t) = pages touched in the last Δ references. Give each process |WS(t)| frames. If Σ WS > RAM: **suspend** some processes.

Fix: working-set admission, add RAM, reduce multiprogramming.

## Copy-on-Write

`fork()` marks pages COW in both page tables; both point at the same frames (read-only). First write:

1. Fault on write to RO page
2. OS sees COW flag
3. Copy the frame
4. Point writer's PTE at the new frame
5. Resume the write

`fork()` + `exec()`: fork marks COW, exec **discards** the address space. Near-zero copy.

### Day 5 self-quiz
1. Internal vs external. What does paging kill? What remains?
2. 32-bit 4KB split. Recite PTE bits. Why multi-level? x86-64 4-level split.
3. TLB hit path vs miss. Flush vs ASID. Full VA→PA walk including fault.
4. Page-fault 8 steps. Why restart the instruction?
5. FIFO vs OPT vs LRU vs Clock. Recite the 12-ref FIFO trace (9 faults). Belady in one sentence.
6. Thrashing spiral + working-set fix. COW on fork+exec.

---

# DAY 6 — Syscalls, IPC, File Systems

## User mode vs Kernel mode

| | User | Kernel |
|---|---|---|
| Privileged ops | **No** | Yes |
| Hardware, page tables, disable IRQ | Forbidden | Allowed |
| Who runs | Apps | OS |

Without this, any program could trash another process, the FS, or the machine. OS is the only privileged intermediary.

## System call path

Only legal user → kernel gate.

1. App calls libc wrapper (`read()`, `write()`, …)
2. Wrapper puts args in registers/stack
3. **`syscall`** (x86-64) / `svc #0` (ARM) / `int 0x80` (old)
4. CPU **atomically** → kernel mode, jumps to dispatcher
5. Syscall number indexes the table → handler
6. Handler **validates** args, does the work
7. Return value in a register
8. Back to user mode
9. Wrapper returns

Categories: process (`fork exec exit wait`), file (`open read write close`), device (`ioctl`), comm (`pipe socket send recv`), info (`getpid gettime`).

## Interrupts vs Traps vs Exceptions

| | Sync? | Cause | Example |
|---|---|---|---|
| **Hardware interrupt** | Async | Device | Timer, disk done, keypress |
| **Trap** | Sync, **deliberate** | Software | `syscall`, sometimes div-by-zero |
| **Exception** | Sync, **error** | Fault | Page fault (handled), SIGSEGV (kill) |

**IVT / IDT:** array of handler addresses, indexed by interrupt number. Hardware looks up and jumps.

## Kernel architectures

| | What's in kernel mode | Tradeoff | Examples |
|---|---|---|---|
| **Monolithic** | Scheduler, FS, drivers, net, MM | Fast (function calls). Driver bug can panic | **Linux** |
| **Microkernel** | IPC + basic MM + tiny sched. FS/drivers are user servers | Safer, modular. **Slow** (IPC per I/O) | Mach, MINIX, QNX |
| **Hybrid** | Some in, some out | Middle | Windows NT, macOS XNU |

## IPC — five mechanisms

Processes are isolated. Need a channel.

**Pipes** — unidirectional byte stream. Kernel buffer. `ls | grep`.
- **Anonymous:** `pipe()`, related processes only (fd copied on fork)
- **Named (FIFO):** pathname; any two processes; outlives creator

**Shared memory** — same physical pages mapped into both VAs. Fastest after setup (no kernel per access). **You** synchronize (mutex/sem). `shmget` / `shmat`.

**Message queues** — kernel queue. `msgsnd` / `msgrcv`. Select by type. More structured than pipes.

**Sockets** — bidirectional. Across machines (TCP/IP) or local (**Unix domain** — no net stack, faster). Foundation of network I/O.

**Signals** — async notification. Software interrupt. Handler via `signal`/`sigaction`. **`SIGKILL` and `SIGSTOP` cannot be caught or ignored.** Others: `SIGTERM` (graceful), `SIGSEGV`, `SIGALRM`, `SIGCHLD`.

### Shared memory vs message passing

| | Shared memory | Message passing |
|---|---|---|
| Speed | Fast, zero-copy after setup | Kernel copy each time |
| Sync | **Your** job | Built in |
| Best for | Large, high-throughput | Protocols, distributed, safety |

**Pipe vs socket:** pipe = uni, same machine. Socket = bi, can be remote. Socket is the general case.

## File descriptors — three tables

fd is an **int in a process**. 0/1/2 = stdin/stdout/stderr. `open()` → lowest unused.

1. **Per-process fd table:** fd → open-file entry
2. **System-wide open-file table:** offset, flags, inode ptr (shared across fork)
3. **Inode table:** one in-memory inode per open file

That's why parent and child **share the file offset** after fork (same open-file entry), but two `open()`s of the same file get **independent** offsets.

## Inode

Everything about a file **except the name**:

size, type, UID/GID, `rwxrwxrwx`, ctime/mtime/atime, **link count**, **block pointers**.

Filename lives in the **directory** (name → inode #). File dies when link count = 0 **and** no process has it open.

## Allocation

| Scheme | Idea | Win | Loss |
|---|---|---|---|
| **Contiguous** | Consecutive blocks | Fast seq + easy random | External frag, hard to grow |
| **Linked** | Each block → next | No external frag | Slow random; one broken ptr loses the tail |
| **FAT** | Next-ptrs in a **table in memory** | O(1)-ish random (walk FAT in RAM) | Table size, corruption of FAT |
| **Indexed (Unix)** | Pointers in the inode + indirection | Balanced | Extra I/O for large files |

**Unix inode pointers (4KB blocks, 4-byte ptrs → 1024 ptrs/block):**

```
12 direct              → 12 × 4KB = 48KB
1  single-indirect     → 1024 × 4KB = 4MB
1  double-indirect     → 1024² × 4KB = 4GB
1  triple-indirect     → ~4TB
```

## Hard link vs symlink

| | Hard | Symlink |
|---|---|---|
| What | Another dir entry → **same inode** | Separate inode whose **content is a path** |
| Link count | +1 | Unchanged on target |
| Cross FS | **No** (inode # is per-FS) | Yes |
| Dangling | Impossible (same file) | Yes, if target deleted |
| Command | `ln file hard` | `ln -s file soft` |

## `open` + `read` walkthrough

`open("/home/zap/notes.txt", O_RDONLY)`:

1. Root inode **#2**
2. Lookup `"home"` → inode N
3. Lookup `"zap"` → inode M
4. Lookup `"notes.txt"` → inode K
5. Load K, check UID/GID perms
6. Open-file table entry (offset=0, flags, → K)
7. Process fd table → that entry
8. Return fd

`read(fd, buf, 100)`:

1. fd → open-file → inode K
2. Block index = `offset / block_size`
3. Walk direct/indirect pointers
4. Page cache: hit → copy; miss → disk → cache → copy
5. Advance offset by 100

### Day 6 self-quiz
1. Recite the 9-step syscall path. Interrupt vs trap vs exception.
2. Monolithic vs micro vs hybrid + one example each.
3. Five IPC mechanisms in one line each. Shared mem vs messages. Pipe vs socket. Uncatchable signals.
4. Three kernel tables for an fd. Why fork shares offset but two `open()`s don't.
5. Inode contents. Recite 12 / 1 / 1 / 1 sizes. Hard vs symlink table.
6. Walk `open("/home/zap/notes.txt")` then `read`. Where does the filename live?

---

# DAY 7 — Disk, I/O, Advanced FS, Performance

## HDD

Platters → tracks → sectors (512B/4KB). Cylinder = same track all platters. Heads on arms.

| Time | What | Cost |
|---|---|---|
| **Seek** | Arm to track | **Dominant ~5–10 ms** |
| **Rotational** | Sector under head | ~4 ms at 7200 RPM |
| **Transfer** | Read/write the sector | Fastest |

Access ≈ seek + rot + transfer.

## Disk scheduling

Arm at **50**. Queue: `98, 183, 37, 122, 14, 124, 65, 67`

| Algo | Behavior | Notes |
|---|---|---|
| **FCFS** | Arrival order | Movement 640. Simple, often terrible |
| **SSTF** | Nearest next | 50→65→67→98→122→124→183→37→14. ~236. **Starves** far requests |
| **SCAN** | Elevator: one way, service, reverse | From 50 up: 65,67,98,122,124,183 then 37,14. Bounded wait |
| **C-SCAN** | Service **one direction only**, jump back empty | More uniform. SCAN biases the **middle** (hit twice per sweep) |
| **LOOK** | SCAN but reverse at last **request**, not disk end |
| **C-LOOK** | LOOK + circular. What modern HDDs use |

**SSD:** no seek, no rotation. Any block equal. **NOOP** or deadline. Do not SCAN an SSD.

## I/O methods

**Polling:** spin on status register. Waste. Only if device finishes in μs.

**Interrupt-driven:** start I/O, do other work, ISR on completion. Better. Bulk I/O → interrupt **per block** → too many.

**DMA:**

1. CPU programs DMA: src, dst, size, direction
2. CPU leaves
3. DMA moves device ↔ memory
4. **One** interrupt at the end

4MB disk read: CPU-driven = millions of moves. DMA = one setup + one IRQ.

## Buffering / cache / spool

- **Single buffer:** OS buffer between user and device
- **Double buffer:** device fills A while user drains B; swap
- **Circular:** ring of N — the modern default
- **Page cache:** file data in free RAM. Shared across processes. Writes often **write-back** until `fsync()`. `free -h` “used” is mostly cache — reclaimable
- **Spooling:** disk queue for a non-interleavable device (printer). Daemon feeds one job at a time

## Crash consistency & journaling

One logical append is **three** writes: data block, inode pointers/size, bitmap. Crash between 1 and 2 → data on disk, nothing points at it, bitmap thinks free.

**Journaling (WAL for the FS):** write a journal entry **first**, then apply, then mark committed.

Recovery:
- Incomplete journal → ignore (pre-op state)
- Complete journal, not applied → **replay**
- Applied + committed → done

Journal commit is the **atomic decision**. ext4, NTFS, HFS+, APFS.

**Metadata journaling** (common): only metadata. Faster. Possible stale data after crash. Full journal = safer, **~2× write amp**.

**`write()`** returns when data is in the **page cache**. **`fsync(fd)`** blocks until dirty pages hit **disk**. Databases are slow because they `fsync` every commit.

## Log-structured FS (LFS)

Whole disk is a **sequential log**. Never overwrite in place.

Win: every write sequential (HDD + flash). Easy recovery (replay). Write-heavy.

Loss: reads need an **inode map**. **Segment cleaning** (GC) to reclaim stale segments.

Lives on in: SSD **FTL**, Kafka, LevelDB/RocksDB.

## mmap and NUMA (advanced memory leftovers)

**mmap:** file appears as VA. Loads/stores **are** file I/O. OS pages, caches, shares physical pages across mappers. Executables and `.so`/`.dll` are mmap'd, not copied. Large `malloc` often mmap.

**NUMA:** each socket has local RAM. Local ~50 ns, remote ~150 ns. Allocate on the node that will run. Keep the process on that socket. `numactl`.

## Cache locality

- **Temporal:** recently used → used again. Whole cache hierarchy.
- **Spatial:** nearby addresses soon. CPU fetches a **64-byte cache line** even for a 4-byte load.

C is **row-major**. `A[i][j]` inner-j is sequential. Inner-i jumps a whole row — miss every time.

## Context-switch cost (why nginx/Redis/Node are event-loop)

Save/restore ~100 ns is the cheap part. TLB flush (no ASID) + **cold cache** is the expensive part.

**One thread + `epoll`/`io_uring`** beats thread-per-connection at high fan-in: no blocked-thread switches, hot cache.

## Memory pressure & OOM

1. Reclaim page cache (LRU)
2. Start **swap** → performance cliff
3. More faults → more I/O → thrashing

**OOM killer:** RAM gone → score processes (RSS, swap, niceness) → kill highest. Can pick wrong. Tune `/proc/PID/oom_score_adj`.

## CFS (Linux)

**Completely Fair Scheduler.** Red-black tree keyed by **virtual runtime** (CPU so far, weighted by priority). Smallest vruntime = leftmost = next. Find **O(1)**, insert **O(log n)**. Near-perfect fairness.

## I/O bottleneck toolkit

- Async + event loop (`epoll`, **`io_uring`** — submit without a syscall on the hot path; shared ring)
- More threads until switch cost wins
- Faster media: SSD ≫ HDD, NVMe ≫ SATA (~10×)
- Warm page cache
- `io_uring` for modern Linux

### Day 7 self-quiz
1. Seek vs rotational vs transfer. Recite SSTF path from 50 and why it starves.
2. SCAN vs C-SCAN (middle bias). LOOK vs C-LOOK. What do you use on SSD?
3. Polling vs interrupt vs DMA in one sentence each. Recite DMA 4 steps.
4. Page cache vs `write` vs `fsync`. Three writes of an append; what journaling fixes.
5. Metadata vs full journal. LFS win/loss. mmap in one sentence. NUMA 50 vs 150.
6. Temporal vs spatial + why row-major. CFS tree key. OOM. Why event-loop beats thread-per-conn.

---

# SUNDAY CLOSE — Full cheat sheet (recite out loud)

**Process vs thread:** process = address space + resources. Thread = stack + registers inside it. Linux 1:1 `clone()`. Threads cheaper: no new address space, TLB stays valid.

**PCB:** PID, state, PC, registers, sched, page-table base, accounting, open fds. Context switch = save A into PCB, restore B. Real cost = cache/TLB.

**States:** New → Ready ⇄ Running → Terminated, plus Waiting. Dispatch / preempt / block / wakeup / exit.

**fork/exec/wait:** copy, replace, reap. Zombie = exited, not waited. Orphan → PID 1. COW: fork copies nothing until write; fork+exec ≈ free.

**Metrics:** wait = ready-queue time. TAT = CT−AT. Response = first CPU. Throughput = done/time.

**Algos:** FCFS convoy. SJF/SRTF optimal avg wait (need bursts; τ = αt + (1−α)τ). RR quantum ~80% bursts fit. Priority + aging. MLFQ: enter top, demote CPU-bound, stay if I/O, age up. RMS static freq; EDF dynamic deadline, 100% if feasible.

**Inversion:** L holds, H waits, M runs. Pathfinder. Inheritance = L runs as H. Ceiling = resource's max user priority.

**CS:** ME + progress + bounded wait. Race = interleaving + a write.

**Mutex = ownership, 0/1. Sem = signal, can count. Spin = burn CPU, short + multi-core / kernel. Monitor + CV: wait atomically drops the lock.**

**P-C:** `wait(empty/full)` **before** mutex. R-W: first reader takes db, last releases; writers starve. Dining: asymmetric / max-4 / atomic pair / order chopsticks.

**Peterson:** flag + turn="you first". Needs fences. TAS: no bounded wait. CAS: lock-free.

**Coffman:** ME, hold-and-wait, no preemption, circular wait. RAG cycle = deadlock iff single-instance. Prevention: resource order. Avoidance: Banker's, max claims, safe sequence. Detection: safety with Request. Unsafe ≠ deadlock. Banker's unused in real OS.

**Paging:** (p,d), 4KB, 20+12 on 32-bit. No external frag. PTE: valid, dirty, ref, RWX, U/S. Multi-level because 4MB tables. x86-64 = 4×9 + 12. TLB 98–99%, flush or ASID.

**Fault:** exception → validate → disk → frame/evict → read → PTE valid → **restart instruction**.

**Replace:** FIFO Belady. OPT future, impossible. LRU past, stack, no Belady. Clock = ref bit second chance. Thrashing = WS > frames → suspend. COW on fork.

**Syscall:** libc → `syscall` → kernel mode → table → validate → return. Trap = deliberate. IRQ = async device. Exception = fault.

**Kernels:** Linux monolithic. Mach/MINIX/QNX micro. NT/XNU hybrid.

**IPC:** pipe uni; shm fastest + you sync; MQ typed; socket bi ± network; signal async. SIGKILL/SIGSTOP uncatchable.

**Inode:** metadata + block ptrs; name in directory. 12 + single 4MB + double 4GB + triple ~4TB. Hard = same inode; symlink = path, can dangle, can cross FS. Three tables: fd → open-file (offset) → inode.

**Disk:** seek dominates. SSTF starves. SCAN elevator. C-SCAN one way (no middle bias). SSD = NOOP. DMA = one setup + one IRQ.

**Journal:** log first, then apply. `write` = cache; `fsync` = disk. LFS = sequential log + GC. mmap = file as VA. NUMA local 50 ns / remote 150 ns.

**Perf:** row-major spatial. Event loop > thread-per-conn. CFS = RB tree on vruntime. OOM killer. `io_uring` = I/O without syscall on hot path.

---

# Interview lines (say these verbatim)

1. **Threads cheaper:** “`fork` builds a new address space and fd table. A thread is a stack plus registers. Same-process switches don't reload page tables, so the TLB stays valid.”
2. **Context switch:** “Save A into the PCB, pick B, restore B, jump to B's PC. The microseconds are not the story — cache and TLB pollution are.”
3. **Zombie vs orphan:** “Zombie: child gone, parent hasn't `wait`ed, PCB holds the exit code. Orphan: parent gone, PID 1 adopts and reaps.”
4. **SJF:** “Optimal average wait, but we don't know bursts. We predict with exponential averaging. SRTF is the preemptive version.”
5. **RR quantum:** “Too small, switches dominate. Too large, it's FCFS. Size it so about 80% of bursts finish in one quantum.”
6. **MLFQ:** “New work starts at the top. Full quantum demotes you (CPU-bound). Blocking keeps you (interactive). Aging boosts everyone so the bottom doesn't starve.”
7. **Pathfinder:** “Low-priority task held a mutex the high-priority bus task needed. Medium tasks ran. Priority inheritance: the holder runs at the waiter's priority until it unlocks.”
8. **Mutex vs sem:** “Mutex has an owner — only the locker unlocks. Semaphore is a counter any thread can signal. Mutex for exclusion, sem for signaling or N resources.”
9. **P-C order:** “Resource semaphores before the mutex. Hold the mutex while waiting for empty/full and the other side can never free a slot.”
10. **Dining:** “All pick left, all wait right — circular wait. Break it with asymmetry, at most four, atomic both, or global chopstick order.”
11. **Coffman + Banker's:** “All four conditions, remove one. Banker's grants only if a safe sequence still exists. Unsafe might deadlock later; safe never does. Real kernels don't use it — they can't know max claims.”
12. **VA→PA:** “Split p and d, TLB, else walk the multi-level table. Valid=0 is a fault: validate, evict if needed, read disk, set PTE, **restart the instruction**.”
13. **Belady:** “FIFO can fault *more* with *more* frames. LRU and OPT are stack algorithms — that cannot happen.”
14. **Thrashing:** “Working set doesn't fit. You page instead of run. CPU looks idle, the scheduler admits more jobs, it gets worse. Suspend processes or add RAM.”
15. **COW:** “`fork` marks pages read-only and shared. First write copies that frame. `fork`+`exec` never copies the parent's data.”
16. **Syscall:** “Libc loads args, `syscall` traps into the kernel, dispatcher indexes the table, handler validates, return in a register, back to user mode.”
17. **Hard vs symlink:** “Hard link is another name for the same inode; can't cross filesystems. Symlink is a path string; can dangle, can cross filesystems.”
18. **SCAN vs C-SCAN:** “SCAN services both ways, so the middle is hit twice per sweep. C-SCAN services one way and jumps — waiting time is more uniform.”
19. **DMA:** “CPU programs the controller once. The device writes memory itself. One interrupt at the end. Not a million loads.”
20. **fsync:** “`write` hits the page cache. Durability is `fsync`. Journaling makes the metadata atomic; LFS makes every write sequential.”

---

# Weekly 15-minute emergency pass (if you only have one sitting)

Recite in this order, eyes closed:

1. Program / process / thread → PCB fields → 5 states + verbs → 8-step context switch
2. fork/exec/wait → zombie / orphan → COW → user vs kernel threads → Linux 1:1
3. Wait / TAT / response / throughput → FCFS convoy → SJF τ → SRTF Gantt → RR 80% → aging
4. MLFQ 4 rules → RMS vs EDF → inversion + inheritance
5. Race LOAD/ADD/STORE → CS 3 → mutex vs sem vs spin → P-C order → R-W starve → dining 4
6. Peterson / TAS / CAS
7. Coffman → RAG cycle rule → prevent/avoid/detect → Banker's P1 P3 P4 P0 P2
8. Internal vs external → (p,d) 20+12 → PTE bits → 4-level → TLB + ASID
9. Fault 8 steps → FIFO 9 / OPT 7 / Clock → Belady → thrashing WS
10. Syscall 9 steps → IRQ/trap/exception → mono/micro/hybrid
11. 5 IPC → SIGKILL/STOP → 3 fd tables → inode 12/1/1/1 → hard vs soft → open walk
12. Seek dominates → SSTF starve → SCAN vs C-SCAN → SSD NOOP → DMA → journal + fsync → CFS + OOM + event loop
