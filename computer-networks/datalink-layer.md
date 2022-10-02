---
title: The Link Layer - Framing, Error Detection, and Media Access
description: Moving a frame across one hop - ARP for IP-to-MAC resolution, the error-detection ladder from parity to CRC to Hamming codes, the multiple-access protocols (ALOHA, CSMA/CD, CSMA/CA) that arbitrate a shared channel, and how Ethernet switches and 802.11 actually work.
date: 2022-10-02
draft: false
slug: /computer-networks/datalink-layer
tags:
  - Networking
  - Ethernet
  - Wireless
---

The [network layer](/citadel/computer-networks/newtwork-layer) plans the whole route; the link layer executes one hop of it. Its job is to carry a datagram — wrapped now in a **frame** — from a node to the physically adjacent node across one link, whether that link is an Ethernet cable, a fibre run, or shared air.

That turns out to need real machinery: a way to map the next-hop IP address to a physical address, error detection strong enough for a noisy medium, and — when many nodes share one channel — a protocol to decide who transmits when. This post covers ARP, the error-detection ladder, the multiple-access protocols, and how Ethernet switching and Wi-Fi's MAC actually operate.

## Two sublayers

Hosts and routers are **nodes**; the channels between adjacent nodes are **links**. The link layer splits in two:

- **Logical Link Control (LLC)** — the interface up to the network layer: error detection, and optionally error recovery and flow control.
- **Media Access Control (MAC)** — framing (wrapping a datagram with header and trailer), controlling access to a shared medium, and switching between LANs.

Ethernet, Wi-Fi (802.11), and PPP are link-layer protocols.

## ARP: from IP address to MAC address

The network layer forwards on IP addresses, but sending a frame over a specific link needs the neighbour's 48-bit **MAC address**. **ARP** resolves one to the other for hosts on the same physical network.

1. **Request (broadcast)** — Host A doesn't know Host B's MAC, so it broadcasts "who has `IP_B`? tell `MAC_A`". Every node on the segment hears it.
2. **Reply (unicast)** — Host B recognises its own IP and replies directly: "`IP_B` is at `MAC_B`".
3. **Cache** — hosts keep an **ARP cache** of recent mappings (entries time out after ~15 minutes) and check it before sending a request. A also caches B's mapping from the reply, and B caches A's from the request.

A **gratuitous ARP** is an unsolicited announcement of a host's own mapping — source and target IP both set to its own address, target MAC the broadcast address. It's sent when an address changes or an interface comes up, to refresh other hosts' caches, and it doubles as duplicate-IP detection (someone else replies claiming that IP).

## Error detection and correction

Noise flips bits. The link layer adds redundant **error-detection-and-correction (EDC) bits** and checks them on receipt.

**Parity.** A single parity bit makes the count of 1s in a block even (or odd). It catches any *odd* number of flipped bits and misses even ones. **Two-dimensional parity** lays the data in a grid with a parity bit per row and per column; the row and column that both fail pinpoint a single-bit error, so it *corrects* single-bit errors and detects many double-bit ones.

**Checksum.** Treat the data as 16-bit integers, sum them in one's-complement arithmetic (carries wrap around and add back), and send the one's complement of the total. The receiver re-sums including the checksum and expects all 1s. It's weak against some error patterns but trivial in software, which is why TCP, UDP, and IP use it — on the assumption the link layer's stronger check catches the rest.

**Cyclic redundancy check (CRC).** The powerful one, used by Ethernet, 802.11, and HDLC. Treat the $n$-bit message as a polynomial $M(x)$; sender and receiver agree on a **generator polynomial** $C(x)$ of degree $k$. Append $k$ zero bits ($M(x) \cdot x^k$), divide by $C(x)$ using mod-2 polynomial division, and the $k$-bit **remainder** $R(x)$ is the CRC. Transmit message + CRC. The receiver divides the received frame by $C(x)$: zero remainder means no detected error. A well-chosen degree-$k$ generator (Ethernet uses 32 bits) catches all single-bit errors, all burst errors of length $\le k$, a burst of length $k+1$ with probability $1 - 0.5^{k-1}$, and longer bursts with probability $1 - 0.5^k$.

**Once an error is detected**, the options are: **drop** the frame (let TCP recover, or accept the loss for streaming media); **forward error correction (FEC)** — carry enough redundancy to fix it in place, worthwhile on noisy wireless or high-latency links where retransmission is expensive (see [erasure coding](/citadel/interview/erasure-coding)); or **retransmission** — signal the sender and resend.

