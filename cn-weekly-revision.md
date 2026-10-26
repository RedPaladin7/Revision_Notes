# Computer Networks Interview — Weekly Revision Guide

Placement recitation for a fourth-year interview loop (product and campus). Same shape as the OS and DBMS weekly guides. Use this **every week** as a timed recitation pass. Every critical fact is here; examples are compressed, not dropped.

**How to use (one full pass per week, ~40 min/day):**

| Day | Cover | Recite until you can say it without looking |
|-----|--------|---------------------------------------------|
| **Mon** | Models, delay, switching, devices | 5-layer vs OSI, 4 delays, BDP, hub/switch/router |
| **Tue** | Link, ARQ, MAC, Ethernet, ARP | CRC, GBN vs SR windows, CSMA/CD vs CA, ARP for the gateway |
| **Wed** | IP, subnetting, NAT, fragments, IPv6 | Header fields, /26 cuts, longest prefix, 4000→1500 fragments |
| **Thu** | Routing | Count-to-infinity, Dijkstra steps, RIP / OSPF / BGP |
| **Fri** | UDP + TCP reliability and flow | 4-tuple, handshake, TIME_WAIT, rwnd 0, Nagle |
| **Sat** | Congestion + DNS / HTTP / TLS | Tahoe vs Reno, DORA, DNS walk, TLS 1.3 |
| **Sun** | URL walk, TCP states, leftovers + cheat sheet | Type-a-URL in order, state machine, full recitation |

Each day: **Recite** the boxed facts → **Scan** the section → **Answer** the self-quiz at the end of that day. Sunday: run the full cheat sheet out loud.

---

# DAY 1 — Models, Delay, Switching, Devices

## What a network guarantees (and what IP does not)

The Internet is a **best-effort packet network**. IP delivers datagrams with no promise of delivery, order, integrity of the payload, or delay. Reliability, order, and congestion response are **end-to-end**, mostly TCP, when the app asks for them.

**Interview line:** “The network moves packets. TCP, not IP, is what makes a byte stream reliable.”

## Three stacks you must be able to draw

Interviewers mix these. Draw the one they name, then map.

**Kurose 5-layer** (what most courses mean by “the stack”):

| # | Layer | PDU | Job | Typical |
|---|---|---|---|---|
| 5 | Application | message | What the app means | HTTP, DNS, SMTP, DHCP |
| 4 | Transport | segment | Process-to-process | TCP, UDP |
| 3 | Network | datagram | Host-to-host, routing | IP, ICMP |
| 2 | Link | frame | One hop, NIC to NIC | Ethernet, Wi‑Fi, ARP |
| 1 | Physical | bit | Signal on the medium | Cable, fiber, radio |

**TCP/IP 4-layer:** link (physical folded in), internet, transport, application.

**OSI 7-layer:** inserts **Session** and **Presentation** between transport and application. Session = dialog / checkpoints. Presentation = encoding, compression, encryption *in the OSI story*.

Where people get marked wrong:

| Thing | Say this |
|---|---|
| **ARP** | Link layer. It is a broadcast frame on one LAN. Routers do not forward it. It *serves* IP. |
| **ICMP** | Network layer, IP protocol **1**. Companion of IP, not a transport protocol. |
| **DHCP** | Application (it configures an IP address; the protocol itself is still app-over-UDP). |
| **OSPF** | Control plane directly over IP, protocol **89**. Not TCP, not UDP. |
| **BGP** | Application in the stack diagram, **TCP 179**. |
| **RIP** | UDP **520**. |
| **TLS** | Between HTTP and TCP in real life. If they force an OSI box, presentation — then say where it actually sits. |
| **Switch** | Link. **Router** = network. **Hub** = physical. |

## Encapsulation — say it as a walk down

Sending host, top to bottom:

1. App writes bytes.
2. Transport adds **TCP/UDP header** → segment. Port numbers live here.
3. Network adds **IP header** → datagram. Addresses live here.
4. Link adds **frame header + trailer** → frame. MACs and CRC live here.
5. Physical puts **bits** on the wire.

Each hop’s router: strip the frame, read IP, decrement TTL, build a **new frame** for the next link (new MACs, new CRC). The IP packet is what survives the path. The frame is hop-local.

A switch does not open the IP header. A hub does not open the frame.

## Addressing at each layer

| Layer | Address | Scope |
|---|---|---|
| App | URL, email, cookie | The service |
| Transport | Port (16-bit) | A process on a host |
| Network | IP | A host interface, globally routed (or NAT’d) |
| Link | MAC (48-bit) | A NIC on **this** LAN |

**Socket** = IP + port. **TCP connection** = 4-tuple `(src IP, src port, dst IP, dst port)`.

MAC is not who you are on the Internet. Next-hop MAC changes every hop. Destination IP does not, until NAT.

## Four delays — nodal delay

At **one router, one packet**:

```
d_nodal = d_proc + d_queue + d_trans + d_prop
```

| Delay | Formula | Depends on | Depends on traffic? |
|---|---|---|---|
| **Processing** | — | Header check, forwarding lookup | No (μs, ignore in problems) |
| **Queuing** | — | How full the buffer is | **Yes. This is congestion.** |
| **Transmission** | **L / R** | Packet length L (bits), link rate R (bps) | No |
| **Propagation** | **d / s** | Distance d, speed in medium s (~2×10⁸ m/s) | No |

Traps, say them cold:

- Longer **packet** raises transmission delay. Propagation unchanged.
- Longer **cable** raises propagation delay. Transmission unchanged.
- Faster **link** lowers transmission delay. Propagation unchanged.
- Queuing is the only one that grows when the network is busy.

**Throughput** of a path = rate of the **bottleneck** link (the minimum R), once the pipe is full. **Goodput** = useful application bytes per second (headers and retransmits removed).

**Latency** is delay. **Bandwidth** is capacity. A truck full of disks has enormous bandwidth and terrible latency.

## Bandwidth-delay product

```
BDP = R × RTT     (bits in flight that fill the pipe)
```

The sender’s window must be **at least one BDP** or the link sits idle waiting for ACKs.

```
R = 1 Gbps,  RTT = 30 ms,  packet = 1250 bytes

Tt = (1250 × 8) / 1e9 = 10 μs
BDP = 1e9 × 0.030 = 3×10⁷ bits = 3.75 MB
packets to fill the pipe = 3.75 MB / 1250 B = 3000
```

One packet in flight uses 10 μs of a 30 ms RTT → utilization ≈ **0.03%**. This is why stop-and-wait dies on fast, long links, and why the TCP window (and the window-scale option) exists.

## Switching

| | Circuit | Packet | Virtual circuit |
|---|---|---|---|
| Path | Reserved end-to-end before data | None; each packet routed alone | Setup, then packets carry a label |
| Idle capacity | Wasted | Shared (statistical multiplexing) | Reserved or soft-reserved |
| Delay | Setup, then tiny and stable | Store-and-forward + queue (variable) | Setup + smaller per-packet header |
| Example | Old telephone | **The Internet (IP)** | MPLS, old ATM/frame relay |

**Store-and-forward:** a router must receive the **whole** packet before it can start sending it on the next link. One packet, N links of rate R, no other delay: transmission component = **N × L/R**.

Cutting a file into packets **pipelines**: hop 2 forwards packet 1 while the source sends packet 2. For a file of F bits, packet L, N links:

```
time ≈ F/R + (N − 1) × (L/R)     (+ propagation)
```

Message switching (whole file as one unit) cannot pipeline. Packet switching can.

## Devices — collision domain vs broadcast domain

| Device | Layer | What it reads | Collision domains | Broadcast domains |
|---|---|---|---|---|
| **Hub / repeater** | 1 | Bits only | **1** shared by all ports | 1 |
| **Switch / bridge** | 2 | MAC | **1 per port** | 1 (per VLAN) |
| **Router** | 3 | IP | 1 per port | **1 per port** |

- **Hub** repeats every bit out every port. One big wire. Useless above legacy questions.
- **Switch** learns **source MAC → port**, forwards known unicast to that port, **floods** unknown unicast, broadcast, and unknown multicast.
- **Router** forwards between subnets, **does not forward** broadcasts or ARP, decrements TTL, rebuilds the frame.
- **Default gateway** = the router IP on **your** subnet. Not a different kind of device.
- A **gateway** in old OSI vocabulary means an application-layer translator. In every practical interview, “gateway” means that router IP.

**Broadcast domain** = who hears a broadcast (ARP, DHCP discover). Routers are the walls. Switches are not, until VLANs.

### Day 1 self-quiz
1. Draw 5-layer, TCP/IP, and OSI. Where do ARP, ICMP, DHCP, OSPF, BGP, TLS sit?
2. Encapsulation on the sender. What does a router change, and what does a switch refuse to look at?
3. Four delays: formula, and which three changes (longer packet, longer cable, faster link) hit which delay.
4. Compute BDP: 1 Gbps, 30 ms RTT. How many 1250-byte packets fill the pipe? Why is stop-and-wait ~0% utilization?
5. Circuit vs packet vs virtual circuit in one line each. Store-and-forward formula for one packet over N hops.
6. Hub vs switch vs router: layer, collision domains, broadcast domains. What is a default gateway?

---

# DAY 2 — Link Layer, ARQ, MAC, Ethernet, ARP

## What the link layer is for

One hop: **framing**, **MAC** (who may send), **error detection**, sometimes **reliable delivery** and **flow control**.

Reliability at layer 2 is **optional**. Fiber is clean, so Ethernet mostly just CRCs and drops. Wi‑Fi is dirty, so 802.11 ACKs every frame. End-to-end reliability is still TCP’s job — a CRC on one hop does not protect you two hops later against a router bug, and a dropped Wi‑Fi frame is recovered locally only if that hop does ARQ.

## Error detection

| Scheme | Idea | Catches | Used by |
|---|---|---|---|
| **Parity** (even/odd) | 1 bit, XOR of data | Odd number of flips. 2 flips look valid | Teaching, some buses |
| **2D parity** | Row + column parity | Can **locate** a single-bit error | Teaching |
| **Internet checksum** | 16-bit **one’s complement** sum | Many errors, not bursts reliably | **IP header, TCP, UDP** |
| **CRC** | Polynomial division over GF(2) | All bursts of length ≤ r (r = CRC bits) | **Ethernet FCS = CRC-32** |

Checksum verify: sum of words **including** the checksum, one’s complement, should be **0xFFFF**.

