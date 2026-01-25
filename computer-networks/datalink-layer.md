---
title: The Link Layer - Bridging the Gap from Bits to Networks
description: Exploring the link layer - error control mechanisms, multiple access protocols (ALOHA, CSMA/CD/CA), the intricacies of Ethernet and VLANs, the challenges of Wi-Fi (802.11 MAC), and how data moves locally.
date: 2023-06-05
draft: false
slug: /pensieve/computer-networks/datalink-layer
tags:
  - Computer Networks
  - CS Basics
---

Hello again, network explorers! We've seen how applications generate data (Application Layer), how it's prepared for its journey (Transport Layer), and how it's routed across the vast internet (Network Layer). Now, let's zoom in on the **Link Layer**. This layer is responsible for moving data packets, called **frames** at this layer, between two physically adjacent nodes over a specific link.

Think of it as the local courier service that handles the "last mile" or, more accurately, the "next hop" delivery. Whether it's from your computer to your Wi-Fi router, or between two routers in the internet's backbone, the link layer makes it happen.

---
## Link Layer: An Introduction

In our network journey, hosts and routers are **nodes**, and the communication channels connecting these adjacent nodes are **links**. These links can be wired (like Ethernet cables) or wireless (like Wi-Fi). The link layer's primary responsibility is to transfer a network layer datagram from one node to a physically adjacent node over one such link.

### Key Link Layer Services:
The link layer is often conceptually divided into two sublayers:
* **Logical Link Control (LLC):** This sublayer acts as an interface between the network layer and the MAC sublayer. It can provide services like:
    * Error Detection
    * Error Recovery (optional)
    * Flow Control (optional)
* **Media Access Control (MAC):** This sublayer controls how nodes access and share the physical media, especially in broadcast channels where multiple nodes share the same link. Its key functions include:
    * Framing (encapsulating datagrams into frames with headers and trailers)
    * Switching (interconnecting LANs)

Link layer protocols define the format of frames and the actions nodes take in response to these frames. Examples include Ethernet, Wi-Fi (802.11), and PPP.

---
## Finding Your Neighbor: ARP (Address Resolution Protocol)

The network layer uses IP addresses to identify source and destination hosts globally. However, when it comes to delivering a frame over a specific physical link (like an Ethernet LAN), devices use physical addresses, also known as MAC (Media Access Control) addresses. So, how does a device translate a known IP address (of the next hop, which could be a router or the final destination if it's on the same local network) into its corresponding MAC address? This is where ARP comes in.

ARP operates at the Link Layer (its frames have a specific type, e.g., 0x0806 for Ethernet).
* **Purpose:** To resolve an IP address to a MAC address for hosts on the same physical network.
* **Operation:**
    1.  **ARP Request (Broadcast):** When a host (say, Host A) wants to send an IP datagram to another host (Host B) on the same network but doesn't know Host B's MAC address, Host A broadcasts an ARP request message. This message essentially asks, "Who has IP address [IP_B]? Tell [MAC_A]." This broadcast is received by all devices on the local network segment.
    2.  **ARP Reply (Unicast):** The host on the network that has the IP address [IP_B] (i.e., Host B) recognizes its IP address in the request and sends an ARP reply directly (unicast) back to Host A. This reply contains Host B's MAC address: "I have IP [IP_B], and my MAC address is [MAC_B]."
    3.  **ARP Cache:** Hosts maintain an ARP cache (a table) of recently resolved IP-to-MAC address mappings. Before sending an ARP request, a host first checks its cache. If a mapping is found, it uses it. Entries in the cache are typically timed out periodically (e.g., around 15 minutes) to ensure freshness.
        * When the originator (Host A) receives the ARP reply, it adds the IP-to-MAC mapping for Host B to its cache.
        * The target host (Host B), upon receiving the ARP request from Host A, can also add Host A's IP-to-MAC mapping to its cache (since Host A's IP and MAC addresses are in the ARP request).
