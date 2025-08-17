---
title: High Availability, Throughput, and Scalability - The Three Ilities
description: Three non-functional requirements that get conflated. Availability is staying up, throughput is work done per second, scalability is holding performance as you add resources. How each is measured and the architectures that deliver them.
date: 2025-08-17
draft: false
slug: /system-design/highs
tags:
  - System Design
  - Interview Prep
  - Reliability
---

"Handles scale" usually means three separate things: the system stays up (availability), it processes a lot per second (throughput), and it keeps its response time as load and resources grow (scalability). They're related but not the same, measured differently and achieved with overlapping but distinct techniques.

## High availability

Staying operational through hardware failure, bugs, and network outages.

**Measured** by uptime "nines" and by recovery objectives:

| Availability | Downtime per year |
| --- | --- |
| 99.9% (three nines) | ~8.76 hours |
| 99.99% (four nines) | ~52.56 minutes |
| 99.999% (five nines) | ~5.26 minutes |

Each extra nine is exponentially harder and costlier. **RTO** (recovery time objective) is the maximum acceptable downtime after a failure; **RPO** (recovery point objective) is the maximum acceptable data loss, measured in time. HA systems aim for RTO in seconds and RPO near zero. See [disaster recovery](/citadel/interview/cloud-disaster-recovery) for these in depth.

**Principles:** find every **single point of failure** and remove it with **redundancy** at each layer; add **reliable, automated failover** to a healthy replica; and **detect faults** fast, because you can't fail over to what you don't know is broken.

**Topologies:**
- **Active-active** — every instance serves live traffic behind a load balancer; a failure is absorbed by the rest, and users are served from the nearest instance.
- **Active-passive** — a primary serves everything, a standby waits. Hot standby replicates continuously and fails over in seconds; warm standby has core infra running but needs scaling up; cold standby is provisioned from backups on demand.
- **Leader-follower** — one node takes writes and coordinates (Raft, Paxos); followers replicate and serve reads; a failed leader triggers an election.
- **Leaderless** — any node accepts reads and writes (Cassandra-style), data is replicated to several, and conflicts resolve by rules like last-write-wins or vector clocks; the system runs as long as a quorum is up.

## High throughput

How much work per second the system completes.

**Measured** by **QPS** (queries per second, read-heavy) and **TPS** (transactions per second, write or business operations).

**Principles:** [cache](/citadel/interview/caching) frequently read data so backends are freed for more requests; profile to find and fix the current bottleneck; add concurrency (more threads per instance, more instances behind a load balancer); level spikes with a [message queue](/citadel/interview/message-queue) so workers process at a steady pace, plus throttling and rate limiting; optimize database schema, queries, indexes, and connection pooling; move non-critical work to async background processing.

## High scalability

Keeping performance steady while adding resources to meet growth.

**Measured** by response time as load rises (should stay acceptable), throughput gain per added resource (ideally near-linear), and the cost of that added capacity.

**Principles:** decouple into independently scalable services; load-balance across instances; plan capacity ahead of the growth curve; make services **stateless** so any instance handles any request and sessions live in a shared store; prefer **horizontal** scaling over vertical for elasticity and fault tolerance; scale the database with [replicas and sharding](/citadel/interview/data-sharding); decouple with queues so components scale on their own load.

**Structures:** [microservices](/citadel/interview/cloud-native) so each service scales to its own demand; containers and [Kubernetes](/citadel/interview/orchestration) to make horizontal scaling routine; **auto-scaling** against CPU, queue depth, or traffic; and a [CDN](/citadel/interview/cdn) to offload static delivery so the origin scales for dynamic work.

## Key takeaways

- **Availability** is minimizing downtime through redundancy and fast failover — measured in nines and RTO/RPO.
- **Throughput** is work completed per second — measured in QPS/TPS, raised by caching, concurrency, and async processing.
- **Scalability** is holding response time as load and resources grow — measured by sustained latency under load, achieved with statelessness, horizontal scaling, and microservices.
- They interlock: a queue that levels a throughput spike also decouples services for scaling, and redundancy for availability is the same replica set that scales reads. See [the scaling playbook](/citadel/interview/scalability) for the progression of moves.
