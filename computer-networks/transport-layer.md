---
title: The Transport Layer - Ports, UDP, and How TCP Stays Reliable
description: Turning host-to-host delivery into process-to-process delivery - multiplexing with port numbers, UDP's bare-bones service, the reliable-transfer progression from stop-and-wait through Go-Back-N to Selective Repeat, and how TCP combines them with flow control and congestion control.
date: 2022-08-28
draft: false
slug: /computer-networks/transport-layer
tags:
  - Networking
  - TCP/IP
  - Transport
---

The [network layer](/citadel/computer-networks/newtwork-layer) gets a packet from one host to another, best-effort — it can lose it, corrupt it, reorder it. The transport layer turns that into something an application can use: delivery to the right *process*, and — if the application wants it — a guarantee that every byte arrives once, in order.

Two protocols do this. **UDP** adds almost nothing to IP and gets out of the way. **TCP** builds a reliable, in-order byte stream on top of unreliable packets, and throttles itself so the network doesn't collapse. This post is how each works, and the reliable-transfer ideas TCP is assembled from.

## Multiplexing and demultiplexing

Your machine runs many network programs at once. The transport layer's first job is sorting.

- **Multiplexing** (sender) — collect data from each process, wrap it in a transport header carrying **source and destination port numbers**, hand the **segment** to IP.
- **Demultiplexing** (receiver) — read the incoming segment's header and deliver its payload to the right **socket** (a process's doorway to the network).

The identifying tuple differs by protocol:

- **UDP** — a socket is `(destination IP, destination port)`. Every segment with that pair goes to the same socket, whoever sent it.
- **TCP** — a socket is the full 4-tuple `(source IP, source port, destination IP, destination port)`. Two clients hitting the same server port get *different* sockets, so the server can hold many connections at once.

## UDP

UDP is IP with ports bolted on. It takes an application message, prepends source port, destination port, length, and a checksum, and sends it. Reasons to choose it:

- **No handshake** — no round trip before data, which suits short request/response exchanges like DNS.
- **No connection state** — the server keeps nothing per client, so it can serve more of them.
- **8-byte header** — less overhead than TCP's 20+.
- **No congestion control** — it sends as fast as the app wants, good for constant-rate media, and a hazard for the network.
- **No retransmission** — a lost packet stays lost, which is right for VoIP and gaming where a late packet is worse than none.

An app that needs reliability over UDP must build it itself.

The **checksum** covers the UDP header, the data, and a *pseudo-header* (source/destination IP and protocol number lifted from the IP header). It's optional in IPv4, mandatory in IPv6. Why bother, when links often check errors too? Because not every link does, and a segment can be corrupted sitting in a router's memory after a clean transmission.

## Reliable data transfer, built up

IP is best-effort, so reliability is the transport layer's problem. It needs **flow control** (don't send faster than the receiver can absorb) and **error control** (detect corruption and loss, recover by retransmission — *automatic repeat request*, ARQ). The classic progression:

### Stop-and-wait

Send one segment, wait for its **ACK**, then send the next. Corruption is caught by a checksum (discard, or send a NAK). Loss is caught by a **timer** — no ACK before it fires, retransmit. Delayed (not lost) ACKs cause duplicate deliveries, so segments carry a **sequence number**; here a single bit (0/1) is enough to tell a retransmission from a new segment.

Efficiency is poor on fast or long links: link utilisation is

$$U = \frac{1}{1 + 2a}, \qquad a = \frac{\text{propagation time}}{\text{transmission time}}$$

A satellite link has large $a$, so the sender spends almost all its time waiting.

### Pipelining: Go-Back-N and Selective Repeat

Let the sender have up to $N$ unacknowledged segments in flight (a **window**).

**Go-Back-N.** The receiver accepts segments *only in order* and discards anything out of order; its window is 1. It sends **cumulative ACKs** — "I have everything through $k$". The sender keeps one timer, for the oldest unacknowledged segment; on timeout it retransmits *the entire window* from there. With $m$-bit sequence numbers the send window must be $< 2^m$ (commonly $2^m - 1$). The weakness: one lost segment forces retransmission of everything after it, including segments the receiver actually got.

**Selective Repeat.** The receiver has a window of $N$ too and **buffers** out-of-order segments, **ACKing each one individually**. The sender runs a timer per segment and retransmits only the ones that time out. The receiver delivers a contiguous run to the application once it's complete. To keep a retransmitted old segment from being mistaken for a new one, both windows must be $\le 2^{m-1}$.

## TCP

TCP gives applications a **reliable, in-order stream of bytes**, connection-oriented, full-duplex, using a hybrid of Go-Back-N and Selective Repeat ideas.

### The segment

Key header fields:

