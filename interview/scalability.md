---
title: Scaling an Application - From One Server to Many
description: The strategies for handling growth — vertical and horizontal scaling, load balancing, caching, database replication and sharding, async processing, microservices, CDNs, auto-scaling — and the rough order you reach for them.
date: 2024-07-24
draft: false
slug: /interview/scalability
tags:
  - System Design
  - Interview Prep
  - Scalability
---

**Scalability** is a system's ability to handle more load by adding resources, without performance falling apart. It is not one technique — it is a toolbox applied in roughly the order the bottlenecks appear. This post is the map: what each strategy does, and where it sits in the progression from a single server to one serving millions. Most items have a dedicated post that goes deep.

## Two directions

- **Vertical (scale up)** — a bigger machine: more CPU, RAM, disk. Like giving one supermarket cashier a faster scanner.
- **Horizontal (scale out)** — more machines sharing the load. Like adding more cashiers.

Vertical is simpler and needs no architecture change, but it hits a hard ceiling, gets expensive non-linearly, keeps a single point of failure, and often needs downtime to upgrade. Horizontal has no ceiling but requires the application to be built for it.

Why it matters: user experience degrades under load, the business needs the service up as it grows, and a system that scales *out* can also scale *in* during quiet hours to save money.

## The strategies

**[Load balancing](/citadel/interview/load-balancing).** A load balancer spreads requests across servers, health-checks them, and lets you add capacity by adding machines. The entry point to horizontal scaling.

**Stateless services.** Horizontal scaling only works if any instance can serve any request, which means no session state on the server — push it to a shared store. Then instances are interchangeable, elastic, and individually disposable. This is [twelve-factor](/citadel/interview/12-factor-app) factor VI.

**[Caching](/citadel/interview/caching).** Keep hot data in a fast layer (Redis, a CDN, the app's memory) so most reads never reach the database. Cuts latency and offloads the database; brings its own [invalidation and failure modes](/citadel/interview/cache-pitfalls).

**Database scaling.** Usually the hardest part.

![Vertical and horizontal database sharding. A single User table with columns UserID, Name, Status, Desc, Photo is split three ways: hash-based horizontal partitioning sends rows 1 and 3 to shard 1 and rows 2 and 4 to shard 2; range-based horizontal partitioning sends rows 1 to 2 to shard 1 and 3 to 4 to shard 2; vertical partitioning splits the columns into a User table (UserID, Name, Status) and a User Extension table (UserID, Desc, Photo).](../images/db-partitioning.png "Horizontal partitioning (sharding) splits rows across servers; vertical partitioning splits columns into separate tables. Source: system-design illustration.")

- **[Read replicas](/citadel/interview/read-replica-pattern)** — writes to a primary, reads spread across copies. For read-heavy loads; watch replication lag.
- **[Sharding](/citadel/interview/data-sharding)** — split rows across servers by a shard key. Scales writes and dataset size; adds routing, cross-shard join, and re-sharding complexity.
- **Vertical partitioning** — split a table's columns, or move a heavy table to its own server, to isolate workloads and cut per-query I/O.

**[Async processing](/citadel/interview/message-queue).** Push work that needs no immediate response — emails, report generation, image processing — onto a queue for background workers. Frees the request path, adds resilience (a failed worker leaves the message for later), and lets workers scale on queue depth.

**[Microservices](/citadel/interview/cloud-native).** Decompose a monolith into independently deployable services, each scaled on its own load curve, with fault isolation between them — at the cost of a distributed system to operate.

**[CDN](/citadel/interview/cdn).** Cache static assets at edge locations near users; requests are served from the nearest edge instead of the origin, cutting latency and offloading origin traffic.

**Auto-scaling.** Cloud platforms and Kubernetes add and remove instances automatically against a metric — CPU, memory, queue length — maintaining performance at peak and cutting cost off-peak.

## The journey

The strategies come into play in a fairly predictable order:

1. **One server** — monolith plus local database. Fine to start.
2. **Split app and database** onto separate servers.
3. **App cluster + load balancer** as traffic grows.
4. **Read replicas** to offload database reads.
5. **Caching layer**, then **sharding** if the database is still the bottleneck.
6. **Break into microservices** when the monolith's coupling slows the team down.

## Scalability is a sequence of bottlenecks

There is no single move that makes a system scalable — there is the current bottleneck, the strategy that clears it, and the next bottleneck it exposes. Add a load balancer and the database becomes the limit; add replicas and writes become the limit; shard and operational complexity becomes the limit. The skill is identifying which bottleneck you actually have before reaching for the technique associated with a much later stage.