* **Usage:** When a device needs to forward a datagram, it checks its ARP cache for the next-hop IP address's MAC address. If no mapping is found, it initiates the ARP protocol.

### Gratuitous ARP
A Gratuitous ARP is a special ARP broadcast a host sends about its own IP-to-MAC address mapping, without being prompted by an ARP request. It can be formatted as either a request or a reply. In a gratuitous ARP, the source IP address and the destination (target) IP address in the ARP packet are both set to the IP address of the machine generating the gratuitous ARP, and the target MAC address is often the broadcast MAC address (ff:ff:ff:ff:ff:ff).
**Uses of Gratuitous ARP:**
* Issued when an IP or MAC address of an interface changes or when an interface is brought up from a down state.
* Helps other hosts on the network update their cached ARP entries for the sender.
* Can be used to detect IP address conflicts (if another host replies claiming to own that IP address).

---
## Making Data Robust: Error Detection and Correction

Data can get corrupted during transmission due to noise or interference. The link layer employs mechanisms to detect and sometimes even correct these errors. This involves adding **Error Detection and Correction (EDC) bits** (redundancy) to the data.

### 1. Parity Checks
* **Single Parity Bit:** A simple scheme where an extra bit is added to a block of data to make the total number of 1s either even (even parity) or odd (odd parity). The receiver checks the parity. If it's wrong, an error is detected. However, a single parity bit can only detect an odd number of bit errors. If two bits flip, it will look correct!
* **Two-Dimensional Parity:** Data is arranged in a table, and parity bits are calculated for each row and each column. This can detect and even correct single-bit errors because the intersection of the row and column with incorrect parity pinpoints the erroneous bit. It can detect but not correct some two-bit errors.

### 2. Checksum
* The sender treats the data as a sequence of 16-bit integers and sums them using one's complement arithmetic. Each time a carry-out is produced, it's added back into the sum. The one's complement of this final sum is the checksum, which is sent with the data. The receiver performs the same calculation on the received data (including the received checksum, with the checksum field itself treated as zero during its own calculation if it's verifying) and expects the final sum to be all 1s (or all 0s if the transmitted checksum was part of the calculation that should result in all 0s).
* It's not very strong in detecting errors, especially certain pairs of errors.
* **Why is it used in Transport/Network Layers (like TCP/UDP/IP) if it's not that strong?**
    * It's very easy to implement in software.
    * The assumption is that more robust error detection (like CRC) at the link layer will catch most errors.

### 3. Cyclic Redundancy Check (CRC)
* CRC is a powerful error detection mechanism used by many link-layer protocols like Ethernet, 802.11, and HDLC.
* It's based on polynomial arithmetic in finite fields.
* The message bits are treated as a polynomial $M(x)$. Both sender and receiver agree on a **generator polynomial** $C(x)$ of degree $k$. The choice of $C(x)$ is crucial for CRC's effectiveness. Ethernet, for example, uses a 32-bit CRC. A generator must have at least 2 bits, and its rightmost and leftmost bits should be ones.
* **Calculation:**
    1.  Append $k$ zero bits to the message, making it $M(x) \cdot x^k$.
    2.  Divide $M(x) \cdot x^k$ by $C(x)$ using polynomial long division.
    3.  The remainder $R(x)$ (which will be $k$ bits long) is the CRC.
    4.  The sender transmits the original message bits plus this CRC remainder.
* The receiver divides the received frame (data + CRC) by the same $C(x)$. If the remainder is zero, no error is detected. If non-zero, an error occurred.
* **Capabilities:** Qualified CRC generators can detect all single-bit errors. They can also detect all burst errors of length $L \le k$ (where $k$ is the degree of the generator polynomial, i.e., the length of the remainder). Burst errors of length $L=k+1$ are detected with probability $1-(0.5)^{k-1}$, and burst errors of length $L > k+1$ are detected with probability $1-(0.5)^k$.

