---
title: Mobile and Wireless Communication - Cellular, Mobile IP, and MANETs
description: How data moves to and from a device that keeps moving. The components of a cellular system and the GSM architecture, the MAC protocols that let devices share the air, Mobile IP for keeping one address across networks, why TCP struggles over wireless, the cellular generations, and infrastructure-less MANETs and VANETs.
date: 2023-07-08
draft: false
slug: /miscs/mobile-computing
tags:
  - Mobile Computing
  - Networking
---

Wireless communication is the infrastructure — radio links, base stations, protocols — that carries data between a moving device and the rest of the network. **Mobile computing** is the use of that infrastructure to run applications while on the move. This post covers the components of a mobile network, how devices share a shared radio medium, how a device keeps its identity while roaming, and what happens when there is no fixed infrastructure at all.

## The components

- **Mobile handsets** — handheld computers with processors, memory, sensors, and radios; the primary gateway to networked services.
- **The cell system** — networks divide an area into **cells**, each served by a **base station**. As a device moves, it **hands off** between cells, coordinated by a **Mobile Switching Centre (MSC)**, so service is continuous.
- **Network types** — PSTN (traditional landline), ISDN (early digital telephony), cellular (4G, 5G), satellite (remote coverage). Computer networks are classified by span: **LAN** (a building), **MAN** (a city), **WAN** (the internet).
- **A wireless system** is a **transmitter** (encodes information onto a radio wave), a **receiver** (decodes it), **antennas**, and the **channel** (air/space).

### GSM architecture

- **MS (Mobile Station)** — the handset and SIM.
- **BSS (Base Station Subsystem)** — the radio link. **BTS** is the tower's radio; **BSC** controls several BTSs.
- **NSS (Network Switching Subsystem)** — the core. **MSC** routes calls and handoffs; **HLR** holds subscriber data; **VLR** is a temporary register for roamers in its area; **EIR** tracks valid and stolen handsets; **AuC** handles authentication.

### Wireless standards

- **IEEE 802.11 (Wi-Fi)** — WLANs; versions a/b/g/n/ac/ax differ in speed and frequency. Access Points broadcast an **SSID** to connect devices to a wired LAN. See [wireless networks](/citadel/computer-networks/wireless-networks).
- **IEEE 802.15** — personal-area networks: **Bluetooth** (802.15.1, short-range peripherals) and **Zigbee** (802.15.4, low-power sensors and home automation).

## MAC protocols: sharing the air

The radio medium is shared; simultaneous transmissions on the same frequency collide. **Medium Access Control** protocols set the rules. See also [the data link layer](/citadel/computer-networks/datalink-layer).

**Wireless-specific problems:**

- **Hidden terminal** — A and C can both reach an access point but cannot hear each other, so they transmit at once and collide at the AP.
- **Exposed terminal** — B is transmitting to A; C hears B and holds off, even though its transmission to D would not have interfered.

**Approaches:**

- **Fixed assignment** — each user gets a dedicated slice: **FDMA** (frequency bands), **TDMA** (time slots), **CDMA** (same frequency, unique codes).
- **Random access** — contend for the channel: **ALOHA** (transmit anytime; on collision, wait a random interval and retry), **CSMA** ("listen before talk"). CSMA/CD (collision detection) is used in wired Ethernet; CSMA/CA (collision avoidance, with RTS/CTS handshakes) is used in Wi-Fi, because detecting a collision mid-transmission is hard on radio.
- **Reservation** — nodes reserve future slots; good for real-time voice and video.

**Cognitive radio** senses its spectrum environment, finds unused bands (whitespace), and adapts its frequency and power to use them without disturbing licensed users — especially useful when the radios form an infrastructure-less ad-hoc network.

## Mobile computing characteristics

- **Mobility** — the defining trait.
- **Limited resources** — less battery, CPU, screen, and memory than a desktop.
- **Intermittent connectivity** — wireless links drop.
- **Dynamic environment** — network conditions and available resources shift quickly.

Applications are usually client-server: a lightweight client on the device, heavy processing and storage on a server.

## Mobile IP: one address while roaming

**Mobile IP** lets a device (Mobile Node, MN) move between networks while keeping its permanent **home address**, transparently to TCP and applications.

