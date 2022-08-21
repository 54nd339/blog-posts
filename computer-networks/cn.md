---
title: Computer Networks - The Edge, the Core, and the Layered Model
description: The frame for everything else - what a network is, the topologies that connect nodes, the client-server and peer-to-peer edge, circuit versus packet switching in the core, the four things "performance" actually means, and why the whole stack is built in layers.
date: 2022-08-21
draft: false
slug: /computer-networks/cn
tags:
  - Networking
  - TCP/IP
  - Protocols
---

Loading this page pulled bytes through a dozen or more machines you'll never see — your Wi-Fi access point, a home router, several ISP routers, a load balancer, a server — each running software that agreed in advance on exactly how to format and hand off the data. A computer network is that agreement made concrete: a system for moving information between machines as electrical, optical, or radio signals.

This post is the frame the rest of the series hangs on. How nodes are wired together (topologies), where the applications live (the edge), how data crosses the middle (the core, switched one of two ways), what "fast" and "good" actually decompose into, and why networking is built as a stack of layers rather than one big program. The individual layers get their own posts: [application](/citadel/computer-networks/application-layer), [transport](/citadel/computer-networks/transport-layer), [network](/citadel/computer-networks/newtwork-layer), [link](/citadel/computer-networks/datalink-layer), plus [wireless](/citadel/computer-networks/wireless-networks), [high-speed networks and QoS](/citadel/computer-networks/high-speed-networks), and [security across the stack](/citadel/computer-networks/network-security).

## The city analogy

A network moves a commodity between entities the way a road system moves vehicles. The parallel is close enough to be useful:

| Road system | Computer network |
| --- | --- |
| vehicle | packet |
| street address | IP address |
| intersection | router |
| traffic jam | congestion |
| traffic light | flow control |
| alternative route | alternative route |
| collision | packet collision |
| following directions | routing algorithm |

## Topologies

A **topology** is the shape of the connections — physical or logical.

- **Point-to-point** — two nodes, one link. The simplest case.
- **Bus** — every node taps one shared cable; a transmission reaches all, only the addressee keeps it. A node failing is harmless; the cable failing takes down everyone. Needs terminators to stop signal reflection.
- **Star** — every node has its own link to a central hub or switch, and all traffic passes through it. One node or cable failing is contained; the centre failing isn't. The standard modern LAN.
- **Ring** — each node links to exactly two others in a circle; messages to a non-neighbour pass through the nodes in between. One break can halt the ring unless it's a dual ring.
- **Mesh** — nodes link directly to many others. *Full mesh* connects every pair (maximum redundancy, maximum cabling); *partial mesh* connects only the pairs that exchange the most traffic.
- **Tree** — a hierarchy of star segments hanging off a bus backbone. Common in campus and building networks.

## The internet: a network of networks

The **internet** is the largest network there is — a mesh of Wide Area Networks, joined to Local Area Networks and home networks. Making machines talk across it needs both hardware and software.

**Hardware**: end systems (servers, laptops, phones); a **network interface card (NIC)** to attach each one physically; **links** (twisted pair, coax, fibre, or air); **switches and routers** to interconnect networks.

**Software**: **protocols** — the rulebooks. A protocol fixes the *format* of the messages (what fields, in what order) and the *rules* for exchanging them (what to send when, how to react). A TCP connection setup is a stylised conversation: "can we talk?" / "yes, can we talk?" / "yes" — three messages with a fixed meaning.

## The network edge

The **edge** is where users and their applications sit, on machines called **end systems** or **hosts**. Two models for how they interact ([application layer](/citadel/computer-networks/application-layer) goes deeper):

- **Client-server** — clients request, an always-on server responds. The server has a fixed, well-known address; clients' addresses can change. Simple to reason about, but the server infrastructure costs money and is a bottleneck.
- **Peer-to-peer (P2P)** — end systems (peers) talk to each other directly with little or no central server. Cheap to run and scales with its users; harder to coordinate.
- **Hybrid** — a server for discovery (finding peers, tracking users), then direct peer-to-peer for the bulk data. Instant messaging and BitTorrent work this way.

## The network core

Between the edges is the **core**: interconnected routers. Two ways to push data through it.

**Circuit switching.** Reserve a dedicated path — and a fixed slice of bandwidth on every link along it — for the whole session, then tear it down. The classic telephone network. Predictable rate, but the reservation sits idle whenever you're not talking.

**Packet switching.** Chop the data into **packets**, each carrying a slice of payload plus a header with source and destination addresses. Packets from many senders share the same links (**statistical multiplexing**), and each router **stores** a whole packet before **forwarding** it toward the destination — possibly by a different route than the packet before it. The internet works this way. It's far more efficient for the bursty traffic most applications generate, at the cost of variable delay and possible reordering.

**Message switching** is the older middle ground: each switch receives and buffers an *entire message* before forwarding it. Lower per-message overhead, but store-and-forwarding the whole thing at every hop makes it slow. Packet switching keeps the store-and-forward idea but shrinks the unit.

## What "performance" means

Not one number. Four concerns, and applications weight them differently:

- **Reliability** — is delivery guaranteed? A file transfer needs every byte; a video call would rather drop a frame than wait for it.
- **Throughput** — the actual successful data rate. *Bandwidth-sensitive* apps (video) need a floor; *elastic* apps (email, web) take whatever's available.
- **Timing / latency** — *real-time* apps (telephony, conferencing) have hard delay budgets; for the rest, lower is nicer but not critical.
- **Security** — confidentiality (kept private), integrity (not altered), authentication (you're talking to who you think).

The internet's base transport protocols promise reliability (TCP) but not throughput or timing guarantees — that's what the [QoS architectures](/citadel/computer-networks/high-speed-networks) try to add.

## Why layers

Networking is split into layers, each providing a **service** to the layer above and using the service of the layer below — the way building a house splits into foundation, framing, plumbing, wiring, each trusting the last.

Two models:

| OSI (7 layers) | TCP/IP (5 layers) | Job | Examples |
| --- | --- | --- | --- |
| Application / Presentation / Session | Application | app-to-app messages | HTTP, SMTP, DNS |
| Transport | Transport | process-to-process, reliability | TCP, UDP |
| Network | Network | host-to-host routing of packets | IP, ICMP, OSPF |
| Data link | Link | node-to-node frames over one link | Ethernet, Wi-Fi, ARP |
| Physical | Physical | raw bits on the medium | 10Base-T, OFDM |

OSI is the reference vocabulary; TCP/IP is what the internet actually runs, folding OSI's presentation and session concerns into the application.

As data goes *down* the stack at the sender, each layer wraps it in that layer's header (**encapsulation**); going *up* at the receiver, each layer strips its header (**decapsulation**). A web request is an HTTP message inside a TCP segment inside an IP datagram inside an Ethernet frame — unwrapped in reverse on arrival.

## The one idea to keep

The layered model is the reason the internet could grow: fibre replaced copper without touching IP, HTTP/2 replaced HTTP/1.1 without touching TCP, because each layer only promises a service to the one above and only depends on the service below. Everything else in this series — addressing, routing, reliable delivery, congestion control, security — is one layer keeping its promise so the next one doesn't have to care how.