### Error Control Strategies
Once an error is detected, what next?
* **Drop Frame:** Simply discard the corrupted frame. Higher layers (like TCP) might handle recovery, or for some applications (like streaming audio), a few lost frames might not be critical.
* **Error Correction (Forward Error Correction - FEC):** The frame carries enough redundant information for the receiver to correct errors without retransmission.
* **Retransmission (Backward Error Correction):** The receiver signals the sender about the error (e.g., via a NAK or absence of ACK), and the sender retransmits the frame.

Error correction requires more overhead (more EDC bits per frame) than just detection. It's useful when error rates are high (like in wireless) or when retransmission latency is too high (like satellite links).

#### Error Correction using Hamming Distance
* **Hamming distance** between two codewords is the number of bit positions in which they differ.
* To detect up to $d$ errors, you need a code with a minimum Hamming distance of $d+1$. To correct up to $d$ errors, you need a minimum Hamming distance of $2d+1$.
* For single-bit error correction, we need to determine the number of redundant bits ($r$) for a given number of data bits ($d$). The relationship $2^r \ge d+r+1$ must hold. This ensures that $r$ bits can represent all possible $d+r$ single-bit error locations plus the no-error state. For example, if $d=7$, then $r=4$ is the smallest value satisfying this.
* **Hamming Code Placement:** Parity bits are typically placed at positions that are powers of two (1, 2, 4, 8, ...). Each parity bit checks specific data bit positions based on their binary representation.
* At the receiver, parity checks are re-calculated. The resulting pattern of correct/incorrect parity checks (called the **syndrome**) indicates if an error occurred. If the syndrome is all zeros, no error. If it contains only one bit set to 1, the error is in a check bit itself. If more than one bit is set to 1, the numerical value of the syndrome indicates the position of the data bit in error.

---
## Multiple Access Links and Protocols: Sharing the Channel

Link layer protocols can operate over two types of links:
* **Point-to-Point:** A single sender and a single receiver (e.g., PPP for dial-up, or a direct link between an Ethernet switch and a host).
* **Broadcast (Shared Medium):** Multiple nodes are connected to the same communication channel (e.g., older bus-topology Ethernet, Wi-Fi, satellite links).

When multiple nodes share a broadcast channel, if two or more transmit simultaneously, their signals interfere – this is a **collision**. **Media Access Control (MAC) protocols** are needed to coordinate transmissions and determine when a node can transmit.

### Categories of MAC Protocols:

1.  **Channel Partitioning Protocols:**
    * Divide the channel into smaller "pieces" (e.g., time slots in TDMA, frequency bands in FDMA, codes in CDMA).
    * Allocate a piece to each node for its exclusive use.
    * Efficient and fair at high load but can be inefficient at low load if allocated pieces go unused. TDMA with FDM is used in GSM and WiMAX; CDMA is used in IS-95 and 3G/4G.

