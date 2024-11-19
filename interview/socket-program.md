---
title: Sockets - The Endpoint Under Every Network Protocol
description: A socket is a programmable endpoint for network communication, sitting between an application and the transport layer. The conceptual data journey through encapsulation and back, and how TCP and UDP sockets differ.
date: 2024-11-19
draft: false
slug: /interview/socket-program
tags:
  - System Design
  - Interview Prep
  - Networking
---

Every chat message, web page fetch, and inter-service call bottoms out at the same primitive: a **socket** — a programmable endpoint for sending and receiving data across a network. Higher-level protocols like [HTTP](/citadel/interview/http) and WebSockets are built on top of it. This is a conceptual tour of what a socket is and how data actually travels through one, without the specific C API.

## Where sockets sit

Two layers matter:

- **Transport (Layer 4)** — TCP and UDP live here. TCP is reliable and connection-oriented (a phone call); UDP is fast and connectionless (a postcard). Both use **port numbers** to identify which application on a host the data is for.
- **Network (Layer 3)** — IP handles addressing and routing packets across networks by IP address.

A socket is the interface between your application and the transport layer — a standard way to use TCP/IP without implementing it.

## What a socket is

An **endpoint for communication**, like a power socket in a wall is an endpoint for electricity. Two applications that want to talk each create a socket; the server **listens** on its socket, the client **connects** to the server's. The socket abstracts the protocol machinery into "open this, write bytes, read bytes, close it".

## The data journey

A chat app sends a message:

1. **App to kernel.** The application (user space) hands the message to the kernel via the socket interface. It lands in a kernel send buffer for that socket.
2. **Encapsulation.** The kernel's network stack wraps it, header by header:
   - **Transport** — TCP splits the data into segments and adds a header with source/destination ports, sequence numbers (for ordered, reliable delivery), and control flags. UDP adds a smaller header: ports and length.
   - **Network** — IP wraps each segment in a packet with source and destination IP addresses and routing fields.
   - **Data link** — each packet becomes a frame with MAC addresses for the local network hop.
3. **Onto the wire.** The NIC transmits the frames as electrical or optical signals.
4. **Across the network.** Routers and switches forward the frames to the destination host.
5. **De-encapsulation.** The receiving NIC buffers incoming frames and interrupts the CPU. The kernel strips each layer's header, verifies checksums, and — for TCP — reassembles segments in order and sends acknowledgements.
6. **To the app.** The reconstructed message is placed in the socket's receive buffer and handed to the receiving application in user space.

## TCP versus UDP sockets

- **TCP (stream sockets)** — a connection is established first with a three-way handshake, giving a reliable, ordered, byte-stream channel. What HTTP, SSH, and databases use.
- **UDP (datagram sockets)** — no handshake; datagrams are fired at an IP and port with no delivery or ordering guarantee. What DNS, streaming, and games use, where a lost packet is cheaper than a stalled one.

## Why it matters for design

- **Distributed systems** — sockets are how every microservice talks to every other, same host or across the world.
- **Performance** — the TCP-versus-UDP choice, plus connection pooling and non-blocking I/O, are real levers on throughput and latency.
- **Troubleshooting** — knowing the model makes `netstat` and `ss` output legible when a connection hangs.

## Sockets are the floor you rarely stand on

Almost nobody writes to the raw socket API anymore — you use an HTTP client, a database driver, a message-queue library — but all of them are calling `socket()`, `connect()`, `send()`, `recv()` underneath. Knowing that layer is what lets you reason about why a connection pool helps, why a chatty protocol is slow over a high-latency link, and what "the TCP handshake" in the [HTTPS](/citadel/interview/https) sequence is actually doing.