IP checksum covers the **header only**, and routers **recompute it every hop** because TTL changes. TCP/UDP checksum covers a **pseudo-header** (src IP, dst IP, protocol, length) **plus** header **plus** payload, so a packet delivered to the wrong machine fails the check.

**Interview line:** “CRC on the frame, Internet checksum in IP/TCP/UDP. Different algorithms, different jobs. NAT must recompute **both** the IP checksum and the transport checksum, because it changed addresses that sit inside the pseudo-header.”

## Reliable delivery over one link — three ARQ schemes

Sequence numbers exist so the receiver can tell a **retransmission** from a **new** frame.

### Stop-and-wait

Send one. Wait for ACK. Timer → send again.

```
a = Tp / Tt
η = 1 / (1 + 2a)          error-free
η = (1 − P) / (1 + 2a)    frame loss probability P
```

Needs **2 sequence numbers** (1 bit: 0 and 1). With only “frame / ACK” and no number, a lost ACK makes the retransmit look like the next new frame.

Utilization collapses when `a` is large (the Day 1 gigabit example).

### Go-Back-N

- Sender window **Ws ≤ 2ᵐ − 1** (m-bit sequence numbers).
- Receiver window **= 1**.
- **Cumulative ACK.** ACK n means “everything up to n is in.”
- Out-of-order frame: **discard**, re-ACK the last in-order one.
- Timeout: retransmit the lost frame **and everything after it** still in the window.

### Selective Repeat

- Sender window and receiver window **both ≤ 2ᵐ⁻¹**.
- Receiver **buffers** out-of-order frames.
- ACK is per frame (or a bitmap). Retransmit **only** the loss.
- Timer per outstanding frame, in the full scheme.

**Why SR’s window is half the sequence space.** Sequence space 0..3 (m=2), window 3: a delayed old frame can land inside the new window and be accepted as fresh. Window 2 cannot. Same rule for any m: **Ws ≤ 2ᵐ⁻¹**.

| | Stop-and-wait | GBN | SR |
|---|---|---|---|
| Sender window | 1 | ≤ 2ᵐ − 1 | ≤ 2ᵐ⁻¹ |
| Receiver window | 1 | 1 | ≤ 2ᵐ⁻¹ |
| On loss | Retransmit the one | Retransmit from the hole to the end | Retransmit the hole |
| Receiver buffer | No | No | Yes |
| m = 3 (seq 0–7) | — | Ws ≤ **7** | Ws ≤ **4** |

If the window is big enough to fill the pipe (`Ws ≥ 1 + 2a`), error-free utilization hits 1. Otherwise **η = Ws / (1 + 2a)**.

GBN wastes the pipe when loss is common (it rewinds). SR does not. TCP’s byte stream is closer to GBN with cumulative ACKs; **SACK** makes it behave like SR.

## Who gets the wire — MAC

**Channel partition** (TDMA, FDMA, CDMA): a slice each, idle slices wasted, no collisions.

**Random access:** send when you want, recover from collisions.

**Taking turns:** polling or token. No collision, a token/poll cost. Interview mention is enough.

### ALOHA

| | When you send | Vulnerable period | Max throughput |
|---|---|---|---|
| **Pure** | Anytime | **2 × frame time** | **1/(2e) ≈ 18.4%** at G = 0.5 |
| **Slotted** | Only at a slot boundary | **1 × frame time** | **1/e ≈ 36.8%** at G = 1 |

G = offered load (average transmission attempts per frame time), including retransmits. Past the peak, more load → more collisions → throughput falls. That collapse is the argument for carrier sense and for congestion control.

### CSMA / CD (wired Ethernet, half duplex)

Carrier sense: don’t start talking if the wire is busy.

Collision **detection**: you can hear a collision **while you are still transmitting**, because the wire carries your signal and theirs at once.

1. Sense. If busy, wait (persistent CSMA: keep listening; non-persistent: wait a random time and sense again).
2. Send.
3. If a collision is heard, send a **32-bit jam**, abort.
4. **Binary exponential backoff.** After the nth collision, k = min(n, 10). Pick uniformly in `0 .. 2ᵏ − 1` slot times. Give up at **16** attempts.

**Why a minimum frame length exists.** The sender must still be transmitting when the collision noise gets back from the far end:

```
Tt ≥ 2 × Tp          Lmin = R × (worst-case RTT)
```

Classic 10/100 Mbps half-duplex: slot = **512 bit times**, minimum frame = **64 bytes** (without the preamble). At 10 Mbps that slot is 51.2 μs, which capped cable length.

**Faster Ethernet killed CSMA/CD.** At 1 Gbps the minimum frame for a long cable would be huge (gigabit half-duplex specified carrier extension; almost nobody deployed it). Real Ethernet is **switched full duplex**: one sender per direction per link, **no collisions**, CSMA/CD unused. The 64-byte minimum and 1500-byte MTU stayed.

### CSMA/CA (Wi‑Fi) — you cannot CD on a radio

A radio cannot transmit and listen for a collision at the same time usefully, and a collision may happen at the **receiver**, not at the sender.

- **DIFS** idle, then random backoff, then send. Backoff pauses when the channel goes busy.
- Every unicast data frame gets a link-layer **ACK** (SIFS, shorter, so ACKs win the channel). No ACK → retransmit.
- **RTS/CTS** optional, for large frames: sender asks, AP/receiver grants, everyone who hears either frame sets **NAV** and stays quiet. Fixes the **hidden terminal**.

**Hidden terminal:** A and C both reach B, A and C do not hear each other. Both send, collide at B. RTS/CTS lets B’s CTS silence C.

**Exposed terminal:** B is sending to A. C hears B and defers, even though C→D would not collide at A. CA is conservative; exposed terminals waste capacity. Say the geometry if asked; don’t pretend RTS/CTS fixes exposed terminals. It fixes hidden ones.

## Ethernet frame (DIX / Ethernet II)

| Field | Size |
|---|---|
| Preamble + SFD | 7 + 1 (not counted in the 64) |
| Dest MAC, Src MAC | 6 + 6 |
| EtherType | 2 |
| Payload | **46–1500** |
| FCS (CRC-32) | 4 |

Minimum frame **64 bytes**, maximum **1518**, without the preamble. Payload padded to 46 if the IP packet is smaller.

EtherTypes to remember: **0x0800** IPv4, **0x0806** ARP, **0x86DD** IPv6, **0x8100** is an 802.1Q tag (then the real EtherType follows).

**MAC:** 48 bits. First 24 = OUI (vendor). I/G bit = 1 means multicast. **Broadcast = FF:FF:FF:FF:FF:FF.** A switch floods that. A router does not route it.

## Switch behavior

1. On receive, learn **source MAC → ingress port**. Refresh the aging timer.
2. Destination known unicast → that port only.
3. Unknown unicast, broadcast, unknown multicast → **flood** every other port in the VLAN.
4. Never forward a frame back out the port it arrived on.

**Why STP exists.** Ethernet frames have **no TTL**. A layer-2 loop forwards broadcasts forever: a **broadcast storm**, MAC table flapping as the same source appears on two ports. **Spanning Tree** elects a root bridge (lowest bridge ID) and blocks redundant ports so the active topology is a tree. Classic 802.1D sat in listening + learning for ~30 s before forwarding; **RSTP** converges in seconds. Interview depth stops there unless they push.

## VLAN

One physical switch, many broadcast domains. **802.1Q** inserts 4 bytes (TPID 0x8100 + 12-bit VLAN ID → 4094 usable VLANs). Tagged frame max **1522**.

- **Access port:** one VLAN, frames untagged toward the host.
- **Trunk:** many VLANs, tagged, between switches.
- Two VLANs do not share broadcasts. Traffic **between** VLANs needs a router or a layer-3 switch (router-on-a-stick if it’s one physical router port with subinterfaces).

## ARP — IP to MAC, on this LAN only

Host wants to send an IP packet to someone on **its own subnet**:

1. Compare `(dst IP & mask)` with `(my IP & mask)`.
2. Same subnet → ARP for the **destination** MAC.
3. Different subnet → ARP for the **default gateway’s** MAC. The IP destination stays the real remote host. The frame’s dest MAC is the router.

ARP request: broadcast Ethernet, “who has 192.168.1.1? tell 192.168.1.5”. Reply: unicast. Both sides cache. Entries time out.

**Gratuitous ARP:** a host announces its own IP→MAC (comes up, or failover). Updates neighbors’ caches; can detect a duplicate IP.

ARP is not encrypted and not authenticated. On a LAN, anyone can answer. That is **ARP spoofing**: poison the cache, become the gateway, MITM. Switches that do **dynamic ARP inspection** drop replies that don’t match the DHCP binding.

### Day 2 self-quiz
1. Checksum vs CRC: who uses which, what the IP checksum covers, why NAT touches two checksums.
2. Window sizes for m = 3: GBN and SR. Why is SR half the sequence space? What does the receiver do with an out-of-order frame in each?
3. Stop-and-wait η in terms of `a`. Why 2 sequence numbers?
4. Pure vs slotted ALOHA max throughput. CSMA/CD backoff rule. Why Lmin = R × RTT, and why gigabit Ethernet dropped CD.
5. Hidden vs exposed terminal. What does RTS/CTS actually fix?
6. Draw the Ethernet frame sizes. Switch learn/flood rules. Why STP. ARP: when do you resolve the gateway instead of the destination?

---

# DAY 3 — IPv4, Subnetting, NAT, Fragmentation, IPv6

## Forwarding vs routing (one sentence, then Day 4)

**Forwarding** is the data plane: this packet, this router, longest-prefix lookup, send it out one interface. **Routing** is the control plane: building the table. Don’t mix the words.

## IPv4 header — fields they actually ask

Minimum **20 bytes**. IHL is the header length in 32-bit words (5 = 20 bytes). If IHL > 5 there are options; that is rare and disliked by routers.

| Field | Remember |
|---|---|
| **Total length** | Header + payload. 16-bit → max datagram **65535**. |
| **Identification** | Same value on **every fragment** of one datagram. |
| **Flags** | DF = don’t fragment. MF = more fragments follow. |
| **Fragment offset** | In **8-byte units**, from the start of the **original payload**. |
| **TTL** | Hop count. Router decrements. Hits 0 → drop + **ICMP Time Exceeded**. Not seconds. |
| **Protocol** | **1** ICMP, **6** TCP, **17** UDP, **89** OSPF. Tells the receiver who gets the payload. |
| **Header checksum** | Header only. Recomputed every hop. |
| **Src / Dst** | 32-bit. What routing looks at. Dst does not change along the path (NAT is the exception). |