2.  **Random Access Protocols:**
    * Allow nodes to transmit at the full channel rate.
    * No prior coordination among nodes.
    * If a collision occurs, the protocol specifies how to detect it and how to recover (e.g., by retransmitting after a random delay).
    * Efficient at low load but can suffer from many collisions (and thus wasted capacity) at high load. Used in Ethernet and Wi-Fi.

    * **ALOHA (Pure ALOHA):**
        * When a frame arrives, transmit immediately.
        * **Vulnerable time:** If frame transmission time is $T_{fr}$, a collision occurs if another node transmits in the interval $[t_0 - T_{fr}, t_0 + T_{fr}]$ around the start time $t_0$ of our transmission. So, vulnerable period = $2 \times T_{fr}$.
        * Maximum efficiency (throughput S in terms of offered load G, $S=Ge^{-2G}$) is about 18%.

    * **Slotted ALOHA:**
        * Time is divided into equal slots (equal to $T_{fr}$).
        * Nodes can only transmit at the beginning of a slot.
        * **Vulnerable time** is reduced to $1 \times T_{fr}$ (collision only if another node transmits in the *same* slot).
        * Maximum efficiency improves to about 37% ($S=Ge^{-G}$).

    * **Carrier Sense Multiple Access (CSMA):** "Listen before talk".
        * If channel is busy, defer transmission. If idle, transmit.
        * Reduces collisions but doesn't eliminate them (due to propagation delay – two nodes might sense idle and transmit almost simultaneously).
        * **1-Persistent CSMA:** If busy, wait until idle, then transmit immediately. If collision, wait random time and restart. (Employed by Ethernet ).
        * **Non-Persistent CSMA:** If busy, wait a random amount of time *before sensing again*. Better channel utilization than 1-persistent but longer delays. (Used in 802.15.4 ).
        * **P-Persistent CSMA:** If idle, transmit with probability $p$; otherwise, defer to the next slot with probability $1-p$, repeating until sent or channel becomes busy. If busy, wait until idle, then repeat. If collision, wait random time and try again. (Employed by 802.11 Wi-Fi ).

    * **CSMA with Collision Detection (CSMA/CD):** "Listen while talk".
        * An enhancement to CSMA used in traditional Ethernet MAC.
        * If a transmitting node detects a collision, it immediately stops transmitting its frame to reduce wastage.
        * It then sends a brief jam signal and then backs off for a random amount of time before retrying. Collisions can occur if two stations sense an idle channel and transmit at slightly different times or at the same time.

3.  **Controlled Access Protocols:**
    * Aim to combine the efficiency of random access at low loads with the efficiency of channel partitioning at high loads.

    * **Polling (Centralized):**
        * A master node (coordinator) invites slave nodes to transmit in turn.
        * Eliminates collisions and empty slots, achieving high efficiency.
        * Disadvantages: Polling delay, single point of failure (the coordinator). (Used in Bluetooth, 802.15 ).

    * **Token Passing (Decentralized):**
        * A special control frame (token) is passed sequentially from node to node in a predefined order.
        * Only the node holding the token is allowed to transmit.
        * Highly efficient and decentralized.
        * Disadvantages: Token overhead, single point of failure (token loss/corruption), failure of one node can crash the channel. (Used in Token Ring, FDDI ).

---
## Switched Local Area Networks (LANs)

Early Ethernet used a bus topology with coaxial cables (Thicknet 10Base5, Thinnet 10Base2). This had problems: cable breaks affected the whole network, and adding/removing nodes was disruptive. This led to the dominance of **star topology**, where nodes connect to a central device.

### Hubs vs. Switches
* **Hubs:**
    * Physical layer "dumb" repeaters.
    * Bits coming in one link are simply regenerated and sent out on all other links.
    * All nodes connected to a hub are in the same **collision domain** (they can collide with one another).
    * No frame buffering or MAC protocol at the hub. Cannot connect links of different speeds.

* **Layer-2 Switches (Ethernet Switches/Bridges):**
    * Smarter link-layer devices, taking an active role.
    * They store and forward Ethernet frames.
    * Examine incoming frame's MAC addresses and selectively forward the frame to one or more output links.
    * **Break up collision domains:** Each port on a switch is its own collision domain. This means multiple nodes can transmit simultaneously without collision, as long as they are on different ports and destined for different ports (or the switch can buffer).
    * Run MAC protocol on each interface and can buffer packets.
    * Can support different speeds on different ports (e.g., 10Mbps and 100Mbps).
    * **Self-Learning (Switching Algorithm):**
        1.  When a frame arrives on a port, the switch inspects the source MAC address. If it's not in its MAC address table (or if the entry is outdated), it adds/updates an entry mapping that MAC address to the incoming port and sets a timer.
        2.  It then looks up the destination MAC address in its table:
            * If no entry for the destination MAC, the switch **floods** the frame – forwards it out all ports *except* the one it arrived on.
            * If the destination MAC is in the table and associated with the *same port* the frame arrived on, the frame is filtered (discarded).
            * If the destination MAC is in the table and associated with a *different port*, the frame is selectively forwarded to that specific port.
    * Switches offer **connectionless** (no handshake) and **unreliable** (no ACK/retransmission for lost frames) service at the link layer. Reliability is up to higher layers like TCP.

