---
title: The Holy Trinity of System Design - High Availability, Throughput & Scalability Explained
description: Building systems that are always on, super fast, and can handle millions of users? We dive into High Availability (HA), High Throughput, and High Scalability – what they mean, how they relate, and key strategies to achieve them.
date: 2024-08-04
draft: true
slug: /pensieve/highs
tags:
  - system-design
  - SDE
---

Hey everyone, and welcome back to the blog! When we talk about truly great software systems, it's not just about cool features. It's about how well they perform under pressure, how reliably they stay online, and how gracefully they can grow. These crucial non-functional requirements are often summed up by three interconnected pillars: **High Availability**, **High Throughput**, and **High Scalability**.

Understanding these "ilities" is fundamental for any system designer or architect. They aren't just buzzwords; they represent core engineering principles that dictate how robust, responsive, and future-proof an application will be. Today, let's break down each of these pillars, look at how they are measured, the principles that guide their design, and common architectural structures used to achieve them.

## I. High Availability (HA): Ensuring Your Service is Always On

**High Availability** is all about ensuring your application remains operational and accessible to users, even in the event of hardware failures, software bugs, network outages, or other unexpected issues. The goal is to minimize or eliminate downtime.

### Measurements for High Availability:
* **The "Nines" of Availability:** This is a common way to express uptime percentage:
    * **99.9% (Three Nines):** Allows for about 8.76 hours of downtime per year.
    * **99.99% (Four Nines):** Allows for about 52.56 minutes of downtime per year.
    * **99.999% (Five Nines):** Allows for just about 5.26 minutes of downtime per year. Achieving higher "nines" becomes exponentially more complex and expensive.
* **RTO (Recovery Time Objective):** As discussed in the context of Disaster Recovery, RTO is the **maximum acceptable downtime** after a failure occurs. For HA systems, the RTO is typically very low, often aiming for seconds or minutes, or even zero in active-active setups.
* **RPO (Recovery Point Objective):** Also from DR, RPO is the **maximum acceptable amount of data loss** measured in time. For HA systems aiming for no data loss during failover, RPO is near zero.

### Guiding Principles for High Availability:
To achieve HA, several core principles must be followed, effectively aiming to have **no single point of failure (SPOF)** by implementing robust redundancy:
* **Eliminate Single Points of Failure (SPOF):** Identify any component in your system (hardware, software, network link) whose failure would cause the entire system or a critical part of it to become unavailable. Then, design to remove or mitigate these SPOFs.
* **Redundancy:** This is key to eliminating SPOFs. Implement redundant components at every critical layer – multiple servers, multiple database instances, redundant network paths, multiple power supplies, etc. If one component fails, a redundant one can take over.
* **Reliable Failover:** Implement automated mechanisms to quickly and seamlessly switch from a failed component or system to a healthy, redundant one. This failover should ideally be transparent to users.
* **Fault Detection:** You can't failover if you don't know something has failed! Implement robust monitoring and health checks to rapidly detect failures in components or services.

### Common Architectural Structures for HA:
These structures often overlap with Disaster Recovery strategies, especially those aiming for very low RTO/RPO.
* **Active-Active (Hot-Hot):**
    * Multiple instances of the application or service run simultaneously in different locations (e.g., multiple servers, availability zones, or even regions), and all actively serve live user traffic.
    * A load balancer typically distributes requests across these active instances.
    * If one instance or location fails, the remaining active instances can absorb its load. This is a common pattern for achieving very high availability and can also improve performance by serving users from the closest active instance. (This aligns with the "Multi-site Active-Active" DR strategy).
* **Active-Passive (Hot-Warm / Hot-Cold):**
    * A primary instance or set of instances handles all the traffic, while a secondary (standby) instance or set of instances is kept ready to take over in case the primary fails.
    * **Hot Standby (Active-Passive with fast failover):** The standby system is fully operational, data is actively replicated (often synchronously or near-synchronously), and failover can happen very quickly (similar to "Warm Standby" in DR).
    * **Warm Standby (Pilot Light):** The standby system has core infrastructure running but might need to be scaled up or have services fully started during a failover.
    * **Cold Standby (Backup and Restore):** The standby infrastructure might need to be provisioned from scratch and data restored from backups. This offers less "availability" and more "recoverability."
* **Leader-Follower (Single Leader):**
    * Common in distributed databases (e.g., primary-replica setups) and systems using consensus algorithms like Raft or Paxos.
    * One node (the leader or primary) handles write operations and coordinates state. Other nodes (followers or replicas) replicate data from the leader and can often serve read requests.
    * If the leader fails, a failover process elects a new leader from among the followers.
* **Leaderless (Decentralized):**
    * In some distributed systems (e.g., certain NoSQL databases like Cassandra in specific configurations), all nodes can accept read and write operations.
    * Data is replicated across multiple nodes. Requires mechanisms for resolving write conflicts (e.g., last-write-wins, vector clocks).
    * Can offer high availability as the failure of individual nodes doesn't stop the system from processing requests, as long as a quorum of nodes is available.

## II. High Throughput: Handling a Flood of Requests Efficiently
**High Throughput** refers to the ability of your application to process a large volume of requests or data efficiently within a given period. It's about how much "work" your system can get done.

### Measurements for High Throughput:
* **QPS (Queries Per Second):** Often used for read-heavy systems, indicating the number of read requests or queries the system can handle per second.
* **TPS (Transactions Per Second):** Typically used for systems involving write operations or more complex business transactions, indicating the number of such transactions completed per second.

