---
title: High-Speed Networks - Frame Relay, ATM, QoS, and MPLS
description: The WAN and LAN technologies built for speed and the traffic machinery around them - Frame Relay and ATM's virtual circuits and cell relay, Fast and Gigabit Ethernet, queuing and congestion effects, traffic shaping and policing, the IntServ and DiffServ QoS models with WFQ and RED, RSVP signaling, and MPLS label switching.
date: 2022-10-05
draft: false
slug: /computer-networks/high-speed-networks
tags:
  - Networking
  - QoS
  - MPLS
---

Best-effort IP treats every packet the same and makes no promises about rate or delay. That's fine for a file download and hopeless for a phone call. This post is the collection of technologies and mechanisms built to do better: the WAN designs (Frame Relay, ATM) that pioneered virtual circuits and traffic contracts, the faster Ethernet variants, the queuing and congestion behaviour those speeds expose, and the quality-of-service architectures — IntServ, DiffServ, MPLS — that try to give some traffic guarantees the internet's base layer won't.

## Legacy high-speed WANs

### Frame Relay

A 1990s WAN service, leaner than the X.25 it replaced. It runs on **virtual circuits (VCs)** — logical connections between endpoints — either **permanent** (PVC, pre-provisioned) or **switched** (SVC, set up on demand). Data travels as variable-length frames, with error *checking* (not correction) pushed to the endpoints so the network nodes do less and run faster. Congestion is signalled by header bits: **FECN** (forward explicit congestion notification, set toward the destination), **BECN** (backward, toward the source, telling it to slow down), and **DE** (discard eligibility — mark lower-priority frames to be dropped first).

### ATM

A more ambitious design meant to be *the* unified network for voice, video, and data with real QoS. Its own layered model: a physical layer, an **ATM layer** (cell relay, routing by **Virtual Path Identifier / Virtual Channel Identifier**, multiplexing), and an **ATM Adaptation Layer (AAL)** that segments higher-layer data into cells and reassembles it. A Virtual Path bundles many Virtual Channels.

The signature choice is the **cell**: a fixed 53 bytes — 5 header, 48 payload — small and constant so switching can be pure hardware and queuing delay for real-time traffic stays low. ATM defined service categories for different needs: **CBR** (constant bit rate, guaranteed), **rt-VBR** and **nrt-VBR** (variable, with or without tight timing), **ABR** (adapts its rate to network feedback), **UBR** (best-effort). AAL types matched traffic to cells — AAL1 for CBR, AAL2 for compressed voice/video, AAL3/4 for data (complex), **AAL5** simple and widely used for IP over ATM.

ATM didn't become the universal network — Ethernet's speed growth and IP's flexibility won — but its traffic-contract and QoS ideas carried into everything after it.

## High-speed LANs

- **Fast Ethernet** — 10 → 100 Mbps, keeping the CSMA/CD mechanism and frame format for compatibility. Added **auto-negotiation** of speed (10/100) and duplex (half/full). Mostly 100BASE-TX over twisted pair.
- **Gigabit Ethernet** — 1 Gbps, run almost exclusively full-duplex through switches, so [CSMA/CD](/citadel/computer-networks/datalink-layer) is irrelevant. Copper (1000BASE-T over Cat 5e/6) and fibre; **jumbo frames** beyond the 1500-byte payload for bulk-transfer efficiency. 10, 40, and 100 GbE followed.
- **Fibre Channel** — 1 Gbps to 128 Gbps+, built for **storage area networks**: in-order, lossless block transfer between servers and storage. Topologies of point-to-point, arbitrated loop, and (usual) switched fabric; a protocol stack FC-0 to FC-4 carrying SCSI, IP, or mainframe FICON.

## Queuing and congestion

Queues form at routers whenever packets arrive faster than the output link drains them. **Queuing theory** models this — an M/M/1 queue (Poisson arrivals, exponential service times, one server) predicts average queue length, waiting time, and loss probability, which is what buffer sizing is based on.

When traffic exceeds capacity, **congestion** produces increased **delay** (longer queues), increased **jitter** (variable delay, bad for audio/video), **packet loss** (buffers overflow), and — as loss triggers retransmissions — falling **goodput**.

**Congestion control** comes in two shapes:

- **Open-loop** (prevention) — admission control and traffic shaping to stop congestion arising.
- **Closed-loop** (reaction) — feedback adjusts sending rates. *Explicit* feedback: routers signal congestion directly (ECN bits in IP, RM cells in ATM). *Implicit* feedback: the sender infers congestion from loss or rising RTT — [TCP](/citadel/computer-networks/transport-layer)'s approach.

**Traffic management** at the network edge:

- **Traffic shaping** (leaky bucket, token bucket) — buffer and meter packets out at a controlled rate, smoothing bursts.
- **Traffic policing** — check traffic against its negotiated contract; drop, re-mark, or shape whatever doesn't conform.

## TCP in fast, long networks