- **Source / destination port** (16 bits each).
- **Sequence number** (32 bits) — the byte-stream offset of this segment's first data byte. On a SYN it's the **initial sequence number (ISN)**.
- **Acknowledgment number** (32 bits) — the next byte the sender of the ACK expects. Cumulative.
- **Header length** (4 bits), **flags** (6 bits): `URG`, `ACK`, `PSH`, `RST`, `SYN`, `FIN`.
- **Receive window** (16 bits) — for flow control; bytes the sender is willing to accept.
- **Checksum** (16 bits, mandatory), **urgent pointer**, and **options** (MSS negotiation, window scaling, selective ACK).

The ISN isn't 0 — a random start keeps stray segments from a previous, dead connection with the same socket pair from being accepted.

### Connection management

**Three-way handshake** to open:

1. client → server: `SYN=1`, `Seq=client_isn`
2. server → client: `SYN=1`, `ACK=1`, `Seq=server_isn`, `Ack=client_isn+1`
3. client → server: `ACK=1`, `Ack=server_isn+1`

**Four-way handshake** to close, with a **half-close** (each direction shuts independently): `FIN` / `ACK` / `FIN` / `ACK`. The side that closed first sits in `TIME_WAIT` for 30 s to 2 minutes so lingering segments die before the socket pair is reused.

### Flow control

The receiver advertises its free buffer space as the **receive window** in every segment; the sender keeps its unacknowledged data within that window, so it can't overrun the receiver's buffer.

### Error control

- **Checksum** for corruption.
- **Cumulative ACKs** for confirmation.
- **Retransmission timeout (RTO)** — derived from measured round-trip times, **doubled** on each timeout (exponential backoff).
- **Fast retransmit** — three duplicate ACKs for the same byte strongly imply the next segment was lost, so retransmit it immediately rather than wait for the RTO.
- **Selective acknowledgment (SACK)** — an option letting the receiver name out-of-order blocks it holds, so the sender retransmits precisely.

## Congestion control

Flow control protects the *receiver*; congestion control protects the *network*. When routers are overwhelmed they drop packets, delays balloon, and in the worst case retransmissions pile on to cause **congestion collapse** — almost no useful throughput.

TCP maintains a **congestion window `cwnd`**, and the amount it may have in flight is

$$\text{effective window} = \min(\text{cwnd},\ \text{rwnd})$$

The control loop: network looks clear (ACKs arriving) → grow `cwnd`; loss detected (timeout or duplicate ACKs) → shrink it.

### Slow start and congestion avoidance

- **Slow start** — begin with `cwnd = 1 MSS`; add 1 MSS per ACK, which *doubles* `cwnd` every RTT. Continue until `cwnd` reaches a threshold `ssthresh`, or loss occurs. (The name is historical — the growth is exponential.)
- **Congestion avoidance** — past `ssthresh`, grow gently: about +1 MSS per RTT (`cwnd += MSS × MSS/cwnd` per ACK). Additive probing for spare capacity.

### Reacting to loss

- **On a timeout** (severe): `ssthresh = max(cwnd/2, 2 MSS)`, `cwnd = 1 MSS`, re-enter slow start. Every TCP does this.
- **On three duplicate ACKs** (a single lost segment, network still flowing) — this is where "flavours" differ:
  - **Tahoe** treats it like a timeout: `ssthresh = cwnd/2`, `cwnd = 1 MSS`, slow start.
  - **Reno** does **fast recovery**: `ssthresh = cwnd/2`, `cwnd = ssthresh + 3 MSS` (crediting the 3 segments that have left the network), retransmit the lost segment, and inflate `cwnd` by 1 MSS per further duplicate ACK. When an ACK for new data arrives, deflate `cwnd` to `ssthresh` and go to congestion avoidance — never back to slow start for a single loss.

A Tahoe run: `ssthresh` 16, `cwnd` climbs 1→2→4→8 (slow start), timeout at 8 → `ssthresh` 4, `cwnd` 1; climbs 1→2→4 (slow start), then 4→5→6→…→12 (congestion avoidance), three duplicate ACKs at 12 → `ssthresh` 6, `cwnd` 1. Reno diverges at that last step: `ssthresh` 6, `cwnd` = 6 + 3 = 9, fast recovery, then congestion avoidance from 6 — no return to `cwnd = 1`. Later flavours (NewReno, CUBIC) refine the recovery and the growth curve for [high-speed, high-latency links](/citadel/computer-networks/high-speed-networks).

## The one idea to keep

TCP is UDP plus three feedback loops: sequence-numbers-and-ACKs for reliability, the advertised receive window for flow control, and the congestion window for congestion control. The first makes an unreliable channel reliable; the second keeps a fast sender from drowning a slow receiver; the third keeps every sender together from drowning the network. Take away the loops and you have UDP — which is exactly why UDP is the right choice when the application can run those loops better itself, or doesn't need them.