**TTL is how traceroute works.** Send packets with TTL = 1, 2, 3, … The router that decrements it to 0 replies ICMP Time Exceeded, and that reply’s source IP is the hop. Linux `traceroute` sends **UDP** to a high port (the last hop often answers ICMP port unreachable). Windows `tracert` sends **ICMP echo**. Same TTL trick.

**Ping** is ICMP **echo request (type 8) / echo reply (type 0)**. “Ping a port” in casual speech is a TCP connect, which is not ping.

## Special addresses

| Address | What |
|---|---|
| **0.0.0.0** | “This host”, unknown source; also the **default route** 0.0.0.0/0 |
| **127.0.0.0/8** | Loopback. Never on the wire. |
| **255.255.255.255** | Limited broadcast. This LAN. Routers do not forward. |
| **x.x.x.255** in a /24 (host bits all 1) | Directed subnet broadcast |
| **169.254.0.0/16** | Link-local. DHCP failed (APIPA). |
| **224.0.0.0/4** | Multicast (class D) |
| **10.0.0.0/8**, **172.16.0.0/12**, **192.168.0.0/16** | **Private.** Routable inside, not on the public Internet. |

Classful, only because they still say “class C”:

| Class | First bits | Range | Default mask |
|---|---|---|---|
| A | 0 | 0–127 | /8 |
| B | 10 | 128–191 | /16 |
| C | 110 | 192–223 | /24 |
| D | 1110 | 224–239 | multicast |
| E | 1111 | 240–255 | reserved |

127 is inside class A’s numeric range and is still loopback. Real networks are **CIDR**: the mask is explicit, class is irrelevant.

## Subnetting — do it mechanically

```
hosts needed H  →  smallest h with 2^h − 2 ≥ H
prefix          →  32 − h
block size      →  2^h addresses
usable          →  network+1  …  broadcast−1
broadcast       →  network + block − 1
```

The **interesting octet** is the one the mask only partly covers. If prefix = 8k + r, block size in octet k+1 is **2^(8−r)**.

Mask ↔ prefix, fourth octet, memorize:

| Prefix | Mask ends | Block |
|---|---|---|
| /24 | 0 | 256 |
| /25 | 128 | 128 |
| /26 | 192 | 64 |
| /27 | 224 | 32 |
| /28 | 240 | 16 |
| /29 | 248 | 8 |
| /30 | 252 | 4 (2 usable) |
| /31 | 254 | 2 usable, **no** network/broadcast (point-to-point, RFC 3021) |
| /32 | 255 | one host route |

**−2** is the network address and the broadcast address. It does **not** apply to /31 or /32.

### Worked: split 192.168.1.0/24 into subnets of at least 50 hosts

50 hosts → h = 6 because 2⁶ − 2 = 62 (h = 5 gives 30, too small). Prefix **/26**. Block = 64. Four subnets:

| Network | Usable | Broadcast |
|---|---|---|
| 192.168.1.0/26 | .1 – .62 | .63 |
| 192.168.1.64/26 | .65 – .126 | .127 |
| 192.168.1.128/26 | .129 – .190 | .191 |
| 192.168.1.192/26 | .193 – .254 | .255 |

### Worked: where does 172.16.5.20/20 sit?

/20 → 4 bits into the third octet → block of **16** in that octet. 5 falls in **0–15**.

- Network **172.16.0.0/20**
- Broadcast **172.16.15.255**
- Usable **172.16.0.1 – 172.16.15.254**
- Mask **255.255.240.0**

### Worked: aggregate

10.1.4.0/24, 10.1.5.0/24, 10.1.6.0/24, 10.1.7.0/24 → **10.1.4.0/22** (third-octet block of 4, aligned on 4).

10.1.3.0/24 does **not** join them. A /22 covering 3 would have to start at 0. Aggregation has to land on the bit boundary or you black-hole someone else’s prefix.

## Longest prefix match

The forwarding table can match several prefixes. **The longest one wins.** Default route 0.0.0.0/0 matches everything and loses to any real prefix.

```
192.168.0.0/16      → R1
192.168.1.0/24      → R2
192.168.1.128/25    → R3
0.0.0.0/0           → R4

192.168.1.200  →  /16, /24, and /25 all match  → R3
192.168.1.10   →  /25 starts at .128, so no    → R2
10.0.0.1       →  only the default             → R4
```

Say the method: write each prefix, test membership, keep the biggest mask length. Don’t eyeball.

## NAT and PAT

Private hosts share one public address. The home router rewrites:

- Out: source IP → public IP, source port → a chosen public port. Record `(priv IP, priv port, pub port, dst IP, dst port)`.
- In: the reply’s destination hits that public port. Rewrite back to the private host.

That is **PAT** (one public IP, many hosts), which is what everyone means by NAT.

Consequences, say all three:

1. **Outbound works, unsolicited inbound does not.** There is no table row until an inside host speaks. A server behind NAT needs a **port forward**, or the inside host must punch out first.
2. **End-to-end addressing is gone.** The address a server sees is the router, not the laptop.
3. **Checksums change.** IP header checksum and TCP/UDP checksum both recomputed.

P2P (WebRTC) gets through NAT with **STUN** (learn your public mapping) and **TURN** (relay when punching fails). ICE tries the candidates. One sentence is enough unless the role is networking-heavy.

**Interview line:** “NAT was an address-sharing hack that became architecture. It is why IPv4 survived and why ‘just listen on a port’ fails on a laptop.”

## ICMP — the messages that show up in interviews

Carried in IP, protocol 1. Not TCP, not UDP.

| Type | Name | Who uses it |
|---|---|---|
| 8 / 0 | Echo request / reply | ping |
| 3 | Destination unreachable | no route, host down, **port unreachable**, **fragmentation needed but DF set** (code 4, PMTUD) |
| 11 | Time exceeded | TTL hit 0 (**traceroute**), or fragment reassembly timed out |
| 5 | Redirect | router tells a host a better first hop. Rare, often ignored, easy to abuse |

## Fragmentation

IPv4 routers **may** fragment if the next-link MTU is smaller and **DF = 0**. Fragments are reassembled **only at the destination**. Routers do not reassemble. Lose one fragment and the destination timer fires, the whole datagram is dropped, ICMP time exceeded may be sent. Identification is identical on all fragments. Offset is absolute from the original payload, so a fragment can be fragmented again.

```
non-final payload = largest multiple of 8 that fits in (MTU − header)
offset field      = (bytes of original payload before this fragment) / 8
MF                = 1 on every fragment except the last
```

**MTU 1500, header 20, total length 4000 (payload 3980).** 1480 is divisible by 8, so no rounding.

| Frag | Payload | Total length | Offset | MF |
|---|---|---|---|---|
| 1 | 1480 | 1500 | 0 | 1 |
| 2 | 1480 | 1500 | 1480/8 = **185** | 1 |
| 3 | 1020 | 1040 | 2960/8 = **370** | 0 |

1480+1480+1020 = 3980. Check that before you finish.

**Rounding trap:** MTU 506, header 20 → 486 bytes of room, 486 is not a multiple of 8, so a non-final fragment carries **480** bytes, offset steps of 60.

**MTU vs MSS.** Ethernet MTU 1500 is the largest **IP packet**. TCP MSS is the largest **TCP payload**:

```
MSS = MTU − IP header − TCP header = 1500 − 20 − 20 = 1460
```

PPPoE adds 8 bytes, MTU often 1492, MSS 1452. Path MTU can be smaller than the local Ethernet MTU (tunnels).

**PMTUD:** send with **DF = 1**. If a router cannot forward, it drops and sends ICMP type 3 code 4 with the next MTU. Sender shrinks. If some firewall **eats that ICMP**, the sender never learns — a **black hole**. Symptom: small transfers work (DNS, small HTTP), large ones hang. IPv6 has no router fragmentation, so PMTUD is mandatory.

## IPv6 — differences, not the address format alone

| | IPv4 | IPv6 |
|---|---|---|
| Address | 32-bit | **128-bit** |
| Header | 20 bytes, variable options | **Fixed 40 bytes**, extension headers |
| Checksum | Header checksum every hop | **None.** Link CRC + transport checksum |
| Fragment | Routers may | **Sender only.** Routers drop and send ICMPv6 Packet Too Big |
| Broadcast | Yes | **No.** Multicast and anycast |
| ARP | ARP | **NDP** (Neighbor Solicitation / Advertisement), ICMPv6 |
| Autoconfig | DHCP | **SLAAC** from Router Advertisements, plus DHCPv6, plus privacy addresses |
| NAT | Everywhere | Designed so you don’t need it. Happens anyway in some networks |

Addresses they ask: **::1** loopback, **fe80::/10** link-local, **fc00::/7** unique local (the private range), **2000::/3** global unicast, **ff00::/8** multicast.

Compression: drop leading zeros in a group; replace **one** run of all-zero groups with `::`. `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:db8::1`.

### Day 3 self-quiz
1. Recite the IPv4 fields that change or matter in transit: TTL, checksum, fragmentation trio, protocol numbers.
2. How do traceroute and ping differ (Linux vs Windows traceroute)?
3. Split 192.168.1.0/24 for ≥50 hosts. Place 172.16.5.20/20. Why can’t 10.1.3.0/24 join 10.1.4.0–10.1.7.0/24?
4. Longest prefix: next hop for .200, .10, and 10.0.0.1 in the worked table.
5. What does the NAT table store? Why doesn’t an inbound connection work? Which checksums change?
6. Fragment 4000 bytes into MTU 1500. State offset, MF, total length. Where is reassembly done? What does DF + filtered ICMP break?

---

# DAY 4 — Routing

## The split

| | Forwarding | Routing |
|---|---|---|
| Plane | Data | Control |
| When | Every packet | When topology or policy changes |
| Question | Which interface for **this** dst? | What should the table contain? |
| Implementation | Longest prefix, hardware | RIP, OSPF, BGP, static config |

**Static route:** a human typed it. No adaptation. Right tool for a stub network and for the default route. Wrong tool as the only mechanism in a network that fails over.

**Dynamic:** routers exchange information and recompute. Costs convergence time and the chance of loops while they disagree.

## Distance vector — Bellman-Ford

Each router knows the cost to its **neighbors**, and periodically receives each neighbor’s **distance vector** (cost from that neighbor to every destination).

```
d_x(y) = min over neighbors v of { c(x, v) + d_v(y) }
```