**Hamming codes** make single-bit correction concrete. The **Hamming distance** between two codewords is the number of bit positions where they differ. To *detect* up to $d$ errors a code needs minimum distance $d + 1$; to *correct* up to $d$ it needs $2d + 1$. For single-bit correction of $d$ data bits, the number of parity bits $r$ must satisfy $2^r \ge d + r + 1$ (for $d = 7$, that's $r = 4$). Parity bits sit at power-of-two positions (1, 2, 4, 8, …), each checking a fixed subset of bit positions. On receipt, the pattern of failed checks — the **syndrome** — is 0 for no error, or its numeric value gives the position of the flipped bit.

## Sharing the channel: multiple-access protocols

A **point-to-point** link has one sender and one receiver. A **broadcast** (shared) link has many nodes on one medium, and two simultaneous transmissions **collide**. MAC protocols coordinate access.

**Channel partitioning** hands each node a dedicated slice — time slots (**TDMA**), frequency bands (**FDMA**), or orthogonal codes (**CDMA**). Efficient and fair under heavy load; wasteful under light load when slices go unused. GSM uses TDMA over FDM; 3G uses CDMA.

**Random access** lets nodes transmit at the full rate with no coordination, and recover from collisions by retrying after a random delay. Efficient when the channel is lightly loaded, collision-bound when it's busy — this is Ethernet and Wi-Fi.

- **Pure ALOHA** — transmit as soon as a frame is ready. A frame of transmission time $T_{fr}$ collides if another node starts anywhere in a window of $2T_{fr}$ around it. Peak throughput $S = G e^{-2G} \approx 18\%$.
- **Slotted ALOHA** — transmit only at slot boundaries. The vulnerable window shrinks to $T_{fr}$, and peak throughput rises to $S = G e^{-G} \approx 37\%$.
- **CSMA** ("listen before talk") — sense the channel; transmit if idle, defer if busy. Collisions still happen because of propagation delay (two nodes both sense idle and start). Variants: *1-persistent* (transmit the instant the channel goes idle — Ethernet), *non-persistent* (wait a random time before re-sensing), *p-persistent* (transmit with probability $p$ when idle — 802.11).
- **CSMA/CD** ("listen while talking") — a transmitting node that detects a collision stops immediately, sends a brief jam signal, and backs off a random time. Traditional wired Ethernet.

**Controlled access** trades some overhead for no collisions. **Polling** — a master invites each node to transmit in turn (used in Bluetooth); simple, but adds polling delay and a single point of failure. **Token passing** — a special token frame circulates and only its holder may transmit (Token Ring, FDDI); efficient and decentralised, but token loss or a failed node can stall the ring.

## Switched LANs

Early Ethernet was a shared coax bus; a cable break took down everyone. Modern Ethernet is a **star** around a central device.

**Hubs** are physical-layer repeaters: bits in on one port go out on all others. Every attached node is in one **collision domain**, there's no buffering, and all ports run one speed.

**Layer-2 switches** (bridges) are smarter. They **store and forward** frames, and each port is its own collision domain — so nodes on different ports transmit simultaneously without colliding. They buffer, and can mix port speeds. The **self-learning** algorithm:

- On a frame's arrival, record its *source* MAC against the incoming port (with a refresh timer).
- Look up the *destination* MAC: unknown → **flood** to every port except the arrival port; known and on the same port → **filter** (drop); known and on a different port → **forward** to just that port.

Switching is connectionless and unreliable at layer 2; reliability is TCP's job.

## Ethernet

The dominant wired LAN — 10BaseT, 100BaseT (Fast), 1000BaseT (Gigabit), all star topology with switches. The frame:

| Field | Size | Purpose |
| --- | --- | --- |
| Preamble | 7 B | clock synchronisation |
| Start Frame Delimiter | 1 B | `10101011`, marks the frame start |
| Destination MAC | 6 B | recipient's physical address |
| Source MAC | 6 B | sender's |
| Type / Length | 2 B | upper-layer protocol (IP, ARP) |
| Data | 46–1500 B | the datagram; 46 min so old bus networks could detect collisions |
| CRC | 4 B | error detection |
| Inter-Frame Gap | 12 B equiv. | mandatory quiet time between frames |

A **MAC address** is 48 bits, burned into the NIC, written as 12 hex digits with a manufacturer prefix. An adapter keeps a frame if it's addressed to its own unicast address or to the broadcast address `FF:FF:FF:FF:FF:FF`.

## VLANs

A **VLAN** is a LAN defined in switch software rather than by wiring. One physical switching infrastructure is partitioned into several virtual LANs, isolated from each other at layer 2 as if physically separate; traffic between VLANs must be routed by a layer-3 device. **IEEE 802.1Q** tagging carries the VLAN ID on frames travelling between switches. ([MPLS](/citadel/computer-networks/high-speed-networks), a related label-based forwarding scheme, gets its own treatment.)

## 802.11 (Wi-Fi) MAC

Wireless links differ from wired: signal strength drops with distance, other devices share the band, and reflections arrive as **multipath** interference. Crucially, **a node can't hear others while it transmits** — its own signal drowns them — so collision *detection* is impractical. 802.11 uses **CSMA/CA** (collision *avoidance*): sense before transmitting, and don't attempt to detect collisions mid-frame.

Two wireless-specific problems:

- **Hidden terminal.** A transmits to B; C can't hear A, senses the channel idle, transmits, and collides at B. Fix with **RTS/CTS**: A sends a Request To Send to B, B replies Clear To Send which *C* hears, so C defers. Then A sends data and B ACKs.
- **Exposed terminal.** S1 transmits to R1; S2 hears S1 and needlessly defers its transmission to R2, even though R2 is out of S1's range and would receive fine.

The MAC flow: if the channel is idle for a **DIFS** interval, transmit. If busy, pick a random **backoff** from a contention window, decrement it only while the channel is idle, and transmit at zero. The receiver waits a shorter **SIFS** and sends an ACK. No ACK → assume loss, double the contention window, back off again. When RTS/CTS is used, their duration fields let other stations set a **Network Allocation Vector** — a virtual carrier sense — and defer until it expires.

## The one idea to keep

The link layer is where "just send the bits" stops being simple. A shared medium forces an arbitration protocol, and the right one depends on load and on whether collisions can even be detected — CSMA/CD for wired Ethernet, CSMA/CA with RTS/CTS for wireless. Error detection is a similar spectrum: cheap parity where the medium is clean, strong CRC at every Ethernet frame, full correction only where retransmission costs too much.
