---
title: Nanoseconds to Seconds - Latency Numbers Every Software Engineer Should Know
description: Why is accessing RAM so much faster than SSDs or round trips across continents? We dive into the crucial latency numbers that shape system design, from L1 cache hits to database commits and global network hops.
date: 2024-06-19
draft: false
slug: /pensieve/latency-number
tags:
  - thoughts
  - SDE
---

Hey everyone, and welcome back to the blog! As software engineers and system designers, performance is king, we're constantly making decisions that impact how quickly our applications respond. Is it okay to make that extra database call? Should we cache this data in memory or fetch it from disk? How far away can our users be before things feel sluggish?

To make these decisions intelligently, it's incredibly helpful to have a good mental model of typical **latency numbers** – how long common computer operations take. These numbers, famously popularized by engineers like Jeff Dean, act as crucial rules of thumb. They aren't always exact down to the nanosecond, as they are based on various online benchmarks and can change with technology, but they give us a vital sense of scale and help us reason about performance trade-offs.

## Why Do These Numbers Matter? The Impact of a Millisecond (or Nanosecond!)

Understanding these latency numbers is vital for several reasons:

* **Informed Design Choices:** Knowing that reading from RAM is thousands of times faster than reading from a typical spinning disk helps you justify caching strategies.
* **Back-of-the-Envelope Calculations:** During system design interviews or initial planning, these numbers allow you to quickly estimate potential bottlenecks.
* **Performance Debugging:** When an application is slow, having a sense of expected latencies can help you pinpoint where the delays are likely occurring.
* **Appreciating Scale:** They provide a visceral understanding of the vast differences in speed between operations happening on-chip versus across a global network.
* **Business Impact:** As Amazon famously discovered about a decade ago, every 100ms of latency could cost them 1% in sales. Slow performance isn't just annoying; it can directly affect the bottom line.

So, let's climb the latency ladder, from the lightning-fast operations within your CPU to the comparatively glacial pace of cross-continent communication. Note that these are approximate figures based on various online benchmarks (like Jeff Dean's numbers and other sources) and are intended to give a sense of magnitude.

## The Latency Ladder: From Nanoseconds to Seconds

It's important to remember these units:
* 1 nanosecond (ns) = $10^{-9}$ seconds
* 1 microsecond (µs) = $10^{-6}$ seconds = 1,000 ns
* 1 millisecond (ms) = $10^{-3}$ seconds = 1,000 µs = 1,000,000 ns

### 1. L1 Cache Reference: ~1 ns
* **Description:** The Level 1 cache is the fastest memory available, located directly on the CPU chip itself. It stores data the CPU is very likely to need again almost immediately.
* **Relevance:** While incredibly fast, L1 cache is usually quite small. Unless you are working directly with hardware or very low-level performance optimization, application developers typically don't manage L1 cache directly  – it's handled by the CPU.

### 2. L2 Cache Reference: ~10 ns
* **Description:** The Level 2 cache is slightly larger and slower than L1 cache, also typically on the CPU chip or very close to it. It serves as a backup for L1.
* **Relevance:** Similar to L1, direct management by application developers is rare.

### 3. Main Memory (RAM) Access: ~100 ns
* **Description:** This is the time it takes to access data from your computer's main Random Access Memory (RAM).
* **Example:** An in-memory data store like Redis, which keeps its data primarily in RAM, can achieve read latencies in the order of 100 nanoseconds for data already in memory. This is why in-memory caches are so effective for speeding up applications.

### 4. Send 1KB over 1 Gbps Network: ~10 µs (microseconds)
* **Description:** This represents the time it might take to transfer a small 1KB packet of data over a fast local area network (LAN) with 1 Gigabit per second bandwidth.
* **Example:** When a service like Memcached sends 1KB of data to an application server over a local network, the network transmission part might take around 10 microseconds. This doesn't include the application processing time on either end.

### 5. Read from SSD (Solid-State Drive): ~100 µs
* **Description:** Accessing data from a fast Solid-State Drive. SSDs are significantly faster than traditional spinning Hard Disk Drives (HDDs) because they have no moving parts.
* **Example:** A disk-based key-value store like RocksDB running on an SSD might have a read latency of around 100 microseconds for accessing data from the drive.

### 6. Database Insert Operation (e.g., RDBMS): ~1 ms (millisecond)
* **Description:** The time taken for a typical database to commit an insert operation. This is a more complex operation than a simple read from storage.
* **Example:** A commit operation in a relational database like PostgreSQL might take around 1 millisecond. This involves multiple steps: the database needs to store the data, potentially create or update indexes, and flush transaction logs to ensure durability, all of which take time.

### 7. Round Trip Time (RTT) - e.g., Data Center California to Netherlands and back: ~100 ms
* **Description:** This is the time it takes for a data packet to travel a significant geographic distance over the internet (e.g., across continents) and for a response to return. The speed of light is a fundamental physical limit here!
* **Example:** The latency experienced during a long-distance video call, like a Zoom call between California and the Netherlands, might be around 100 milliseconds. This highlights why Content Delivery Networks (CDNs) and geographically distributed services are crucial for global applications.

### 8. Typical Retry/Refresh Interval: ~1-10 s (seconds)
* **Description:** This isn't a hardware latency but rather a common application-level timing. It represents typical intervals used in monitoring systems to refresh data or for applications to retry failed operations.
* **Example:** Monitoring systems like Grafana often have default refresh intervals for dashboards in the range of 5 to 10 seconds.

## Understanding the Scale: Why Nanoseconds Matter

It's easy to see "ns," "µs," and "ms" and gloss over them, but the differences in magnitude are enormous:
* Accessing RAM (~100 ns) is about **1,000 times faster** than reading from an SSD (~100 µs).
* Reading from an SSD (~100 µs) is about **10 times faster** than a typical database commit (~1 ms).
* A database commit (~1 ms) is about **100 times faster** than a cross-continent internet round trip (~100 ms).

These orders of magnitude are what drive many system design decisions:
* Why is caching in RAM so effective? Because it's orders of magnitude faster than disk or network access.
* Why are CDNs important? Because network round trips across large distances add significant user-perceived latency.
* Why are too many synchronous database calls in a single user request bad? Because each one might add milliseconds, quickly accumulating to a noticeable delay.

## Key Takeaways

* Having a feel for these "latency numbers every programmer should know" provides an invaluable intuition for system performance.
* They highlight the vast differences in speed between CPU caches, RAM, SSDs, network operations, and database commits.
* These numbers help justify design choices like caching, data partitioning, asynchronous processing, and the use of CDNs.
* While the exact numbers can vary with specific hardware and conditions, understanding the orders of magnitude is what truly matters for effective system design.

Keeping these ballpark figures in mind can help you build faster, more responsive systems and make more informed decisions when you're deep in the trenches of system design or performance tuning.