The router does not know the path. It knows a next hop and a distance. Updates are periodic and **triggered** (on a change).

### Count-to-infinity

A—B—C, each link cost 1. B’s route to C is cost 1. A’s route to C is cost 2 **via B**.

Link B–C dies. B should set C to ∞. Before that advertisement lands, B still has A’s old vector in hand: “I can reach C at cost 2.” B now thinks C is cost 3 **via A**. A then adopts cost 4 via B. They climb 3, 4, 5… until they hit the protocol’s infinity.

**Split horizon:** do not advertise a route back out the interface you learned it from. Stops the two-node version of this loop.

**Poison reverse:** advertise the route back, but with metric **infinity**, so the neighbor is explicitly told “don’t use me for this.”

Both reduce two-node loops. **They do not kill every loop** once three or more routers are involved. That is why distance vector converges slowly and why RIP’s infinity is small.

### RIP

- Distance vector, metric = **hop count**.
- **15** is the maximum, **16 = infinity**. A network wider than 15 hops cannot be described. This caps count-to-infinity time and caps the protocol.
- UDP **520**, updates every **30 s**.
- Dead interval typically 180 s before a neighbor is declared gone.
- Fine for a small, old, or lab network. Not how a campus or a provider is built today. Know it because they ask why it died: hop-count metric, slow convergence, infinity problem.

## Link state — Dijkstra

Each router floods a **link-state advertisement**: “I am X, my neighbors and costs are these.” Flooding is reliable (sequence numbers, acks). Every router in the area ends up with the **same map**, then runs shortest paths **from itself**.

Dijkstra (priorities = cost from source):

```
dist[src] = 0;  dist[others] = ∞
settled = {}
while a node remains:
  u = unsettled node with smallest dist
  settle u
  for each neighbor v of u:
    dist[v] = min(dist[v], dist[u] + cost(u, v))
```

### Worked Dijkstra from A

```
links: A—1—B, A—4—C, B—2—C, B—6—D, C—3—D
```

| Step | Settle | dist A | B | C | D |
|---|---|---|---|---|---|
| init | | 0 | ∞ | ∞ | ∞ |
| 1 | A | 0 | **1** via A | **4** via A | ∞ |
| 2 | B | 0 | 1 | **3** via B | **7** via B |
| 3 | C | 0 | 1 | 3 | **6** via C |
| 4 | D | 0 | 1 | 3 | 6 |

Tree: A→B (1), B→C (2), C→D (3). Distances 0, 1, 3, 6.

Say it as you fill the row: “settling B, C improves from 4 to 1+2=3, D becomes 1+6=7. Settling C, D improves from 7 to 3+3=6.”

Link-state cost is CPU and memory for the map, plus flooding traffic. Win: each router has the topology, convergence is fast, loops are transient only while maps disagree. No count-to-infinity.

### OSPF

- Link state. IP protocol **89** (not UDP/TCP). Has its own reliable flooding and its own acks.
- On broadcast links, routers elect a **DR** and **BDR**. Everyone adjoins the DR, not a full mesh of adjacencies. Cuts LSA chatter.
- Metric: Cisco’s default is `reference bandwidth / link bandwidth` with reference **100 Mbps**, integer, minimum 1. So Fast Ethernet and Gigabit both cost **1** until someone raises the reference. Interviewers like this trap.
- **Areas.** Area 0 is the backbone. ABRs sit on the border and leak **summaries**, not every internal link. Inside an area, full Dijkstra. This is how OSPF scales past “one map of the whole company.”
- Fast convergence, used **inside** an organization (one administrative domain).

## BGP — routing between organizations

OSPF optimizes a metric you control. BGP expresses **policy** between autonomous systems that do not trust each other and do not share one metric.

- **Path vector.** The route carries the **AS-PATH**, the list of ASes to the destination. A router that sees its own AS in the path drops the route. That is the loop prevention. Infinity is unnecessary.
- **TCP 179.** The control plane wants reliability and can afford TCP. Peers are configured by hand; BGP does not discover neighbors by multicast the way OSPF does.
- **eBGP** between ASes (usually on directly connected routers). **iBGP** inside an AS, to spread the routes BGP learned.

**iBGP split horizon:** a route learned from an iBGP peer is **not** re-advertised to another iBGP peer. Otherwise three iBGP routers would loop advertisements with no extra loop check inside the AS. Consequence: iBGP wants a **full mesh**, which is n². Real networks use **route reflectors** (clients may learn reflected routes; the reflector is allowed to re-advertise) or confederations.

### How a BGP router picks one route

Vendor knobs differ. The order interviewers want, for a standard decision process:

1. Highest **LOCAL_PREF** (this AS’s outbound preference; higher wins; stays inside the AS).
2. Shortest **AS-PATH**.
3. Lowest **MED** (a hint to a neighbor about which entry point you prefer; lower wins; easy to get wrong across different neighboring ASes).
4. Prefer **eBGP-learned** over iBGP-learned.
5. Lowest **IGP cost to the next hop** (hot-potato: leave your AS at the nearest exit).
6. Tie breaks: oldest route, then router ID. Don’t lead with these.

Cisco **weight** is local to one router and sits above LOCAL_PREF. It is not an Internet-wide attribute. Mention it only if they say Cisco.

**Hot potato:** forward to the closest exit (IGP cost). Your AS carries the packet a short distance; the other AS carries it the long way. **Cold potato** is the opposite: carry it inside your AS to a better exit. Providers default toward hot potato because internal capacity is what they pay for carefully, and BGP’s step 5 does exactly that.

**Business, not shortest path.** Customers pay providers. Peers exchange without paying. A sane policy: prefer customer routes (they pay you to carry them), then peers, then providers. LOCAL_PREF encodes this. A longer AS-PATH through a customer beats a shorter path through a provider. If you say “BGP picks the shortest path,” you have told them you don’t know BGP.

## Side by side

| | RIP | OSPF | BGP |
|---|---|---|---|
| Algorithm | Distance vector | Link state | Path vector |
| Metric | Hops (max 15) | Link cost | Policy, then AS-PATH length |
| Transport | UDP 520 | IP proto 89 | TCP 179 |
| Scope | Small LAN / legacy | **Inside one AS** | **Between ASes**, and iBGP inside |
| Loop trick | Split horizon, infinity = 16 | Consistent map + Dijkstra | AS-PATH contains self → drop |
| Failure | Slow, count-to-infinity | Fast inside the area | Slow **on purpose**. Stability beats reaction |

**Administrative distance** is a Cisco local tie-break when **two different protocols** offer the same prefix. Lower wins. Connected 0, static 1, eBGP 20, OSPF 110, RIP 120, iBGP 200. It is not a routing algorithm. OSPF will beat RIP onto the table even if the RIP hop count “looks” better, because 110 < 120.

### Day 4 self-quiz
1. Forwarding vs routing. Static vs dynamic in one line.
2. Write the Bellman-Ford update. Tell the A—B—C count-to-infinity story. What do split horizon and poison reverse fix, and what do they leave open?
3. RIP’s infinity, transport, and why hop count is a bad metric.
4. Run the worked Dijkstra out loud and justify the C and D updates.
5. OSPF: protocol number, why a DR exists, what the default Cisco cost does to a 1 Gbps link, what area 0 is for.
6. BGP decision order. Why iBGP needs a full mesh or a route reflector. Hot potato. Why “shortest AS-PATH” is not the first rule.

---

# DAY 5 — UDP, TCP Reliability, Handshake, Flow Control

## Multiplexing

A host runs many processes. The **port** picks the process. The **4-tuple** picks the conversation.

- UDP demux is basically the **destination port** (an unconnected UDP socket).
- TCP demux is the **full 4-tuple**. Thousands of clients can share a server’s port 443; each connection is distinct because the client IP and ephemeral port differ.

| Range | Ports | Who binds them |
|---|---|---|
| Well-known | 0–1023 | Privileged services |
| Registered | 1024–49151 | Assigned apps |
| Ephemeral | 49152–65535 (typical) | The client OS, for one connection |

Ports worth memorizing: **20/21** FTP data/control, **22** SSH, **25** SMTP, **53** DNS, **67/68** DHCP server/client, **80** HTTP, **110** POP3, **123** NTP, **143** IMAP, **443** HTTPS, **520** RIP, **179** is TCP for BGP not a “service port” in the same mental list but say it with RIP.

## UDP

Header **8 bytes**: source port, dest port, length, checksum. Checksum optional on IPv4, required on IPv6. Checksum covers the pseudo-header, so a misdelivered datagram fails it.

No connection, no retransmission, no flow control, no congestion control, no order. Messages have boundaries: one `sendto` is one datagram, and a datagram bigger than the path allows is fragmented or dropped. The application sees message boundaries that TCP will not give it.

Use UDP when **you** want the control, or when a late retransmission is worse than a gap:

- DNS queries, DHCP, NTP (small request/response; the app retransmits)
- VoIP, games, live video (timely beats complete)
- QUIC, and therefore HTTP/3 (they rebuilt reliability and congestion **in user space**, per stream)

UDP can **multicast**. TCP cannot. A TCP sender has one receiver’s window and one ACK clock.

**Interview line:** “UDP is not ‘unreliable TCP.’ It is a demux header and a checksum. Anything else is the application’s problem, including not destroying the network.”

## TCP — a reliable, ordered **byte stream**

TCP does **not** preserve `send()` boundaries. The receiver’s `read()` can return half a write or three writes glued together. The application frames messages (length prefix, or a delimiter, or HTTP’s Content-Length / chunking).

Full duplex: each direction has its own sequence numbers, window, and FIN.

Header minimum **20 bytes**. Fields they point at:

| Field | Point |
|---|---|
| **Sequence number** | Byte number of the **first** byte in this segment. Not a packet counter. |
| **Ack number** | Next byte the sender of this ACK **expects**. Cumulative. |
| **Data offset** | Header length (options exist). |
| **Flags** | SYN, ACK, FIN, RST, PSH, URG |
| **Window** | Receiver’s advertised free buffer, **rwnd**, in bytes. 16-bit. |
| **Checksum** | Pseudo-header + header + data. Required. |
| **Options** | MSS, **window scale**, SACK permitted, timestamps |

SYN consumes **one** sequence number. FIN consumes **one**. A pure ACK consumes **zero**. So the ACK of SYN with seq x is **x+1** even though no data was sent.

**MSS** is announced in the SYN options. Each side uses the smaller of what it sent and what it heard, and it is about payload, not the window.

