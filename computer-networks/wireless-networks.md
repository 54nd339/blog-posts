---
title: Wireless Networks - Wi-Fi, Cellular Generations, WiMAX, and VoIP
description: How connectivity works without cables - the shared-spectrum fundamentals of cells, reuse, and handoff, the 802.11 Wi-Fi family, the 3G-to-LTE cellular architecture, WiMAX as an alternative, and how voice is carried as packets with RTP and SIP.
date: 2022-10-09
draft: false
slug: /computer-networks/wireless-networks
tags:
  - Networking
  - Wireless
  - Cellular
---

Every wireless technology shares one hard constraint: the medium is open air, a fixed slice of radio spectrum that everyone nearby is also using. From that flow the recurring ideas — divide the spectrum among users, reuse frequencies in distant cells, hand a moving device from one cell to the next without dropping its session.

This post walks the main wireless systems on top of those fundamentals: Wi-Fi and its 802.11 standards, the cellular progression from 3G through LTE with its all-IP core, WiMAX as a road not taken, and Voice over IP, which turns a phone call into a packet stream.

## Fundamentals

Wireless links carry data on radio waves (or infrared), trading a cable for mobility and reach. The shared spectrum forces a common toolkit:

- **Cells** — geographic areas each served by a base station.
- **Frequency reuse** — the same frequencies used again in non-adjacent cells, to stretch limited spectrum.
- **Handoff / handover** — transferring an active call or data session between cells as a user moves.
- **Multiple access** — **FDMA** (frequency), **TDMA** (time), **CDMA** (codes), **OFDMA** (orthogonal subcarriers) to let many users share the band.

The generations trace a line from Hertz and Marconi's experiments through analogue 1G, digital 2G, and the mobile-internet era of 3G, 4G/LTE, and 5G — each step adding data rate and cutting latency. Wired fibre and coax still do the **backhaul**, carrying traffic from towers and access points to the core; wireless is usually just the last hop. Bodies like the ITU, 3GPP, and IEEE **harmonise** standards and spectrum so equipment interoperates worldwide.

## Wi-Fi (802.11)

A family of IEEE standards for wireless LANs, branded Wi-Fi:

| Standard | Band(s) | Peak rate | Notes |
| --- | --- | --- | --- |
| 802.11a | 5 GHz | 54 Mbps | |
| 802.11b | 2.4 GHz | 11 Mbps | |
| 802.11g | 2.4 GHz | 54 Mbps | b-compatible |
| 802.11n (Wi-Fi 4) | 2.4/5 GHz | 600 Mbps | MIMO |
| 802.11ac (Wi-Fi 5) | 5 GHz | gigabit+ | wider channels, more MIMO streams |
| 802.11ax (Wi-Fi 6/6E) | 2.4/5/6 GHz | higher | OFDMA, MU-MIMO, dense-environment efficiency |
| 802.11be (Wi-Fi 7) | 2.4/5/6 GHz | higher still | lower latency |

Wi-Fi works at the physical and MAC layers, using **CSMA/CA** for channel access — the mechanics (RTS/CTS, DIFS/SIFS, backoff, the hidden-terminal problem) are in the [link layer](/citadel/computer-networks/datalink-layer) post. It runs in the unlicensed 2.4 and 5 GHz ISM bands, plus 6 GHz for 6E. **Modulation and coding schemes** range from BPSK through QPSK to high-order QAM — more bits per symbol, but needing a cleaner signal — with coding adding redundancy for error correction.

Architecture:

- **Basic Service Set (BSS)** — a group of devices communicating. An **independent BSS (IBSS)** is ad-hoc, peer-to-peer; an **infrastructure BSS** routes everything through an **Access Point** connected to a wired network (the common case).
- **Extended Service Set (ESS)** — several BSSs linked by a distribution system, giving larger coverage and roaming.

Security evolved from the broken **WEP** through interim **WPA** (TKIP) to **WPA2** (AES-CCMP) and **WPA3** (SAE, resistant to offline password guessing) — detailed in [network security](/citadel/computer-networks/network-security), along with the **VPN** tunnels used to protect traffic on untrusted hotspots. Cellular and Wi-Fi also cooperate: **Wi-Fi offloading** moves mobile data onto Wi-Fi to relieve cellular networks, and **Wi-Fi calling (VoWiFi)** places cellular voice calls over Wi-Fi.

## 3G

3G aimed at universal mobile service — voice, data, multimedia — at broadband-ish speeds.