---
## Ethernet: The Dominant LAN Technology

Ethernet has evolved significantly but remains the most prevalent wired LAN technology.
* **Topology:** Modern Ethernet exclusively uses star topology with switches (or historically, hubs). Common standards include 10BaseT (10 Mbps), 100BaseT (Fast Ethernet), 1000BaseT (Gigabit Ethernet) over twisted-pair cables.
* **Ethernet Frame Structure:**
    * **Preamble (7 bytes):** Alternating 1s and 0s for synchronization.
    * **Start Frame Delimiter (SFD) (1 byte):** Sequence `10101011`, signals the start of the frame.
    * **Destination MAC Address (6 bytes):** Physical address of the recipient.
    * **Source MAC Address (6 bytes):** Physical address of the sender.
    * **Type (or Length for older variants) (2 bytes):** Identifies the upper-layer protocol encapsulated (e.g., IP, ARP).
    * **Data (Payload) (46-1500 bytes):** The encapsulated datagram (e.g., IP packet). Minimum ensures frame is long enough for CSMA/CD collision detection on older, larger bus networks.
    * **CRC (4 bytes):** Cyclic Redundancy Check for error detection.
    * **Inter Frame Gap (equivalent of 96 bits or 12 bytes):** Minimum quiet time between frames.
* **Ethernet (MAC) Address:**
    * A 48-bit (6-byte) unique address burned into the Network Interface Card (NIC) by the manufacturer.
    * Represented as 12 hexadecimal digits (e.g., C4-85-08-30-33-48).
    * Manufacturers are allocated unique prefixes (e.g., Intel: C4-85-08).
    * An adapter passes up frames if they are addressed to its unicast MAC address or the broadcast MAC address (all 1s: FF-FF-FF-FF-FF-FF).

---
## Virtual LANs (VLANs): Segmenting the Network Logically

Traditionally, a station is part of a LAN if it's physically connected to it. But what if you need to group users logically, irrespective of their physical location within a building connected by switches?
* A **VLAN** is a local area network configured by software, not by physical wiring.
* Switches that support VLANs can be configured to define multiple virtual LANs on a single physical infrastructure.
* Hosts in one VLAN are isolated from hosts in another VLAN at Layer 2, as if they were on physically separate LANs, even if they are connected to the same switch.
* Traffic between VLANs typically needs to be routed by a Layer 3 device (router or Layer 3 switch).
* Switches can connect multiple physical LANs and then use VLAN software to create logical groupings. Multiple switches can also form a backbone, with VLANs spanning across them. This requires a mechanism (like IEEE 802.1Q tagging) to identify which VLAN a frame belongs to as it travels between switches.

---
## MPLS (Multiprotocol Label Switching) and Data Centre Networking

The provided knowledge base (PDF document) does not contain specific, detailed sections on **Multiprotocol Label Switching (MPLS)** or modern **Data Centre Networking** architectures and protocols.

* **MPLS** is a routing technique in telecommunications networks that directs data from one node to the next based on short path labels rather than long network addresses, avoiding complex lookups in a routing table.
* **Data Centre Networking** involves designing high-bandwidth, low-latency, and resilient networks to connect thousands of servers within a data center, often using technologies like Clos topologies, VXLAN, and advanced Ethernet variants.

These are important topics in modern networking but fall outside the scope of the current source material.

---
## IEEE 802.11 (Wi-Fi) MAC: Wireless Access

Wireless links have unique characteristics compared to wired links:
* **Decreased signal strength (attenuation/path loss):** Signals weaken as they propagate.
* **Interference from other sources:** Wireless frequencies (like 2.4 GHz) are often shared by other devices (phones, microwaves).
* **Multipath propagation:** Radio signals reflect off objects, arriving at the destination via multiple paths at slightly different times, which can cause interference.