**Window scale** (in the SYN): the 16-bit window field maxes at 65535 bytes. Any path with BDP above 64 KB — a fast long link — cannot be filled without the scale option, which shifts the window by up to 14 (about 1 GB). Both sides must negotiate it during the handshake. You cannot turn it on later.

## Three-way handshake

```
Client                          Server
  SYN, seq = x          →
                        ←   SYN-ACK, seq = y, ack = x+1
  ACK, seq = x+1, ack = y+1  →
```

x and y are **initial sequence numbers**, chosen unpredictably, not from 0. Reasons: old segments from a previous connection with the same 4-tuple must not look valid, and an off-path attacker should not be able to guess the sequence and inject.

**Why three, not two.** Each side has to know that **its own ISN was received**. After message 2 the client knows the server got x. The server does not know the client got y until message 3. Two messages would leave the server’s sequence unconfirmed, and a duplicated old SYN would be harder to reject.

**State:** client `CLOSED → SYN_SENT → ESTABLISHED`. Server `LISTEN → SYN_RCVD → ESTABLISHED`.

### SYN flood and SYN cookies

The attacker sends SYNs and never ACKs. The server stores half-open state in a backlog until it fills, and real clients get dropped.

**SYN cookies:** the server does not store the half-open connection. It encodes the needed state into the **ISN** of the SYN-ACK (timestamp, MSS, a secret). The legitimate client’s ACK echoes ISN+1, the server reconstructs the state and only then allocates the connection. Tradeoff: some option richness is reduced while cookies are on. Say the idea, not the bit layout.

## Teardown is four segments because each side closes its own stream

```
Active                          Passive
  FIN, seq = u          →          (I am done sending)
                        ←   ACK, ack = u+1     (you may still receive)
                        ←   FIN, seq = v       (now I am done)
  ACK, ack = v+1        →
```

Half-close is real: the passive side can still send after it ACKs the first FIN (client sends a request and FINs; server answers, then FINs). `shutdown(SHUT_WR)` is this.

**TIME_WAIT** is the **active closer**, for **2 × MSL** (spec MSL is often 1–2 minutes; Linux commonly keeps TIME_WAIT around **60 s**).

Two reasons, both mandatory:

1. The **last ACK** may be lost. The passive side will retransmit its FIN. Someone has to be there to ACK it again. That someone is the active side, sitting in TIME_WAIT.
2. Old duplicate segments of this 4-tuple must **die in the network** before the same 4-tuple can be reused. 2 × MSL is the bound on how long a segment can wander.

The server should usually be the **passive** closer. If the server active-closes every short connection, **its** sockets pile up in TIME_WAIT and ephemeral ports get tight. A pile of **CLOSE_WAIT** is different: the peer already FINned, and **this** application has not called `close()`. That is an application bug, not a protocol timeout.

**RST** aborts. No FIN handshake, no TIME_WAIT, unread data discarded. A SYN to a port with nobody listening gets a RST (or nothing, if a firewall drops it).

## Reliability mechanics

- Cumulative ACK: “I have every byte **before** this number, please send this number.”
- Gap: receiver keeps ACKing the **same** number. Those are **duplicate ACKs**. They mean “I am still missing that byte, but later bytes may be arriving” — the network is not completely dead.
- Sender retransmits on **RTO expiry**, or on **3 duplicate ACKs** (fast retransmit, Day 6).
- Retransmitted segment: same sequence number. Receiver drops the duplicate.

**RTO, Jacobson’s estimator.** Don’t use the last sample raw; it jitters.

```
SRTT    = (1 − α) · SRTT + α · SampleRTT          α typically 1/8
RTTVAR  = (1 − β) · RTTVAR + β · |SampleRTT − SRTT|   β typically 1/4
RTO     = SRTT + 4 · RTTVAR
```

On timeout, **double** the RTO (exponential backoff) so a sick network is not hammered. Reset the backoff when an ACK of new data arrives.

**Karn’s algorithm:** if you retransmitted, you don’t know whether the ACK is for the original or the copy. **Do not sample that RTT.** Otherwise one ambiguous sample permanently warps SRTT.

**SACK** (selective acknowledgment): the receiver reports blocks it **does** have, so the sender fills holes instead of rewinding like GBN. Negotiated in the handshake. Essentially universal now. This is the right answer to “does TCP use GBN or SR?” — cumulative ACK is GBN-shaped; SACK makes recovery SR-shaped.

## Flow control — protecting the **receiver**

`rwnd` = free space in the receiver’s socket buffer, advertised in every ACK. Sender may have at most `min(cwnd, rwnd)` bytes unacknowledged. **cwnd** is congestion (Day 6). **rwnd** is the peer’s buffer. Different problems.

**rwnd = 0.** Sender must stop. If the ACK that later reopens the window is **lost**, both sides wait forever: receiver thinks the sender knows, sender thinks the window is still 0. The **persist timer** sends **window probes** (1 byte) so a lost window-update cannot deadlock the connection.

**Silly window syndrome.** Receiver advertises 1 byte, sender sends 1 byte plus 40 bytes of header, the connection crawls. **Clark’s rule** on the receiver: advertise 0 until you can take a full MSS or half the buffer. **Nagle** on the sender: if you already have unacknowledged data, hold small new writes and coalesce them. Nagle plus **delayed ACKs** (receiver waits up to ~200 ms hoping to piggyback the ACK, or ACKs every second segment) can add a painful pause to small RPC writes. Latency-sensitive apps set **TCP_NODELAY** and accept more small packets.

## Building reliability on UDP — the design question

They ask “design a reliable transfer over UDP” to see if you list mechanisms rather than say “I’d use TCP.”

Say the pieces, in order: checksum, sequence numbers, ACK, retransmission timer, a window sized to the BDP, and a congestion reaction so you don’t run open-loop. Then say: a single ordered byte stream with all of that **is TCP**, and the reasons to rebuild it are partial reliability, no head-of-line blocking across streams, user-space control, or connection migration. That rebuild already exists: **QUIC**.

### Day 5 self-quiz
1. What does UDP actually provide? Who should use it? Why can UDP multicast and TCP cannot?
2. Why is TCP a byte stream? What do SYN and FIN do to the sequence number? Why window scale?
3. Draw the handshake. Why three messages? What is a SYN cookie?
4. Draw the four-segment close. Two reasons for TIME_WAIT. CLOSE_WAIT piling up means what?
5. Write the RTO formulas. What does Karn forbid, and why?
6. rwnd vs cwnd. Persist timer. Silly window: Clark vs Nagle. Why TCP_NODELAY.

---

# DAY 6 — Congestion Control, DNS, HTTP, TLS

## Flow control vs congestion control

| | Protects | Signal | Window |
|---|---|---|---|
| **Flow control** | The receiver’s buffer | Explicit `rwnd` | receiver says the number |
| **Congestion control** | The **network** | Implied: loss, and sometimes ECN marks | sender’s **cwnd** |

```
sender window = min(cwnd, rwnd)
```

Congestion control on the Internet is **end-to-end**. Routers mostly drop (or mark). They do not tell you a rate. Senders probe upward and slam backward. **ECN** is the network-assisted version: a router sets a bit instead of dropping, TCP echoes it (ECE/CWR) and reacts as if it saw loss, without the retransmission. Good to name, rarely the whole answer.

UDP has no cwnd. A UDP blast does not share. That is why anything serious on UDP (QUIC) implements congestion control itself.

## AIMD

**Additive increase, multiplicative decrease.** Probe gently (+1 MSS per RTT), back off hard (cut by half) when a loss says the knee was crossed. Two flows doing AIMD converge toward a **fair** share. AIAD and MIMD do not, in the classic Chiu-Jain picture. TCP’s congestion avoidance is AIMD.

## The algorithms — textbook TCP, then what Linux does

`cwnd` below is in **MSS-sized segments**, which is the picture every interviewer draws. Real stacks count **bytes**. Modern Linux initial window is **10 MSS** (IW10), not 1. Problems that say “starts at 1” want the textbook.

**Slow start.** cwnd starts at the initial window. Each ACK grows cwnd by one MSS, so per RTT the window **doubles**. Exponential. Used at the start and after a timeout, until cwnd hits **ssthresh**.

**Congestion avoidance.** Once `cwnd ≥ ssthresh`, grow by about **one MSS per RTT** (each ACK adds `MSS × MSS / cwnd` bytes). Linear. This is the additive increase.

**ssthresh** often starts very high, so the first real event is a loss. The loss **sets** ssthresh.

### Tahoe vs Reno — the thing they make you draw

**Any loss in Tahoe** (timeout **or** 3 dup ACKs):

```
ssthresh = cwnd / 2
cwnd     = 1 MSS
slow start from 1
```

**Reno, 3 duplicate ACKs** (fast retransmit + fast recovery):

```
ssthresh = cwnd / 2
retransmit the missing segment immediately        ← fast retransmit
cwnd     = ssthresh        (do not drop to 1)     ← fast recovery
then congestion avoidance (linear)
```

The +3 MSS inflation in the RFC (cwnd = ssthresh + 3·MSS, then +1 MSS per extra dup ACK, then deflate to ssthresh on the new ACK) is the precise version. The sentence that gets you the mark: **“Three dup ACKs means later packets arrived, so the pipe is not dead. Reno cuts in half and keeps going. A timeout means the pipe looks dead. Reno and Tahoe both go back to 1.”**

**Reno, timeout:** same as Tahoe. ssthresh = cwnd/2, cwnd = 1, slow start. Then linear again once cwnd reaches the new ssthresh.

**NewReno:** Reno leaves fast recovery on the **first** partial ACK. If several segments were lost in the window, that is too early and you timeout anyway. NewReno stays in fast recovery until every byte that was outstanding at the start has been ACKed.

**SACK:** the sender is told the holes and retransmits those, which is what makes NewReno’s recovery actually cheap.

**CUBIC** (Linux default for years): after a loss, cwnd becomes **0.7 × cwnd** (not a half). Growth is a **cubic function of time since the drop**, aimed at the window where the loss happened, then it probes past it. It is not ACK-clocked linear increase. It fills long fat pipes that Reno climbs too slowly. Reno-friendly in the design intent.

**BBR** (Google, used on some servers): measure bottleneck bandwidth and minimum RTT, pace at that rate. **Loss is not the primary signal.** This is the answer to bufferbloat: a huge buffer delays packets for hundreds of milliseconds without dropping, so loss-based TCP never backs off and latency collapses. BBR tries not to fill that buffer.

