---
title: Stack Overflow's Architecture - A Monolith That Scales Vertically
description: Stack Overflow serves billions of page views a month from a .NET monolith on nine web servers in its own data centre, with no separate cache tier. Why that setup beats the microservices design a system-design interview would expect.
date: 2025-02-26
draft: false
slug: /system-design/stack-overflow
tags:
  - System Design
  - Case Study
  - Architecture
---

Ask a candidate to design Stack Overflow in an interview and you'll hear about microservices, a database per service, layered caches, sharding, and a message queue between it all. The real thing is a single .NET application, running on about nine web servers, in a data centre Stack Overflow owns. It serves roughly two billion page views a month across its ~200 sites, and for years the core Q&A path had no Redis or Memcached in front of the database at all.

This is a case study in the architecture that doesn't interview well and works anyway.

## What the interview expects

The "correct" web-scale answer has a familiar shape:

- **Microservices** — a user service, a question service, an answer service, a search service.
- **A database per service**, so teams deploy independently.
- **Multiple cache layers** (Redis, Memcached) to keep read load off the databases.
- **Sharding** to spread data and writes across machines.
- **A message queue** (Kafka, RabbitMQ) for asynchronous work between services.
- **CQRS and event sourcing**, plus fluency in eventual consistency and the CAP theorem.

It's a reasonable design for an organisation that needs dozens of teams shipping in parallel. It is not the only way to handle the traffic.

## What Stack Overflow actually runs

- **A monolith.** The core Q&A product is one .NET application, not a constellation of services.
- **On-premise.** It runs on Stack Overflow's own servers in its own data centres, not on AWS, Azure, or GCP.
- **Nine web servers.** That IIS fleet handled the full ~2 billion monthly page views across the network at the time these numbers were reported.
- **SQL Server with enormous RAM.** The primary database is Microsoft SQL Server on machines with on the order of 1.5 TB of RAM.
- **No separate cache tier for core data.** With that much memory, SQL Server keeps essentially the whole working set in its buffer pool, so the database *is* the cache. Adding Redis in front of it would buy little.

## Why it holds up

**The workload is overwhelmingly reads.** People browse far more questions than they post. A well-tuned relational database with its working set in RAM answers read-heavy traffic extremely well, and that's most of what Stack Overflow serves.

**Vertical scaling, taken seriously.** Rather than shard across many modest database nodes, Stack Overflow buys very large machines so the active dataset lives in memory and disk I/O nearly drops out of the read path. One big well-fed database is simpler to run than a sharded fleet, and within its ceiling it's faster.

**A co-located monolith has no internal network.** Calls between parts of the application are in-process. A microservice design turns many of those into network round trips, each with its own latency and failure mode.

**A small expert team on a stack they know cold.** The people running it know .NET, SQL Server, and Windows deeply enough to push them hard and keep the queries tuned. Operating one well-understood system can be simpler than operating a distributed one, when the workload allows it.

## The point

Stack Overflow's setup raises a blunt question: is good architecture the one that looks right in the interview, or the one running in production? The honest answer is that it depends on the workload, the team, and the constraints. A read-heavy product with a team that knows its stack can go a very long way on a vertically scaled monolith — [microservices](/citadel/interview/cloud-native) buy independent deployment and fault isolation, at the cost of a distributed system to run, and that trade isn't always worth making. Reach for [the scaling playbook](/citadel/interview/scalability) when a specific bottleneck demands it, not by default.