TCP's timers were tuned for slower, shorter paths. Two adjustments matter on long fat networks:

- **Exponential RTO backoff** — double the retransmission timeout on each successive timeout for a segment.
- **Karn's algorithm** — don't update RTT estimates from ACKs of *retransmitted* segments (you can't tell which copy the ACK is for); use the backed-off RTO until a segment is acknowledged without retransmission, then resume normal RTT estimation.

Running TCP/IP over ATM was awkward: losing one cell corrupts the whole segmented IP packet, forcing a full retransmission; AAL5 segmentation adds overhead; and TCP's congestion control didn't mesh with ATM's own traffic management, especially ABR's explicit-rate feedback. ATM's ABR used **RM (resource management) cells** interleaved with data — switches mark fields (Explicit Rate, Congestion Indication, No Increase), and the source adjusts its Allowed Cell Rate when the cell returns — alongside a framework of connection admission control, usage-parameter policing, and traffic descriptors (Peak / Sustainable Cell Rate, Maximum Burst Size) with QoS parameters (Cell Loss Ratio, Cell Transfer Delay, Cell Delay Variation).

## QoS architectures

### IntServ

Fine-grained, per-**flow** guarantees. An application asks for a specific service level and routers **reserve resources** for that flow. Components: **admission control** (can this request be met without breaking existing guarantees?), **RSVP** for signalling, a **packet scheduler** (typically WFQ), and ordinary routing. Two services: **guaranteed** (a firm delay bound and assured bandwidth) and **controlled-load** (performance like an unloaded network). The fatal weakness is scalability — every router holds per-flow state and does per-flow work.

### Queuing disciplines

How a router services its queues largely determines the QoS it can offer:

- **Fair queuing (FQ)** — a separate queue per flow, serviced round-robin, approximating a bit-by-bit round robin.
- **Priority scheduling** — packets in priority classes, high always before low; low-priority traffic can **starve**.
- **Generalized Processor Sharing (GPS)** — the theoretical ideal: flow $i$ with weight $w_i$ is guaranteed rate $\dfrac{w_i}{\sum_j w_j} \times R$ of link capacity $R$.
- **Weighted Fair Queuing (WFQ)** — a practical GPS approximation: each flow (or class) gets its weighted share, with good delay for low-volume flows.

### RED

**Random Early Detection** is active queue management for congestion *avoidance*. Instead of waiting for a full queue and then tail-dropping, RED watches the *average* queue size and starts dropping arriving packets with a probability that rises as the queue grows. Early drops nudge TCP senders to back off before things get bad and prevent whole fleets of connections synchronising their backoffs. **WRED** varies the drop probability by IP precedence or DSCP.

### DiffServ

Coarse-grained and scalable — the answer to IntServ's per-flow cost. Packets are classified into a handful of classes **at the network edge** and marked with a **Differentiated Services Code Point (DSCP)** in the IP header. Core routers apply a **Per-Hop Behavior (PHB)** by class and keep *no* per-flow state. PHBs include **Expedited Forwarding** (low loss, latency, and jitter with assured bandwidth — effectively a virtual leased line) and **Assured Forwarding** (several classes, each with drop precedences). All the complexity — classification, marking, shaping, policing — lives at the edge.

### RSVP

The signalling protocol for IntServ reservations, for unicast and multicast. It's **receiver-initiated**, uses **soft state** (reservations in routers expire unless periodically refreshed, which makes it robust to change), and is **simplex** (one direction per reservation). **Path** messages travel downstream from the sender, leaving path state in routers; **Resv** messages travel back upstream along that state, making the actual reservations, which routers may accept, reject, or modify. Message types: PATH, RESV, PATHERR, RESVERR, RESVTEAR, with filter specs identifying the packets and flow specs defining the QoS.

## MPLS

**Multiprotocol Label Switching** forwards on short fixed **labels** instead of long address lookups. A packet gets a label at the ingress **Label Switch Router (LSR)**; each LSR along the way looks up the incoming label, **swaps** it for an outgoing one, and forwards. The path a labelled packet follows is a **Label Switched Path (LSP)**, set up by a signalling protocol (LDP, RSVP-TE) or by configuration. Packets forwarded identically form a **Forwarding Equivalence Class (FEC)**, assigned at ingress. Labels can be **stacked** for hierarchical LSPs and MPLS VPNs (one label for the VPN, one for the route within it).

MPLS buys **traffic engineering** (route flows explicitly to balance load), simpler **VPNs**, DSCP-style **QoS** by mapping classes to LSPs, and **protocol independence** — it can carry IP, Ethernet, or ATM.

## The one idea to keep

Guaranteeing anything on a packet network means giving up some statistical sharing — reserving capacity (IntServ, ATM CBR), or at least sorting traffic into priority classes (DiffServ, MPLS). The scalable designs push all the per-flow decisions to the edge and keep the core simple and stateless, which is the same architectural move that let IP scale in the first place.