You will not be asked to derive CUBIC’s K. You will be asked Tahoe vs Reno, and maybe “what does Linux actually run?”

### Worked trace — recite the rows

ssthresh starts at **16**. cwnd in MSS. No delayed ACKs. Textbook IW = 1.

| RTT | Phase | cwnd at end of RTT |
|---|---|---|
| 0 | SS | 1 |
| 1 | SS | 2 |
| 2 | SS | 4 |
| 3 | SS | 8 |
| 4 | SS hits 16 | 16, switch to CA |
| 5 | CA | 17 |
| 6 | CA | 18 |

Loss at cwnd = **20** by **3 dup ACKs, Reno:** ssthresh = **10**, cwnd = **10**, then CA → 11, 12, …

Same loss by **timeout:** ssthresh = **10**, cwnd = **1**, slow start 1, 2, 4, 8, then CA from 10 because `cwnd ≥ ssthresh` switches you over. Don’t double from 8 to 16 and blow past the threshold without saying the rule.

**Rule to say:** while cwnd < ssthresh, double (slow start). While cwnd ≥ ssthresh, add one per RTT.

## Queueing the interviewer might name

- **Drop-tail:** buffer full, drop the arrival. Simple. Synchronizes flows (they all see loss together and all cut — **global sync** — then all climb together).
- **RED:** drop (or mark) randomly as the average queue grows, before it is full, to desynchronize and to signal early.
- **Bufferbloat:** deep buffers turn loss into delay. Throughput looks fine, RTT is seconds, loss-based control is blind. **FQ-CoDel** and similar AQM keep the queue short. Name the phenomenon; don’t pretend you implemented CoDel.

## DNS

A hierarchy of delegations, cached at every level.

**Stub** (your OS) asks a **recursive resolver** (ISP, or 1.1.1.1 / 8.8.8.8): one query, “give me the answer,” RD bit set.

The resolver walks **iteratively**:

1. **Root** (it already knows the root IPs) → referral to the **TLD** servers for `.com`
2. TLD → referral to the **authoritative** servers for `example.com`
3. Authoritative → the record (A, AAAA, …)

Each answer has a **TTL**. The resolver caches. Changing an A record does not move the Internet until caches expire. That is why cutovers are planned around the TTL, and why you lower the TTL **before** the move.

| Record | Holds |
|---|---|
| **A** / **AAAA** | IPv4 / IPv6 of the name |
| **CNAME** | An alias. The lookup continues at the target. Don’t point NS or MX at a CNAME |
| **NS** | Who is authoritative for the zone |
| **MX** | Mail host, with a preference number |
| **PTR** | IP → name (reverse). Independent of the A record; you can have one without the other |
| **TXT** | SPF, DKIM, domain verification, anything text |
| **SOA** | Zone serial, refresh, the negative-cache TTL |

Transport: **UDP 53**. If the answer is too big for a classic 512-byte UDP payload, retry over **TCP 53** (EDNS raises the UDP ceiling; DNSSEC answers often need it). Zone transfer (AXFR) is TCP.

Caches consulted, in order, for a browser: browser cache → OS stub cache → recursive resolver → the walk above. A `hosts` file is in front of DNS on the OS.

**Interview line:** “Recursive is what your laptop asks for. Iterative is how the resolver talks to root, TLD, and authority. The TTL, not the protocol, is why a DNS change feels late.”

## DHCP — DORA

A host that knows nothing, asking for everything. UDP, **client 68, server 67**. First messages are **broadcasts** because the client has no IP and does not know the server.

1. **Discover** — broadcast, “anyone a DHCP server?”
2. **Offer** — a server offers an address, mask, gateway, DNS, lease time
3. **Request** — the client picks one offer and broadcasts the request (so other servers learn they lost)
4. **Ack** — the winner commits the lease

Halfway through the lease the client unicasts a renew. If the server is gone, it rebinds by broadcast near the end of the lease.

Broadcasts do not cross routers. A **relay agent** on the router forwards the discover as unicast to a configured server, with **giaddr** set to the router interface. The server uses giaddr to know which subnet the pool must come from, and replies to the relay.

## HTTP — what the interview is actually about

Stateless request/response. The server does not remember you unless the **application** adds a cookie, a token, or a session id. TCP being a connection is not “HTTP state.”

**Methods:**

| Method | Safe | Idempotent | Use |
|---|---|---|---|
| **GET** | yes | yes | Read. No body semantics. Caches love it |
| **HEAD** | yes | yes | Headers only |
| **PUT** | no | **yes** | Create or **replace** the resource **at this URL** |
| **DELETE** | no | **yes** | Remove this URL |
| **POST** | no | **no** | Process this body. Server picks the new URL if one is created. Not safe to blindly retry |
| **PATCH** | no | no, unless designed to be | Partial update |

Safe means “no user-visible side effect,” which is why caches and prefetchers may issue GET. Idempotent means doing it N times equals doing it once, which is why an automatic retry of PUT is sane and an automatic retry of POST may charge the card twice.

**Status, the ones they ask:**

| Code | Say |
|---|---|
| **200** | OK, body is the resource |
| **201** | Created |
| **204** | Success, no body |
| **301** | Moved permanently. Clients and caches update the URL |
| **302** | Temporary redirect |
| **304** | Not modified. Conditional GET matched ETag / If-Modified-Since. Use the cache |
| **400** | Bad request. The client’s message is broken |
| **401** | **Not authenticated.** Send credentials. The word “Unauthorized” in the spec is a lie; this is unauthenticated |
| **403** | **Authenticated, or irrelevant, and the answer is no.** Don’t retry with the same identity |
| **404** | No such resource. Don’t use it to hide 403 unless you mean to |
| **429** | Too many requests |
| **500** | Server bug |
| **502** | Gateway/proxy got a bad response from upstream |
| **503** | Overloaded or down, maybe Retry-After |
| **504** | Gateway/proxy **timed out** waiting for upstream |

301 vs 302 is cacheability and permanence. 307/308 are the modern “and don’t change POST into GET” versions (307 temporary, 308 permanent).

**Connections, because this ties HTTP to TCP:**

- HTTP/1.0: often a **new TCP handshake per object**. Setup dominates small objects.
- HTTP/1.1: **persistent** connections by default. One handshake, many requests. Browsers open about **6** parallel connections per host.
- **Pipelining** (many requests without waiting, responses still **in order**) lost to head-of-line blocking and broken proxies. Don’t claim the web uses it.
- **HTTP/2:** binary frames, many **streams** on **one TCP**, header compression (HPACK), server push optional. A **TCP loss blocks every stream** on that connection. That is HTTP/2’s head-of-line problem, and it is TCP’s fault, not the framing.
- **HTTP/3:** HTTP over **QUIC** over **UDP**. Independent streams, so a loss stalls one stream. Connection ID instead of the 4-tuple, so a network change (Wi‑Fi → LTE) does not reset the connection. TLS 1.3 is built in. Congestion control lives in user space.

**Cookies:** `Set-Cookie` on the response, `Cookie` on later requests. Flags: **HttpOnly** (JS cannot read it), **Secure** (HTTPS only), **SameSite** (when the browser may attach it on a cross-site request). This is how a stateless protocol carries a session. Say so in one breath.

**Caching:** `Cache-Control`, validators `ETag` / `Last-Modified`, conditional request, **304**. A CDN is a cache that also moved closer to the user.

**WebSocket:** HTTP upgrade, then a long-lived full-duplex framed channel on that TCP connection. Use it for chat and live updates. Don’t use it because polling felt unfashionable and the messages are rare; long-polling or SSE may be simpler.

## TLS — enough to explain HTTPS

HTTPS is HTTP inside TLS, port **443**. The certificate binds a **name** to a public key, signed by a CA the client already trusts. Validation checks the chain, the expiry, and that the name matches the **hostname in the URL** (SNI tells the server which certificate to send; the check is against the name, not the IP).

Handshake goal: **authenticate the server**, agree a **symmetric** key, get **forward secrecy**.

**TLS 1.2** is two round trips before application data: ClientHello; ServerHello + certificate; client key exchange + Finished; server Finished.

**TLS 1.3** is **one** round trip: the ClientHello already carries a key share (ECDHE); the server replies with its key share, certificate, and Finished. **0-RTT** resumption sends data in the first flight. 0-RTT data can be **replayed**, so it is only safe for idempotent requests.

Bulk data is symmetric (AES-GCM and friends). Asymmetric crypto is the handshake only. **Forward secrecy** comes from **ephemeral** Diffie-Hellman: the long-term certificate key signs the handshake; it does not encrypt the session. A stolen certificate key later cannot decrypt recorded traffic.

**Interview line:** “TLS 1.3 is 1-RTT, forward secret, and the certificate proves the name. HTTP/2 did not remove TCP head-of-line blocking. HTTP/3 moved the stream onto QUIC to do that.”

## CDN in four sentences

The user should hit a nearby cache, not your origin. **DNS-based:** the resolver is given different A/AAAA answers by location (EDNS client subnet helps, because the resolver’s IP is a rough stand-in for the user). **Anycast:** the same IP is announced from many sites; BGP delivers the user to a nearby one. Edge miss pulls from origin and stores it. You cache things that are safe to cache (GET, correct Cache-Control), and you do not cache a personalized response just because it was a 200.

### Day 6 self-quiz
1. Flow vs congestion. What is the send window? Why must QUIC implement congestion control itself?
2. Draw slow start and congestion avoidance. Tahoe on any loss. Reno on 3 dup ACKs vs on timeout.
3. Recite the worked trace up to cwnd 18, then both loss cases at cwnd 20. What do CUBIC and BBR change? What is bufferbloat?
4. DNS: recursive vs iterative, the three servers in the walk, TTL and cutovers, UDP then TCP. Record types A, CNAME, MX, NS, PTR.
5. DORA, ports, why a relay exists.
6. 401 vs 403, PUT vs POST, 301 vs 302, 502 vs 504. Why HTTP/2 still has HOL and HTTP/3 does not. TLS 1.3 RTT and what forward secrecy means.

---

# DAY 7 — The URL Walk, TCP States, What They Ask Next

## What happens when you type a URL and press enter

Recite this **in order**. It is the question that ties the week together. Use `https://example.com/a`.

