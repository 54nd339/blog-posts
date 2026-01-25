---
title: The Transport Layer - Data's Trusty Charioteer Across the Internet
description: Unpacking the transport layer - its vital role in logical communication, multiplexing/demultiplexing, the speed of UDP, the reliability of TCP (Go-Back-N, Selective Repeat), and the intricacies of congestion control.
date: 2023-06-03
draft: false
slug: /pensieve/computer-networks/transport-layer
tags:
  - Computer Networks
  - CS Basics
---

Hey network adventurers! We've explored the application layer, where our software lives and breathes. But how does data from one application process on your computer find its way to the correct application process on another computer, potentially thousands of miles away? That's where the **Transport Layer** steps in, acting like a trusty charioteer for our data!

This layer provides **logical communication** between application processes running on different hosts. While the network layer (which we'll cover soon!) gets packets from host to host, the transport layer gets the messages to the *correct process* on those hosts. The unit of data at this layer is often called a **segment**.

Let's dive into how it manages this crucial task, the different types of services it offers, and how it ensures our data arrives reliably and efficiently.

---
## Multiplexing and Demultiplexing: Directing Traffic

Imagine a busy mailroom in an office building. Letters (data) arrive for many different people (application processes). The mailroom needs to sort these letters and deliver them to the correct recipients. This is essentially what multiplexing and demultiplexing do.

* **Multiplexing (at the sender):** Gathering data chunks from different application processes on the host, encapsulating each with a transport-layer header (which includes source and destination port numbers), and then passing the segments to the network layer.
* **Demultiplexing (at the receiver):** Examining the header of incoming segments to identify the receiving application process (via the destination port number) and then delivering the segment's data to the correct **socket** for that application.

A **socket** is like a doorway through which data passes from the network to the process and vice versa.

### Connectionless vs. Connection-Oriented Demultiplexing

* **Connectionless Demultiplexing (e.g., UDP):** A UDP socket is typically identified by a 2-tuple: (destination IP address, destination port number). Segments with the same destination IP and port are directed to the same process via the same socket.
* **Connection-Oriented Demultiplexing (e.g., TCP):** A TCP socket is identified by a 4-tuple: (source IP address, source port number, destination IP address, destination port number). This means segments with different source IPs or ports can be directed to different sockets, even if they are intended for the same destination port on the server. This allows a server to communicate with multiple clients simultaneously, with a separate socket for each connection.

---
## Connectionless Transport: User Datagram Protocol (UDP)
UDP is the "bare-bones" transport protocol. It takes messages from the application layer, adds source and destination port numbers and a couple of other small fields, and hands it off to the network layer. That's pretty much it!

### Why Use UDP?

* **No Connection Establishment:** UDP doesn't have a "handshake" process to set up a connection, which saves time. This makes it good for applications like DNS where quick, short queries and responses are needed.
* **Simple:** No connection state is maintained at the sender or receiver. This means a server can often support more active clients than if it were using TCP.
* **Small Segment Header:** UDP headers are small (8 bytes), meaning less overhead per packet.
* **No Congestion Control:** UDP doesn't inherently slow down its transmission rate if the network is congested. It blasts away as fast as the application and underlying network allow. This can be good for some applications but can also contribute to network congestion.
* **No Retransmission Delays:** If a packet is lost, UDP doesn't automatically try to retransmit it. This is useful for real-time applications like Voice over IP (VoIP) or online gaming, where receiving a late packet is often worse than not receiving it at all.

If an application using UDP needs reliability, flow control, or congestion control, it has to implement these features itself.

### UDP Segment Format

The UDP segment has a simple structure:
* **Source Port (16 bits):** Identifies the sending process's port.
* **Destination Port (16 bits):** Identifies the receiving process's port.
* **Length (16 bits):** The length of the UDP segment (header + data) in bytes.
* **Checksum (16 bits):** Used for error detection. It's calculated over the UDP header, UDP data, and a "pseudo-header" (parts of the IP header like source/destination IP and protocol number). The checksum is optional in IPv4 but mandatory in IPv6.

Why have a checksum if the link layer below it might also do error checking?
1.  Not all links guarantee error checking.
2.  Bit errors can occur when a segment is stored in a router's memory, even if it was transmitted correctly over a link.

---
## Principles of Reliable Data Transfer (RDT) ‍
For many applications (like file transfer or email), we need data to arrive correctly and in order. This is **Reliable Data Transfer (RDT)**. The network layer (IP) offers a "best-effort" service, meaning packets can be lost, corrupted, or arrive out of order. The transport layer has to work hard to provide reliability on top of this.

RDT involves two main concepts:

* **Flow Control:** Procedures to restrict the amount of data a sender can send before waiting for acknowledgment, preventing the sender from overwhelming the receiver.
* **Error Control:** Mechanisms to detect and recover from errors, such as corrupted or lost packets, often through retransmissions (Automatic Repeat reQuest - ARQ).

Let's look at the evolution of RDT protocols:

### 1. Stop-and-Wait ARQ
This is the simplest RDT protocol that handles errors.
* The sender sends one segment and then *waits* for an acknowledgment (ACK) from the receiver before sending the next.
* **Error Handling:**
    * **Corrupted Segments:** Checksums are used to detect corruption. If a segment is corrupted, the receiver might discard it, or send a negative acknowledgment (NAK).
    * **Lost Segments/ACKs:** The sender uses a **timer**. If an ACK isn't received before the timer expires, the sender retransmits the segment.
    * **Duplicate Segments:** To handle cases where an ACK is delayed (not lost) and the sender retransmits, **sequence numbers** are used. The receiver can then identify if it's seeing a duplicate of a segment it has already processed. For Stop-and-Wait, a 1-bit sequence number (0 and 1) is sufficient.

**Efficiency of Stop-and-Wait:**
The efficiency can be quite low, especially on links with high bandwidth or long propagation delays (like satellite links). The sender spends a lot of time waiting. The link utilization $U$ is given by $1 / (1 + 2a)$, where $a = \text{Propagation Time / Transmission Time}$. If 'a' is large, utilization is poor.

### Pipelining: Improving Efficiency
To overcome the inefficiency of Stop-and-Wait, protocols allow the sender to send multiple segments ("in-flight" segments) before needing an ACK. This is called **pipelining**. Two main pipelined protocols are Go-Back-N and Selective Repeat.

---
## Go-Back-N (GBN) ARQ ‍
In GBN, the sender can transmit multiple packets (up to a certain **window size**, $N$) without waiting for an acknowledgment.
* **Sequence Numbers:** Segments are still numbered.
* **Sender's Window:** The sender maintains a window of up to $N$ consecutive sequence numbers corresponding to segments it's allowed to send.
* **Receiver's Window:** The receiver maintains an expected sequence number. It only accepts segments that arrive in order. If segment $k$ is received and it's the expected one, it's delivered to the application layer, and an ACK for $k$ is sent. If segment $k+1$ arrives before $k$, $k+1$ is discarded (even if correct).
* **Cumulative ACKs:** The receiver sends an ACK for the highest sequence number received *in order*. For example, if segments 0, 1, 2 are received, ACK2 is sent. If segment 0 arrives, then segment 2 arrives (segment 1 is lost), the receiver ACKs 0 (ACK0). When segment 1 eventually arrives, it ACKs 1, and then it can ACK 2.
* **Timeout & Retransmission:** The sender uses a single timer for the oldest unacknowledged packet. If this timer expires, the sender **goes back** and retransmits all unacknowledged packets in its current window, starting from the timed-out packet.

**Send Window Size ($W_s$) for GBN:** If $m$ bits are used for sequence numbers (so $2^m$ possible sequence numbers), the send window size $W_s$ must be less than $2^m$. A common size is $2^m - 1$. The receive window size is always 1.

**Problem with GBN:** If a single packet is lost, or if an early packet in the window is significantly delayed, GBN can end up retransmitting many packets that were actually received correctly by the receiver but discarded because they were out of order. This can be wasteful, especially on links with high error rates or long delays.

---
## Selective Repeat (SR) ARQ
Selective Repeat aims to be more efficient than GBN by only retransmitting segments that are actually lost or corrupted.
* **Sender's Window:** Sender can transmit multiple packets up to window size $N$. Each packet in the window has its own logical timer.
* **Receiver's Window:** The receiver also maintains a window of size $N$. It can receive and buffer out-of-order packets as long as they fall within its current window.
* **Individual ACKs:** The receiver acknowledges each correctly received packet individually, whether it's in order or not.
* **Retransmission:** The sender retransmits only those packets for which an ACK was not received before their individual timer expired.
* **Buffering & Delivery:** The receiver buffers correctly received but out-of-order packets. Once a contiguous block of packets starting from the current expected sequence number is available, they are delivered to the application layer, and the receive window slides forward.

**Send/Receive Window Size ($W_s$, $W_r$) for SR:** For SR, the send window size and receive window size are typically the same, and they must be less than or equal to half the sequence number space ($W_s = W_r \le 2^{m-1}$). This is crucial to avoid ambiguity between new packets and retransmitted old packets from a previous window cycle.

SR is more complex to implement than GBN (e.g., individual timers, buffering at the receiver), but it's generally more efficient on noisy links because it avoids unnecessary retransmissions.

---
## Connection-Oriented Transport: Transmission Control Protocol (TCP)
TCP is the workhorse of the internet for applications that require **reliable, in-order delivery** of a stream of bytes. It's a **connection-oriented** protocol.

### TCP Services:
* **Process-to-Process Communication:** Using port numbers.
* **Stream Delivery Service:** Treats data as a continuous stream of bytes, not individual messages.
* **Full-Duplex Communication:** Data can flow in both directions simultaneously over a single connection.
* **Multiplexing and Demultiplexing:** As discussed earlier.
* **Connection-Oriented Service:** Requires a three-way handshake to establish a connection before data transfer, and a process to terminate it.
* **Reliable Service:** Uses sequence numbers, acknowledgments, and retransmissions (a hybrid of GBN and SR concepts) to ensure data arrives correctly and in order.

### TCP Segment Format

The TCP header is more complex than UDP's:
* **Source Port (16 bits)** & **Destination Port (16 bits)**
* **Sequence Number (32 bits):** The sequence number of the *first data byte* in this segment. If the SYN flag is set, this is the Initial Sequence Number (ISN), and the first data byte is ISN+1.
* **Acknowledgment Number (32 bits):** If the ACK flag is set, this field contains the sequence number of the *next byte* the sender of the ACK is expecting from the other side. TCP uses cumulative acknowledgments.
* **Header Length (4 bits):** Length of the TCP header in 32-bit words.
* **Reserved (6 bits)**
* **Flags (6 bits):**
    * `URG`: Urgent pointer field is significant.
    * `ACK`: Acknowledgment field is significant.
    * `PSH`: Push function (receiver should pass data to upper layer promptly).
    * `RST`: Reset the connection.
    * `SYN`: Synchronize sequence numbers (used in connection setup).
    * `FIN`: No more data from sender (used in connection termination).
* **Receive Window (16 bits):** Used for flow control. The number of bytes the sender of this segment is willing to accept.
* **Checksum (16 bits):** Similar to UDP, calculated over header, data, and IP pseudo-header. It's compulsory.
* **Urgent Pointer (16 bits):** Points to the sequence number of the byte following urgent data.
* **Options (variable length):** Can be used for things like negotiating Maximum Segment Size (MSS), window scaling, or selective acknowledgments (SACK).

**Initial Sequence Number (ISN):** TCP connections don't start with sequence number 0. This is to avoid confusion with segments from previous, defunct connections that might still be lingering in the network. ISNs are typically chosen randomly or based on a clock.

### TCP Connection Management

* **Connection Establishment (Three-Way Handshake):**
    1.  **Client to Server:** Sends a TCP segment with `SYN=1`, `Seq=client_isn` (client's initial sequence number).
    2.  **Server to Client:** If server accepts, it sends a TCP segment with `SYN=1`, `ACK=1`, `Seq=server_isn`, `AckNum=client_isn+1`.
    3.  **Client to Server:** Sends a TCP segment with `ACK=1`, `AckNum=server_isn+1`.
    At this point, the connection is established.

* **Data Transfer:** Segments are exchanged, with sequence numbers tracking the byte stream and acknowledgment numbers confirming receipt.

* **Connection Termination (Four-Way Handshake with Half-Close):**
    One side can decide to stop sending data (initiating a "half-close").
    1.  **Client to Server (e.g.):** `FIN=1`, `Seq=k`
    2.  **Server to Client:** `ACK=1`, `AckNum=k+1` (Server acknowledges client's FIN. Server can still send data if it wants).
    3.  **Server to Client (later):** When server is also done sending: `FIN=1`, `Seq=p`
    4.  **Client to Server:** `ACK=1`, `AckNum=p+1`
    The client then enters a `TIME_WAIT` state for a period (typically 30 seconds to 2 minutes) to ensure any lingering packets from this connection die out before a new connection with the same socket pair might be established.

### TCP Flow Control
TCP uses a **receive window** (advertised by the receiver in its segments) to tell the sender how much buffer space it has available. The sender ensures that the amount of unacknowledged data it sends does not exceed this advertised window. This prevents the sender from overflowing the receiver's buffer.

### TCP Error Control
TCP uses a combination of techniques:
* **Checksum:** For detecting corrupted segments.
* **Acknowledgments (Cumulative):** To confirm receipt.
* **Retransmission Time-Out (RTO):** TCP maintains an RTO for the connection. If an ACK for a segment isn't received before the RTO expires, the segment is retransmitted. The RTO is dynamically adjusted based on observed Round-Trip Times (RTTs). Typically, when a timeout occurs, the RTO is doubled for the next retransmission (exponential backoff).
* **Fast Retransmission:** If a sender receives three duplicate ACKs for the same segment, it's a strong indication that the segment *after* the acknowledged one was lost. Instead of waiting for the RTO, TCP performs a "fast retransmission" of the suspected lost segment. This significantly improves performance in the face of occasional packet loss.
* **Selective Acknowledgment (SACK - an option):** While base TCP uses cumulative ACKs, SACK allows the receiver to acknowledge out-of-order blocks of data it has received correctly. This helps the sender to be more precise about which segments need retransmission.

---
## Principles of Congestion Control
Imagine too many cars trying to use a highway at once – you get a traffic jam! Network **congestion** happens when routers and links become overwhelmed with too much data. This leads to:
* Packets being dropped (routers run out of buffer space).
* Long delays.
* Potentially, "congestion collapse," where very little useful data gets through.

**Congestion control** aims to prevent this by having senders regulate the rate at which they inject data into the network based on perceived network conditions.

The sender maintains a **congestion window (`cwnd`)**, which is a limit on how much unacknowledged data it can have in transit. The actual amount of data a TCP sender can send is the minimum of the `cwnd` and the receiver's advertised **receive window (`rwnd`)**.
Effective Send Window = $\text{min}(\text{cwnd}, \text{rwnd})$

The general idea is:
* If the network seems clear, increase `cwnd`.
* If congestion is detected (e.g., via packet loss indicated by timeouts or duplicate ACKs), decrease `cwnd`.

---
## TCP Congestion Control: A Dynamic Dance

TCP uses a sophisticated set of algorithms to manage its `cwnd`. The two main phases are:

### 1. Slow Start (Exponential Increase)
* When a connection begins, `cwnd` is typically initialized to a small value (e.g., 1 MSS - Maximum Segment Size).
* For every ACK received, `cwnd` is increased by 1 MSS. This results in the `cwnd` effectively doubling every RTT.
* This phase continues until `cwnd` reaches a **slow start threshold (`ssthresh`)** or until loss is detected.
* The "slow" start isn't actually slow; it's an exponential growth phase to quickly probe for available bandwidth.

### 2. Congestion Avoidance (Additive Increase)
* Once `cwnd` reaches `ssthresh`, TCP enters congestion avoidance.
* In this phase, `cwnd` is increased more conservatively: by approximately 1 MSS per RTT (e.g., for each ACK received, `cwnd = cwnd + MSS * (MSS/cwnd)`). This is an additive increase.
* The goal is to gently probe for more bandwidth without quickly causing congestion.

### Reacting to Congestion (Packet Loss)

How TCP reacts to a loss event (timeout or triple duplicate ACKs) determines its "flavor" (e.g., Tahoe, Reno, NewReno, CUBIC).

* **On a Timeout Event (indicating more severe congestion):**
    * `ssthresh` is set to $\text{max}(\text{cwnd}/2, 2 \times \text{MSS})$.
    * `cwnd` is reset to 1 MSS.
    * The sender re-enters the Slow Start phase.
    (This is common to most TCP versions).

* **On Receiving Three Duplicate ACKs (indicating less severe, single packet loss):**
    * **TCP Tahoe:** Reacts the same way as a timeout: sets `ssthresh = cwnd/2`, `cwnd = 1 MSS`, and re-enters Slow Start.
    * **TCP Reno (introduced Fast Recovery):**
        * `ssthresh` is set to $\text{cwnd}/2$.
        * `cwnd` is set to `ssthresh + 3 MSS` (to account for the 3 packets that have left the network and been ACKed by the duplicate ACKs). This is called "inflating" the window.
        * It retransmits the lost segment (fast retransmission).
        * Enters a **Fast Recovery** state: For each additional duplicate ACK that arrives, `cwnd` is incremented by 1 MSS (further inflating the window as more packets clear the network).
        * When an ACK arrives that acknowledges new data (the retransmitted segment), TCP "deflates" `cwnd` back to `ssthresh` and transitions to the Congestion Avoidance phase. Reno avoids going all the way back to Slow Start for a single lost packet detected by triple duplicate ACKs.

**Example Scenario (Tahoe):**
1.  Start with `ssthresh` = 16 MSS, `cwnd` = 1 MSS (Slow Start).
2.  `cwnd` grows exponentially: 1 -> 2 -> 4 -> 8 MSS.
3.  Timeout occurs when `cwnd` is 8 MSS.
4.  New `ssthresh` = 8/2 = 4 MSS. `cwnd` = 1 MSS. Re-enter Slow Start.
5.  `cwnd` grows exponentially: 1 -> 2 -> 4 MSS.
6.  At `cwnd` = 4 MSS (equals `ssthresh`), switch to Congestion Avoidance.
7.  `cwnd` grows additively: 4 -> 5 -> 6 ... -> 12 MSS.
8.  Three duplicate ACKs arrive when `cwnd` is 12 MSS.
9.  New `ssthresh` = 12/2 = 6 MSS. `cwnd` = 1 MSS. Re-enter Slow Start.

**Example Scenario (Reno - difference from Tahoe occurs at step 8):**
...
8.  Three duplicate ACKs arrive when `cwnd` is 12 MSS.
9.  New `ssthresh` = 12/2 = 6 MSS. `cwnd` = `ssthresh + 3 MSS` = 6 + 3 = 9 MSS.
10. Enter Fast Recovery. If more duplicate ACKs arrive, `cwnd` increases.
11. When a new ACK arrives (for the retransmitted segment), `cwnd` is set to `ssthresh` (6 MSS) and TCP enters Congestion Avoidance.

TCP congestion control is a finely tuned mechanism that tries to be efficient by using as much network capacity as available, fair to other TCP connections, and quick to react to signs of congestion.

---
## Key Takeaways

The Transport Layer is a critical intermediary, providing essential services for applications:
* **Multiplexing/Demultiplexing** ensures data reaches the correct application process using port numbers.
* **UDP** offers fast, connectionless, "no-frills" transport, ideal for speed-sensitive or query-response applications.
* **Reliable Data Transfer (RDT)** principles, implemented through protocols like Stop-and-Wait, Go-Back-N, and Selective Repeat, add error control and sequencing on top of unreliable network layers.
* **TCP** provides reliable, in-order, connection-oriented stream delivery, managing flow control and sophisticated congestion control.
* **Congestion Control** in TCP (Slow Start, Congestion Avoidance, Fast Retransmit/Recovery) is vital for the stability and efficiency of the Internet.

Understanding the transport layer helps us appreciate how applications can communicate so effectively across the complex and sometimes chaotic environment of the internet!