**Collision detection is difficult in wireless mediums** because a node cannot easily "hear" other nodes while it is transmitting (its own signal would drown out weaker incoming signals). Therefore, 802.11 uses **CSMA/CA (Collision Avoidance)** rather than CSMA/CD.

### CSMA/CA in 802.11
The goal is to *avoid* collisions:
1.  **Sense before transmitting (CSMA):** If the channel is busy, defer.
2.  **No Collision Detection:** 802.11 does not detect collisions during transmission.

### Problems in Wireless: Hidden and Exposed Terminals
CSMA alone isn't enough in wireless due to:
* **Hidden Terminal Problem:**
    * Node A is transmitting to B. Node C cannot hear A. C senses the channel as idle and starts transmitting to B, causing a collision at B. A and C are "hidden" from each other with respect to B.
    * **Solution: RTS/CTS (Request to Send / Clear to Send):**
        1.  A sends an RTS to B.
        2.  B responds with a CTS. This CTS is heard by C (and other nodes in B's range).
        3.  C now knows someone (B) is about to receive data and will defer its own transmissions, thus avoiding a collision at B.
        4.  A sends data, B sends ACK.

* **Exposed Terminal Problem:**
    * Node S1 is transmitting to R1. Node S2 wants to transmit to R2. S2 hears S1's transmission and (incorrectly) assumes the medium is busy for its own transmission to R2, even though R2 could receive S2's transmission without interference. S2 is "exposed" to S1's transmission.
    * **Solution with RTS/CTS knowledge:** When a node (S2) hears an RTS from a neighbor (S1) but *not* the corresponding CTS (because R1 is out of S2's range), S2 can deduce it might be an exposed terminal and is permitted to transmit to other neighbors (like R2) if it doesn't hear a CTS from R2 in response to its own RTS.

### 802.11 MAC Protocol Flow (Simplified CSMA/CA)
1.  If the station senses the channel idle for a period called **DIFS (Distributed Inter-Frame Space)**, it transmits its frame.
2.  If the channel is sensed busy, the station chooses a random **backoff value** from a **contention window**. It decrements this backoff counter only when the channel is idle. When the counter reaches zero, it transmits.
3.  The receiver, if the frame is received correctly, waits for a short period called **SIFS (Short Inter-Frame Space)** (which is shorter than DIFS) and then sends an ACK.
4.  If the sender doesn't receive an ACK:
    * It assumes a collision or loss occurred.
    * It increases its contention window size (exponential backoff) to reduce the probability of another collision and repeats the backoff process.
5.  **NAV (Network Allocation Vector):** When RTS/CTS is used, the RTS and CTS frames contain a duration field. Other stations hearing either RTS or CTS set their NAV, which indicates how long the medium will be busy. They defer access until their NAV timer expires. This is a form of "virtual carrier sensing."

---
## Key Takeaways

The Link Layer is the unsung hero ensuring our data makes its crucial first and last hops reliably and efficiently within a local network segment.
* It provides **error detection and correction** capabilities using techniques like parity, checksums, CRC, and Hamming codes.
* **ARP** resolves network layer (IP) addresses to link layer (MAC) addresses, crucial for local delivery.
* **Multiple Access Protocols** (like ALOHA, CSMA/CD in Ethernet, and CSMA/CA in Wi-Fi) manage how shared communication channels are used.
* **Ethernet** is the dominant wired LAN technology, evolving from bus to star topologies using **switches** that learn MAC addresses to intelligently forward frames.
* **VLANs** allow for logical network segmentation on a physical infrastructure.
* **802.11 (Wi-Fi) MAC** uses CSMA/CA and mechanisms like RTS/CTS to deal with the challenges of wireless communication, including the hidden terminal problem.

Understanding the link layer shows how sophisticated even "local" communication needs to be to form the building blocks of larger networks like the internet!