1. **Parse.** Scheme `https` → port **443** unless overridden. Host `example.com`. Path `/a`.
2. **HSTS / cache.** If the browser has previously seen a strict-transport header, it will not try cleartext. If a fresh cached response is allowed, it may not hit the network at all (stop here and say so).
3. **Resolve the name.** Browser cache, OS cache, `hosts`, then the recursive resolver. Resolver walks root → TLD → authoritative. You now have an IP. (Happy Eyeballs: a browser may race A and AAAA and use whoever answers.)
4. **Is the destination on-link?** Compare the IP to your address and mask. If not, the next hop is the **default gateway**. ARP (or NDP) resolves **the gateway’s MAC** if it is not cached. Your frame is addressed to the gateway. Your packet is addressed to example.com’s IP.
5. **Out of the house.** Your router NATs the source IP and port, recomputes checksums, and routes toward the destination. Each router: longest prefix, TTL−1, new frame, new link CRC. TTL 0 would have generated ICMP time exceeded; it doesn’t, because the path is fine.
6. **TCP handshake** to `(IP, 443)`. SYN, SYN-ACK, ACK. ISNs chosen. MSS and window scale negotiated. The server’s accept queue was filled from the listen backlog.
7. **TLS 1.3.** ClientHello with SNI = `example.com` and a key share. Server sends certificate and Finished. Browser checks the chain and the name. One RTT. (TLS 1.2 would have been two.)
8. **HTTP GET /a.** Host header, cookies if any, `Accept`. Server answers 200, or a redirect (then you repeat the needed steps for the new URL — a 301 to another host is a new DNS lookup and a new connection).
9. **The body references other objects.** HTTP/1.1: more requests on the persistent connection, plus parallel connections. HTTP/2: more streams on this one TCP+TLS session. HTTP/3: this whole transport was QUIC, and step 6–7 collapse into the QUIC handshake.
10. **Render.** Parse HTML, apply CSS, run JS. A JS call to another origin is a new lookup and is subject to **CORS** (the browser’s rule, not the network’s).

If any step fails, name the layer: DNS fail = you never open TCP. RST = nothing is listening or a middlebox rejected you. Timeout = filtered, blackholed, or the peer is silent. TLS error = clock, name mismatch, or an untrusted chain, **after** TCP succeeded. 502/504 = TCP and TLS worked and the **server’s upstream** did not.

## TCP state machine — both sides

```
Client (active open)                         Server
CLOSED                                       LISTEN
   | send SYN                                    | recv SYN, send SYN-ACK
   v                                             v
SYN_SENT  --recv SYN-ACK, send ACK-->     SYN_RCVD --recv ACK--> ESTABLISHED
                                              ^
ESTABLISHED <---------------------------------+

Active close                                 Passive close
ESTABLISHED                                  ESTABLISHED
   | send FIN                                    | recv FIN, send ACK
   v                                             v
FIN_WAIT_1  --recv ACK-->  FIN_WAIT_2       CLOSE_WAIT
   |                          | recv FIN,       | app calls close, send FIN
   |                          | send ACK        v
   |                          v              LAST_ACK --recv ACK--> CLOSED
   +-------> TIME_WAIT --2MSL--> CLOSED
```

Simultaneous close (both FIN before either sees the other’s) passes through **CLOSING**. Rare. Know the name.

| State you see in `ss` | It means |
|---|---|
| **SYN_SENT** | Client sent SYN, no SYN-ACK yet. Remote filtered, down, or routing broken |
| **SYN_RCVD** | Server sent SYN-ACK, waiting for the final ACK. A lot of these is a SYN flood or a full accept queue |
| **ESTABLISHED** | Data may flow |
| **FIN_WAIT_2** | We FINed and were ACKed; waiting for their FIN. Peer is slow to close, or it forgot |
| **TIME_WAIT** | We active-closed. Waiting 2MSL. Normal on clients. Painful if the **server** active-closes huge rates |
| **CLOSE_WAIT** | Peer FINed. **Our app has not closed.** Bug until proven otherwise |
| **LAST_ACK** | We are the passive side and our FIN is in flight |

## Socket calls — enough to connect the state machine to code

Server: `socket` → `bind` → `listen(backlog)` → `accept` (returns a **new** fd for the connection; the listen fd stays in LISTEN) → `read`/`write` → `close`.

Client: `socket` → `connect` (this is the SYN) → `write`/`read` → `close` (this is the FIN, and this side will TIME_WAIT).

`backlog` is the queue of connections waiting for `accept`, not the number of ESTABLISHED clients you are allowed. A slow `accept` loop drops handshakes even when the network is fine.

`SO_REUSEADDR` lets you bind a port that is still in TIME_WAIT. It is not a license to have two listeners.

## Which protocol — say it without a hedge

| Situation | Choice | Why |
|---|---|---|
| Web page, file, mail submission, SSH, database | **TCP** (or QUIC for web) | Complete, ordered bytes |
| DNS query, DHCP, NTP | **UDP** | One small exchange, app can retry |
| Voice, game state, live video | **UDP** | A retransmission that arrives late is waste |
| Video **file** / VOD | **TCP or HTTP** | Completeness matters, buffer hides jitter |
| Browser to a site, today | **HTTP/2 over TLS over TCP**, or **HTTP/3 over QUIC** | |
| Routing inside a company | **OSPF** | Fast, one admin |
| Routing between companies | **BGP** | Policy |
| “Reliable file transfer” as a design question | List TCP’s mechanisms, then say QUIC or TCP unless you have a reason to rebuild |

## Security pieces that sit next to the stack

**Stateful firewall vs ACL.** A stateless ACL must explicitly allow the **return** traffic (ephemeral source ports). A stateful firewall remembers the outbound SYN and allows the matching return packets. Same idea as a NAT table, different job: NAT rewrites addresses, the firewall decides pass or drop.

**IDS** watches and alerts (a copy of the traffic). **IPS** sits inline and can drop. A firewall is a policy gate. An IDS is a detector. Don’t swap the words.

**VPN.** An encrypted tunnel with a virtual interface. **IPsec** is network-layer (tunnels IP in IP, site-to-site or remote access). **TLS VPNs** and **WireGuard** do the same job at different layers and with different operational stories. To the laptop, routes point into the tunnel; the outer packet is ordinary UDP or IP to the VPN server.

**ARP spoofing, DNS spoofing, BGP hijack** are the three “who told you that, and why did you believe them?” attacks, one per layer: link (no auth on ARP), naming (poison a resolver or race a response), routing (announce a prefix you don’t own; RPKI exists so others can reject announcements you cannot sign).

**SDN, one paragraph.** Forwarding (the switch ASIC) is separated from the control program (a controller). The controller has a global view and installs rules (OpenFlow, or a vendor API). Same split as “forwarding vs routing,” institutionalized. Google’s WAN traffic engineering is the usual motivation story. You are not expected to configure a controller in a CS placement round.

## Short “what if” cards

| Event | What TCP does |
|---|---|
| Data ACK lost | Sender retransmits when RTO fires or dup ACKs pile up. Receiver ACKs the cumulative point again. Data was not lost |
| SYN lost | Client retransmits SYN. Server has no state |
| SYN-ACK lost | Client retransmits SYN. Server stays in SYN_RCVD and retransmits SYN-ACK |
| Final handshake ACK lost | Server retransmits SYN-ACK. Client, already ESTABLISHED, ACKs again. (This is why the server must not allocate forever — SYN cookies / backlog) |
| FIN lost | Retransmit FIN. This is why TIME_WAIT answers a retransmitted FIN |
| One segment lost, later ones arrive | Dup ACKs, fast retransmit (Reno/SACK). The byte stream stalls at the hole — **head-of-line** — even if later bytes are in the buffer |
| rwnd hits 0, window-update ACK lost | Persist timer, 1-byte probes |
| Sender ignores cwnd | Queues grow, delay grows, drop-tail finally discards, everyone else suffers |

### Day 7 self-quiz
1. Recite the URL walk from parse through render. Where do HSTS, ARP-for-gateway, NAT, and SNI each happen?
2. A TLS alert vs a TCP RST vs a DNS failure: which layer failed, and what already succeeded?
3. Draw active close and passive close. Define TIME_WAIT, CLOSE_WAIT, SYN_RCVD.
4. `listen` / `accept` / `connect`: which call sends the SYN? What is the backlog?
5. Pick the protocol: live scoreboard, OS image download, DHCP, inter-company routes, a video call.
6. Stateful vs stateless firewall. IDS vs IPS. Why a stolen TLS certificate key still shouldn’t decrypt last year’s traffic (assuming ECDHE).

---

# SUNDAY CLOSE — Full cheat sheet (recite out loud)

**Stack:** 5-layer PDU = message, segment, datagram, frame, bit. ARP = link. ICMP = IP proto 1. DHCP = app. OSPF = IP 89. BGP = TCP 179. RIP = UDP 520. TLS sits between HTTP and TCP.

**Delays:** proc, queue (the only traffic-dependent one), trans = L/R, prop = d/s. Path throughput = bottleneck. BDP = R × RTT = bits in flight. 1 Gbps × 30 ms = 3.75 MB.

**Switching:** circuit reserves, packet store-and-forwards, VC sets up a label. One packet, N hops: N·L/R. File pipelines: F/R + (N−1)·L/R.

**Devices:** hub = 1 collision domain. Switch = collision domain per port, one broadcast domain per VLAN, learns source MAC, floods unknown. Router = broadcast domain per port, no ARP forwarding, TTL−1. Default gateway = router on your subnet.

**Errors:** Ethernet CRC-32. IP/TCP/UDP = Internet checksum. IP checksum is header-only, redone every hop. Transport checksum includes the pseudo-header. NAT recomputes both.

**ARQ:** stop-and-wait η = 1/(1+2a), a = Tp/Tt, 1-bit sequence. GBN: Ws ≤ 2ᵐ−1, receiver window 1, cumulative ACK, go back. SR: both windows ≤ 2ᵐ⁻¹, buffer, retransmit the hole. m=3 → GBN 7, SR 4. Pipe full when Ws ≥ 1+2a.

**MAC:** pure ALOHA 1/(2e), slotted 1/e. CSMA/CD: jam + binary exponential backoff, k=min(n,10), give up at 16. Lmin because Tt ≥ 2Tp; 64-byte minimum. Full-duplex switched Ethernet has no collisions. Wi‑Fi is CSMA/CA + ACK; RTS/CTS fixes **hidden**, not exposed.

**Ethernet:** payload 46–1500, frame 64–1518, FCS 4. EtherType 0x0800 / 0x0806 / 0x86DD. STP exists because frames have no TTL.

**ARP:** same subnet → dest MAC. Else → **gateway MAC**, IP dst unchanged. Broadcast request, unicast reply. Spoofable.