### Guiding Principles for High Throughput:
Several principles and techniques contribute to achieving high throughput:
* **Caching:** Storing frequently accessed data in faster memory layers (client-side, CDN, in-memory caches like Redis/Memcached, database caches) drastically reduces the need to fetch data from slower persistent storage or recompute results. This frees up backend resources to handle more requests.
* **Identify and Eliminate Bottlenecks:** Continuously profile your system (CPU, memory, I/O, network, database queries, application code) to find the slowest parts or resource contention points and optimize them.
* **More Threads & Instances (Concurrency & Parallelism):**
    * **Multi-threading:** Within a single application instance, use multiple threads to handle multiple requests or perform tasks concurrently.
    * **More Instances (Horizontal Scaling):** Add more instances of your application servers and use a load balancer to distribute traffic, allowing more requests to be processed in parallel across the cluster.
* **Spike Management (Load Leveling):** Sudden bursts of traffic can overwhelm a system. Use techniques like:
    * **Message Queues (e.g., Kafka, RabbitMQ):** Place incoming requests or tasks into a queue. Backend workers can then process these items from the queue at a sustainable pace, smoothing out spikes.
    * **Throttling/Rate Limiting:** Limit the number of requests a client can make in a given period to prevent abuse and protect system resources.
* **Database Optimization:** Optimize database schemas, queries, and indexes. Use connection pooling.
* **Asynchronous Processing:** For non-critical or long-running tasks, process them asynchronously in the background.

### Common Architectural Structures for High Throughput:
* **Aggressive Caching Layers:** Implementing caches at multiple levels – browser, CDN, load balancer, distributed cache (like Redis or Memcached), application-level cache, and database cache.
* **Asynchronous Processing with Message Queues:** Decoupling frontend request handlers from backend processing. For example, an order placement request might be quickly acknowledged to the user after being placed on a Kafka queue, with actual order fulfillment happening asynchronously by worker services.

## III. High Scalability: Growing Gracefully with Demand
**High Scalability** is the system's ability to efficiently handle an increasing amount of work or users by adding resources, or its ability to be easily modified to accommodate more functionalities, all without a significant drop in performance (like response time).

### Measurements for High Scalability:
* **Response Time (RT) under Increasing Load:** A key indicator. As the number of users or requests increases, how does the application's response time change? A scalable system should maintain acceptable response times even as load grows.
* **Throughput vs. Resources:** How does throughput (QPS/TPS) increase as more resources (servers, CPU, memory) are added? Ideally, it should scale linearly or close to it.
* **Cost Efficiency of Scaling:** How much does it cost to add capacity to handle more load?

### Guiding Principles for High Scalability:
Many principles overlap with those for high throughput and availability, as these "ilities" are interconnected. Key principles include:
* **Segregation of Service Responsibilities (Decoupling):** Break down the application into smaller, independent, and well-defined services that can be scaled independently.
* **Load Balancing:** Essential for distributing traffic across scaled-out instances.
* **Capacity Planning:** Proactively estimating future resource needs based on growth trends and planning for scaling in advance.
* **Statelessness:** Designing application components (especially services that will be horizontally scaled) to be stateless is crucial. This means they don't store any client-specific session data locally. Any required state is externalized to a shared store (like a distributed cache or database).
* **Horizontal Scaling (Scaling Out) over Vertical Scaling:** While vertical scaling (adding more power to existing servers) has its place, horizontal scaling (adding more servers) is generally preferred for cloud-native and large-scale systems due to better elasticity, fault tolerance, and potentially lower cost at high scale.
* **Database Scalability:** Employing techniques like read replicas, caching, and database sharding (horizontal partitioning) to handle growing data volumes and query loads.
* **Asynchronous Processing:** Decoupling components with message queues allows different parts of the system to scale independently based on their specific loads.
* **Efficient Resource Utilization:** Design to use resources like CPU, memory, and network efficiently.

### Common Architectural Structures for High Scalability:
* **Microservices Architecture:** This is a prime example. Breaking down a monolithic application into smaller, independent microservices allows each service to be developed, deployed, and, critically, **scaled independently** based on its specific demands.
* **Containerization & Orchestration (e.g., Docker & Kubernetes):** Packaging services in containers and managing them with an orchestrator like Kubernetes makes it much easier to deploy, manage, and automatically scale services horizontally.
* **Auto-Scaling:** Cloud platforms and orchestrators provide auto-scaling capabilities that automatically add or remove compute resources based on real-time traffic, CPU utilization, queue lengths, or other defined metrics. This ensures the system can adapt to fluctuating demand efficiently.
* **Content Delivery Networks (CDNs):** Offload static content delivery to a distributed network of servers, reducing latency for users and load on origin servers, which helps the origin scale better for dynamic requests.

## Key Takeaways

* **High Availability, High Throughput, and High Scalability** are interconnected, critical non-functional requirements for building robust, performant, and future-proof applications.
* **High Availability** focuses on minimizing downtime through redundancy and rapid failover, measured by "nines" and RTO/RPO.
* **High Throughput** is about processing a large volume of requests efficiently, measured by QPS/TPS, and achieved via techniques like caching and asynchronous processing.
* **High Scalability** ensures the system can gracefully handle growth by adding resources without performance degradation, often measured by sustained response time under increasing load, and commonly achieved with microservices and horizontal scaling.
* Achieving these "ilities" requires deliberate architectural choices, a deep understanding of trade-offs, and continuous monitoring and optimization.

Building systems that excel in these areas is a hallmark of great engineering, enabling businesses to serve their users reliably and adapt to ever-increasing demands.