**UMTS**, the GSM-lineage standard, uses **WCDMA** (wideband direct-sequence spread-spectrum CDMA), letting many users share a frequency band by unique codes. Its network (3GPP Release 99): **User Equipment** → **UTRAN** (NodeB base stations, Radio Network Controllers) → a **core network** with a circuit-switched domain for voice (MSC, GMSC) and a packet-switched domain for data (SGSN, GGSN). Release 4 split the MSC into a server plus a Media Gateway; Release 5 introduced the all-IP **IP Multimedia Subsystem (IMS)** core.

Other 3G branches: **CDMA2000** (evolved from cdmaOne, with EV-DO data), **TD-CDMA**, and China's **TD-SCDMA** (time-division duplex, synchronous CDMA). All were part of the ITU's IMT-2000 vision, differing mainly in air interface.

## 4G LTE

LTE (marketed as 4G) is all-IP, with much higher throughput and lower latency. The **Evolved Packet System**:

- **UE** — the device.
- **E-UTRAN** — the radio access network, built from **eNodeB** base stations that also handle radio resource management.
- **Evolved Packet Core (EPC)** — all-IP core:
  - **MME** — mobility, session setup, authentication, tracking.
  - **S-GW** — routes user data; the mobility anchor during inter-eNodeB handovers.
  - **P-GW** — connects the UE to external packet networks and assigns its IP address.
  - **HSS** — the subscriber and authentication database.
  - **PCRF** — policy control and charging.

The radio uses **OFDMA** on the downlink and **SC-FDMA** on the uplink, in FDD or TDD (**TD-LTE**) mode, with **MIMO** for throughput (spatial multiplexing) and reliability (diversity). The eNodeB **scheduler** allocates resource blocks to UEs by channel quality, QoS, buffer state, and fairness. **Carrier aggregation** combines multiple frequency blocks for more bandwidth. A UE performs **cell search** and, when idle, **cell reselection**; it **attaches** and activates a default bearer to get packet service. **Handover** between eNodeBs runs over the direct **X2** interface (fast) or via the MME on **S1** (including inter-MME).

**LTE-Advanced** (Release 10+) added carrier aggregation, enhanced MIMO, and Coordinated Multi-Point. Supporting features: **SON** (self-organizing networks) to automate planning and healing, **relay nodes** to extend coverage, **HetNets** mixing macro/micro/pico/femto cells, **remote radio heads** on fibre at the tower top, and **VoLTE** — voice over the LTE packet core via IMS, with HD audio and fast call setup.

## WiMAX

Based on **IEEE 802.16** (802.16d fixed, 802.16e mobile), WiMAX offered broadband wireless over longer ranges than Wi-Fi. Its architecture: a **Subscriber/Mobile Station**, a **Base Station**, an **Access Service Network** (base stations plus ASN gateways for radio resource and handover management), and a **Connectivity Service Network** for IP connectivity and AAA. It uses OFDM/OFDMA in licensed and unlicensed bands. LTE overtook it for mobile broadband; it survives mainly in fixed wireless "last mile" deployments.

## Voice over IP

VoIP digitises voice, compresses it, breaks it into packets, and sends it over IP — using **RTP** for the media stream and a separate protocol for signalling. It's cheaper than circuit-switched telephony and integrates with other IP services, but IP's best-effort delivery creates challenges: **latency** (conversational lag), **jitter** (needs a receiver jitter buffer), **packet loss** (degrades audio, mitigated by concealment), and **security**.

Signalling:

- **H.323** — an early, comprehensive, complex ITU standard, with terminals, gateways, gatekeepers, and MCUs.
- **SIP** — the IETF's text-based protocol for creating, modifying, and ending sessions (voice, video, messaging). Simpler and more flexible than H.323, and now dominant. Components: User Agents (client and server), proxy servers, redirect servers, registrar servers.

In large deployments, signalling and media are separated: a **Media Gateway** converts between PSTN TDM and IP RTP, and a **Media Gateway Controller (softswitch)** runs the call logic, talking to gateways via MGCP or Megaco/H.248. Interworking with the traditional telephone network's **SS7** signalling uses SIGTRAN gateways. Voice quality depends on the codec (G.711, G.729, Opus), jitter buffering, loss concealment, and prioritising voice packets with [DiffServ](/citadel/computer-networks/high-speed-networks) markings.

## The one idea to keep

Wireless is spectrum management dressed up in different acronyms. Wi-Fi contends for an unlicensed band with CSMA/CA; cellular schedules a licensed band from the base station; both reuse frequencies across cells and hand moving devices between them. LTE's real contribution was less the radio than the core — collapsing voice and data onto one all-IP network, which is the same convergence VoIP performs at the application level.