**IPv4:** TTL, proto (1/6/17/89), identification + DF/MF + offset in 8-byte units, header checksum. Reassembly only at the destination. traceroute = rising TTL + ICMP time exceeded (Linux UDP, Windows ICMP). ping = ICMP echo 8/0.

**Addresses:** private 10/8, 172.16/12, 192.168/16. Loopback 127/8. Link-local 169.254/16. Multicast 224/4. Limited broadcast 255.255.255.255, not routed.

**Subnet:** h such that 2ʰ−2 ≥ hosts, prefix 32−h, block 2ʰ. /26 → block 64, 62 hosts. 192.168.1.0/24 → .0 .64 .128 .192. 172.16.5.20/20 → network 172.16.0.0, broadcast 172.16.15.255. /31 has 2 usable and no broadcast. Aggregate only on the boundary: .4–.7/24 → /22.

**Longest prefix wins.** Default /0 loses to anything more specific.

**NAT:** rewrite src IP and port, remember the tuple, replies match the public port. Inbound needs a mapping. Breaks end-to-end. Checksums redone.

**Fragment 4000 into MTU 1500, hdr 20:** payloads 1480, 1480, 1020. Offsets 0, 185, 370. MF 1, 1, 0. Total lengths 1500, 1500, 1040. Non-final payload rounded down to ×8. MSS = MTU−40 = 1460. DF + swallowed ICMP = PMTU black hole.

**IPv6:** 128-bit, 40-byte header, no checksum, no router fragmentation, no broadcast, NDP replaces ARP, SLAAC. ::1, fe80::/10, fc00::/7.

**DV:** d(y) = min (c(x,v) + dᵥ(y)). Count-to-infinity. Split horizon / poison reverse stop the 2-node case. RIP: hops, 16 = ∞, UDP 520.

**LS:** flood LSAs, Dijkstra locally. OSPF proto 89, DR on broadcast links, default Cisco cost uses 100 Mbps reference so 1 Gbps still costs 1, area 0 is the backbone.

**BGP:** path vector, TCP 179, policy first. LOCAL_PREF, then AS-PATH length, then MED, then eBGP over iBGP, then IGP cost to next hop (hot potato). iBGP does not re-advertise iBGP routes → full mesh or route reflector. AS-PATH loop detection. Not “shortest path.”

**UDP:** 8-byte header, message boundaries, no congestion control, can multicast. TCP: byte stream, 4-tuple demux, full duplex.

**Handshake:** SYN x, SYN-ACK y / ack x+1, ACK y+1. ISN unpredictable. SYN and FIN consume a sequence number. SYN cookie = state in the ISN. Window scale because rwnd field is 16-bit and BDP exceeds 64 KB.

**Close:** FIN/ACK, FIN/ACK. Active closer → TIME_WAIT for 2MSL, so the last ACK can be resent and old segments can die. CLOSE_WAIT = app didn’t close. RST aborts.

**RTO** = SRTT + 4·RTTVAR. Karn: don’t sample a retransmitted segment’s RTT. Back off the RTO exponentially on timeout.

**Flow:** window = min(cwnd, rwnd). rwnd 0 → persist probes. Clark: don’t advertise tiny windows. Nagle: coalesce small writes while data is unACKed. TCP_NODELAY turns Nagle off.

**Congestion:** SS doubles per RTT until ssthresh, then +1 MSS per RTT. Tahoe: any loss → ssthresh = cwnd/2, cwnd = 1. Reno 3 dup ACKs: fast retransmit, cwnd ≈ half, stay in CA. Reno timeout: back to 1. NewReno holds recovery across partial ACKs. SACK names the holes. CUBIC: ×0.7, cubic in time, Linux default. BBR: model BW and min RTT. Bufferbloat = deep queues, high delay, no loss.

**DNS:** stub → recursive resolver → iterative root, TLD, authoritative. TTL caches. UDP 53, TCP 53 if large. A, AAAA, CNAME, NS, MX, PTR, TXT, SOA.

**DHCP:** Discover, Offer, Request, Ack. UDP 67/68. Relay sets giaddr because broadcasts don’t cross routers.

**HTTP:** GET safe+idempotent, PUT idempotent replace-at-URL, POST neither. 401 authenticate, 403 refuse, 301 permanent, 302 temporary, 304 cache ok, 502 bad upstream, 504 upstream timeout. HTTP/1.1 persistent. HTTP/2 multiplexes on one TCP and still suffers TCP HOL. HTTP/3 = QUIC over UDP, independent streams, connection ID, TLS 1.3.

**TLS 1.3:** 1-RTT, ECDHE, certificate proves the name via SNI. Forward secrecy: certificate key signs, it doesn’t encrypt the session. 0-RTT can be replayed.

**URL:** parse → cache/HSTS → DNS → on-link check → ARP gateway → NAT → route (TTL, new frame) → TCP → TLS → HTTP → more objects → render.

**States:** SYN_SENT waiting for SYN-ACK. SYN_RCVD half-open. TIME_WAIT active closer. CLOSE_WAIT your app. `connect` sends SYN. `accept` dequeues a finished handshake. `listen` backlog is that queue.

---

# Interview lines (say these verbatim)

1. **Best effort:** “IP does not promise delivery, order, or delay. TCP is what turns that into a reliable byte stream, and only end to end.”
2. **ARP:** “ARP is link layer. If the destination is off-subnet I ARP for the gateway’s MAC and leave the IP destination alone.”
3. **Delays:** “Transmission is L/R, propagation is d/s, and only queuing depends on other people’s traffic.”
4. **BDP:** “The window has to cover rate times RTT or the sender spends the RTT idle. That’s the entire argument against stop-and-wait on a fat pipe.”
5. **Switch vs router:** “A switch splits collision domains and still floods broadcasts. A router is the edge of the broadcast domain and it decrements TTL.”
6. **STP:** “Ethernet has no TTL. A loop is a broadcast storm. Spanning tree blocks ports until the topology is a tree.”
7. **GBN vs SR:** “GBN’s receiver window is 1 and the sender rewinds the whole window. SR buffers and retransmits the hole, so both windows stay at most half the sequence space.”
8. **CSMA/CD vs CA:** “Wired half-duplex can hear a collision while sending, so it jams and backs off. A radio can’t, so Wi‑Fi avoids, then ACKs, and uses RTS/CTS for the hidden terminal. Switched full-duplex Ethernet doesn’t collide at all.”
9. **Longest prefix:** “Every matching route is a candidate. The longest mask wins. The default route matches everything and loses to all of them.”
10. **NAT:** “The router rewrites source IP and port and remembers the tuple. Replies find their way back. Unsolicited inbound packets don’t, because there’s no row yet.”
11. **Fragments:** “Offset is in 8-byte chunks, MF marks every piece but the last, identification ties them together, and only the destination reassembles. Lose one piece and the datagram is gone.”
12. **Count-to-infinity:** “After B–C fails, B trusts A’s stale ‘I can reach C,’ and A trusts B, and the metric climbs to infinity. Split horizon stops the two-node version.”
13. **BGP:** “Highest local preference first, because this is policy between companies, not shortest path. AS-PATH length is later. iBGP won’t readvertise an iBGP route, so you full-mesh or you use a route reflector.”
14. **Three-way:** “Both sides have to hear that their ISN arrived. The third message is the client proving it saw the server’s ISN. The ISN is random so old segments and off-path attackers don’t get to guess.”
15. **TIME_WAIT:** “The side that sent the first FIN waits two MSLs so it can re-ACK a retransmitted FIN, and so old duplicates of this 4-tuple are gone before reuse.”
16. **CLOSE_WAIT:** “The peer has finished. Our application hasn’t called close. That’s our bug.”
17. **Tahoe / Reno:** “Tahoe goes to cwnd 1 on any loss. Reno treats three duplicate ACKs as a hole in a living pipe: retransmit, cut in half, stay in congestion avoidance. A timeout still goes back to 1.”
18. **Flow vs congestion:** “rwnd is the peer telling you its buffer. cwnd is you guessing the network. You send the minimum of the two.”
19. **DNS:** “My resolver recurses for me and iterates from the root to the TLD to the authority. The TTL is why the new address doesn’t show up at once.”
20. **HTTP/3:** “HTTP/2 multiplexed streams on one TCP, so one loss stalled all of them. HTTP/3 puts those streams on QUIC over UDP, with congestion control included, because UDP will not provide it.”
21. **TLS:** “TLS 1.3 finishes in one round trip. The certificate proves the name. Ephemeral Diffie-Hellman means today’s stolen certificate key doesn’t decrypt yesterday’s traffic.”
22. **URL:** “DNS, then ARP for the gateway, then NAT, then IP hop by hop, then TCP, then TLS with SNI, then HTTP. A certificate error means TCP already worked.”

---

# Weekly 15-minute emergency pass (if you only have one sitting)

Recite in this order, eyes closed:

1. 5-layer PDUs → where ARP, ICMP, OSPF, BGP, TLS sit → encapsulation and what a router rewrites
2. Four delays → BDP 1 Gbps × 30 ms = 3.75 MB → hub / switch / router domains
3. Checksum vs CRC → stop-and-wait η → GBN 2ᵐ−1 vs SR 2ᵐ⁻¹ → ALOHA 18% / 37%
4. CSMA/CD backoff and min frame → why full duplex has no CD → hidden terminal and RTS/CTS → Ethernet sizes → STP because no TTL
5. ARP: on-link vs gateway → IPv4 header fields and protocol numbers → private ranges
6. /26 split of a /24 → 172.16.5.20/20 → longest prefix → NAT tuple → fragment offsets 0, 185, 370
7. Count-to-infinity → Dijkstra row updates → RIP 16, OSPF 89, BGP 179 and LOCAL_PREF before AS-PATH → iBGP mesh
8. UDP vs TCP byte stream → handshake with ISNs → SYN cookies → four-way close → TIME_WAIT vs CLOSE_WAIT
9. RTO = SRTT + 4·RTTVAR, Karn → rwnd 0 and persist → Nagle / Clark
10. Slow start doubles, CA adds one → Tahoe to 1, Reno 3-dup to half → CUBIC 0.7 and BBR one-liners
11. DNS root / TLD / auth and TTL → DORA → 401/403, PUT/POST, 502/504 → HTTP/2 HOL vs HTTP/3 → TLS 1.3 1-RTT and PFS
12. URL walk in order → TCP states that show up in `ss` → min(cwnd, rwnd)