- **Home Agent (HA)** — a router on the home network that tunnels packets to the MN when it is away.
- **Foreign Agent (FA)** — a router on the visited network assisting the MN (less common in modern Mobile IPv6, where the MN gets a care-of address directly).
- **Care-of Address (CoA)** — a temporary address on the visited network; tells the HA where to forward.
- **Tunnelling (IP-in-IP)** — the HA intercepts packets for the home address, wraps them in a header addressed to the CoA, and forwards them.
- **Route optimisation** — without it, correspondent-to-MN packets detour through the HA (triangle routing); optimisation lets the correspondent learn the CoA and send directly, cutting latency.

## TCP over wireless

The [transport layer](/citadel/computer-networks/transport-layer) TCP assumes packet loss means congestion. On wireless, loss often means bit errors or a brief disconnection. TCP misreads this as congestion, shrinks its window, and underuses the link. Fixes:

- **Snoop protocol** — an agent at the base station caches TCP segments and retransmits lost ones locally, hiding wireless loss from the sender.
- **I-TCP (Indirect TCP)** — splits the connection at the base station: a wired half and a wireless half, with the base station as proxy.
- **M-TCP** — similar, but preserves end-to-end TCP semantics more closely and handles handoffs.

## Cellular generations

- **GSM** — the dominant **2G** digital standard; digital voice and SMS, TDMA access, introduced the **SIM**.
- **GPRS (2.5G)** — adds **packet-switched** data to GSM: always-on connections billed by data, enabling early mobile internet browsing and MMS.
- **UMTS (3G)** — **W-CDMA**-based; much higher data rates, mobile broadband and video calls.
- Since then: 4G/LTE for higher speeds, and 5G for ultra-low latency, massive device density, and VR/AR/[IoT](/citadel/miscs/pervasive-computing)-scale connectivity — the last of which overlaps [real-time systems](/citadel/miscs/realtime-systems) needs.

## Networks with no infrastructure: MANETs

A **Mobile Ad-hoc Network** forms spontaneously among mobile nodes with no routers or access points; nodes route for each other.

- **Characteristics** — dynamic topology, infrastructure-less, limited resources, multi-hop routing, energy constraints.
- **Applications** — disaster response, military communications, conference networks, sensor networks.
- **Design issues** — routing (the hardest), security (open medium, no central authority), QoS, power, scalability.

### Routing in MANETs

Traditional protocols struggle: **distance vector** (RIP) has slow convergence and counting-to-infinity; **link state** (OSPF) floods link information, which is heavy in a changing topology. MANET protocols:

- **Proactive (table-driven)** — maintain routes to everyone continuously; example **DSDV**. Low route-discovery latency, high overhead.
- **Reactive (on-demand)** — discover a route only when data needs to flow; examples **AODV**, **DSR**. Lower overhead, higher initial latency.
- **Hybrid** — proactive within a local zone, reactive outside; example **ZRP**.

### VANETs

A **Vehicular Ad-hoc Network** is a MANET whose nodes are vehicles, enabling vehicle-to-vehicle (V2V) and vehicle-to-infrastructure (V2I) communication.

- **Versus MANETs** — higher but road-constrained (more predictable) mobility, no serious power limits, highly variable density (traffic jam versus empty highway), shorter-lived interactions.
- **Applications** — collision and hazard warnings, traffic management, infotainment.

### Security in MANETs and VANETs

Vulnerable to eavesdropping, message alteration, spoofing, denial of service, and routing attacks (blackhole, wormhole). No central trust authority and constant topology change make defence hard; mechanisms include authentication, encryption, ad-hoc intrusion detection, and reputation systems.

## Key takeaways

- A cellular network hands a moving device between cells; GSM structures this into the MS, the BSS (BTS, BSC), and the NSS (MSC, HLR, VLR).
- **MAC protocols** manage a shared radio medium against the hidden- and exposed-terminal problems; Wi-Fi uses CSMA/CA.
- **Mobile IP** keeps one address across networks via a home agent and a care-of address; **TCP** needs help (Snoop, I-TCP) to not mistake wireless loss for congestion.
- **MANETs and VANETs** drop fixed infrastructure entirely, which turns routing into the central problem — solved proactively (DSDV), reactively (AODV, DSR), or hybrid (ZRP).
