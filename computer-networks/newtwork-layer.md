---
title: The Network Layer - IP Addressing and Routing Algorithms
description: How a packet crosses many networks to reach a host it has never met - the IP datagram and best-effort service, the evolution of IPv4 addressing from classes to CIDR, NAT and ICMP, and the two routing paradigms (link-state with Dijkstra, distance-vector with Bellman-Ford) that scale up through autonomous systems and BGP.
date: 2022-10-01
draft: false
slug: /computer-networks/newtwork-layer
tags:
  - Networking
  - Routing
  - TCP/IP
---

The [transport layer](/citadel/computer-networks/transport-layer) assumes it can get a packet to any host on Earth. The network layer is what makes that true — carrying a **datagram** from the source host across an arbitrary chain of routers and heterogeneous links (Ethernet here, fibre there, Wi-Fi at the end) to a destination it has no direct connection to.

Two problems: every interface needs a globally meaningful **address**, and every router needs to know which way to forward each datagram. This post is IP and its addressing scheme — classes, subnetting, CIDR, NAT — plus ICMP for diagnostics, and the routing algorithms (link-state and distance-vector) that build the forwarding tables, scaled up to internet size with autonomous systems and BGP.

## IP: glue for dissimilar networks

The **Internet Protocol**, designed by Robert Kahn and Vint Cerf to interconnect networks that don't otherwise agree on anything, provides:

- **host-to-host datagram delivery** and **routing** across intermediate routers;
- **fragmentation and reassembly** — if a datagram exceeds a link's **maximum transmission unit (MTU)**, IP splits it, using the header's Identification, Flags (Don't Fragment, More Fragments), and Fragment Offset fields, and reassembles at the destination;
- a **global address space** independent of link-layer (MAC) addresses;
- **error reporting** via its companion, ICMP.

It is **connectionless and best-effort**: no setup, no guarantee. Datagrams can be lost, corrupted, reordered, duplicated, or delayed, and IP won't notice — reliability is left to TCP. That minimalism is the point; IP runs over almost any link technology precisely because it promises so little.

The IPv4 header carries Version; Header Length (in 32-bit words, minimum 5); Type of Service; Total Length; the fragmentation fields; **Time To Live** (a hop count, often starting at 64, decremented per router, discarded at 0 — the loop-stopper); **Protocol** (a demux key: 6 for TCP, 17 for UDP); a header checksum; 32-bit source and destination addresses; and rarely-used Options.

## IPv4 addressing

A 32-bit address gives about 4.3 billion values, split into a **network part** and a **host part** so routers can forward toward a *network* without knowing every host in it.

**Classful addressing** (the original scheme) let the leading bits pick the split:

| Class | Leading bits | Network / host bits | Range |
| --- | --- | --- | --- |
| A | `0` | 8 / 24 | few networks, ~16M hosts each |
| B | `10` | 16 / 16 | ~16K networks, ~65K hosts each |
| C | `110` | 24 / 8 | ~2M networks, 254 hosts each |
| D | `1110` | — | multicast |
| E | `1111` | — | experimental |

`127.0.0.0/8` is reserved for loopback. The rigidity wasted enormous space: a 2-host network still needed a full Class C (254 host slots), a 260-host network jumped to a Class B (65,534).

**Subnetting** splits the host part into a **subnet ID** and a host ID, so one allocated block becomes several internal networks. A **subnet mask** is 32 bits — 1s over the network and subnet portions, 0s over the host portion — and `IP AND mask` yields the **subnet (network) address**. Within a subnet, all-zero host bits name the subnet itself and all-one host bits are its broadcast address; neither is assignable.

> IP `154.71.150.42`, mask `255.255.248.0`:
> `10011010.01000111.10010110.00101010`
> `11111111.11111111.11111000.00000000`
> AND → `10011010.01000111.10010000.00000000` = `154.71.144.0`

**VLSM** (variable-length subnet masking) drops the requirement that all subnets be equal-sized — allocate the largest subnet first, borrowing more subnet bits for smaller ones.

**CIDR** (classless inter-domain routing) removes classes entirely. An address is written `A.B.C.D/X`, where `X` is the prefix length (equivalently, the count of leading 1s in the mask): `222.7.16.0/20` has mask `255.255.240.0`. Blocks can be any power-of-two size, and contiguous blocks whose leading bits match can be **aggregated** (**supernetting**) — 16 adjacent `/24`s advertised as one `/20`, shrinking routing tables.

## NAT

Even with CIDR, public addresses ran short. **Network Address Translation** lets a whole private network share one public address. Private ranges — `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — are not routable on the public internet. A NAT router at the boundary rewrites each outbound packet's private source IP (and usually its source port) to the router's public IP and a chosen port, recording the mapping in a **translation table**; replies to that public IP and port are rewritten back. The common form, **NAPT / PAT**, distinguishes many internal hosts by port number alone. It conserves addresses and hides internal structure, but breaks protocols that assume end-to-end reachability or embed IP addresses in their payload.

## ICMP

IP has no built-in way to tell a source that something went wrong. **ICMP** fills that gap — carried inside IP datagrams but handled by the IP software itself. It reports errors (destination host / network / port / protocol unreachable, TTL exceeded, fragmentation needed with DF set) and supports diagnostics (**ping** uses Echo Request / Echo Reply). A message is a **type**, a **code**, a checksum, and the header plus first 8 bytes of the datagram that triggered it. To avoid floods, ICMP errors are *not* generated for other ICMP errors, for broadcast/multicast datagrams, or for non-first fragments.

**Traceroute** exploits TTL: send datagrams with TTL 1, 2, 3, … Each expires one hop further along, and that router returns an ICMP **Time Exceeded** (Type 11, Code 0), revealing its address and the round-trip time. When the packets finally reach the destination, it replies with **Port Unreachable** (Type 3, Code 3), ending the trace.

## Routing algorithms

A router forwards by looking up a datagram's destination in its **routing table**, built by a routing algorithm. Algorithms vary along three axes: **static** (hand-configured) vs **dynamic** (self-updating); **centralized** vs **distributed**; **global knowledge** (each router has the whole topology) vs **local** (each knows only its neighbours). The internet uses dynamic, distributed algorithms of both knowledge kinds.

### Link-state (OSPF)

Every router ends up with a complete map. Two phases:

1. **Reliable flooding.** A router measures the cost to each directly connected neighbour, packages it into a **link-state packet (LSP)** — its ID, its neighbour list with costs, a sequence number, a TTL — and floods it to every other router. A newer LSP (higher sequence number) replaces the stored one and is forwarded on; old or duplicate LSPs are dropped. LSPs go out on link-state changes and, rarely, on a long timer. Every router builds an identical graph.
2. **Shortest paths with [Dijkstra](/citadel/algorithms/PathFinding).** From that graph, compute least-cost paths to every destination. Maintain a set `N` of nodes whose shortest path is settled (start: just the source). Each step, take the node `w` not in `N` with the smallest current cost `D(w)`, add it to `N`, and relax its neighbours: `D(v) = min(D(v), D(w) + c(w, v))`.

Consistent and loop-resistant, at the cost of CPU, memory, and flooding bandwidth.

### Distance-vector (RIP)

Each router knows only its neighbours and a **distance vector**: for every destination, an estimated cost and a next hop. Initially it knows only direct-neighbour costs. Periodically it sends its whole vector to its neighbours, and on receiving neighbour `v`'s vector it updates each destination `y` with the **Bellman-Ford** equation:

$$D_x(y) = \min_{v \in \text{neighbours}(x)} \{\, c(x, v) + D_v(y) \,\}$$

If its own vector changes it re-sends. This converges — but suffers **count-to-infinity**: when a link fails, the "bad news" propagates slowly while routers keep hearing stale good news from each other, forming transient loops with costs creeping upward. **Split horizon** (don't advertise a route back toward the neighbour you learned it from) and **poison reverse** reduce, but don't eliminate, the problem.

### Hierarchical routing

A flat internet where every router knows every network is unscalable — table size and update traffic explode, and organisations want to run their own networks. The fix: divide the internet into **autonomous systems (ASes)**, each under one administrative authority.

- **Intra-AS** routing uses an **interior gateway protocol (IGP)** — RIP or OSPF — within an AS.
- **Inter-AS** routing uses an **exterior gateway protocol (EGP)** between ASes. **Gateway (border) routers** speak both.

A gateway learns via the EGP which destination networks are reachable through which neighbouring AS, propagates that reachability to the AS's internal routers, and each internal router then uses its *intra-AS* costs to pick the closest exit gateway — **hot-potato routing**, get the packet out of your own AS as cheaply as possible.

**RIP** is a distance-vector IGP: hop-count metric, maximum 15 (16 means unreachable), updates every 30 seconds, slow to converge. **BGP** is the inter-AS standard — a **path-vector** protocol that advertises the full sequence of ASes a route traverses, which lets it detect loops at the AS level and lets operators enforce policy (don't route through a competitor, prefer a peering link). BGP runs over TCP for reliability. See the [internet routing](/citadel/interview/internet-routing) post for how BGP behaves in practice.

### Broadcast and multicast

Unicast routing is one-to-one. **Broadcast** delivers to *every* node — naively $N$ separate copies, better via controlled flooding or a spanning tree. **Multicast** delivers to a *subscribed subset*, which needs group membership tracking and a distribution tree reaching only the interested receivers.

## The one idea to keep

IP addressing and routing are both exercises in aggregation. CIDR lets one routing-table entry stand for a whole range of addresses, so the core routers hold hundreds of thousands of routes instead of billions. Autonomous systems let one BGP advertisement stand for a whole network's worth of internal structure, so an AS's routing is its own business. Link-state and distance-vector are just two ways to compute shortest paths — the scaling comes from not having to compute them over the entire internet at once.
