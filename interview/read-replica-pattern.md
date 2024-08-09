---
title: Read Replica Pattern - Scaling Reads Off the Primary
description: Send writes to one primary database and reads to copies of it. How to route the traffic — in the application or through middleware — and how to handle the replication lag that lets a user miss data they just wrote.
date: 2024-08-09
draft: false
slug: /interview/read-replica-pattern
tags:
  - System Design
  - Interview Prep
  - Databases
---

Most applications read far more than they write. When the database becomes the bottleneck under that read load, the **read replica pattern** is usually the first fix: all writes (`INSERT`, `UPDATE`, `DELETE`) go to a **primary**, all `SELECT`s go to one or more **read replicas** that the primary continuously copies data to, normally asynchronously. The primary is freed to focus on writes, and read capacity scales by adding replicas.

This covers the two ways to route the traffic and the one problem the pattern always brings: replication lag.

![An order service sends database operations over the MySQL network protocol to a database middleware layer. The middleware routes a write (create an order) to the Primary DB and reads (view order details, view order history) to Secondary DB 1 and Secondary DB 2, which the primary replicates to.](../images/read-replica-pattern.png "Middleware routing: the application sends every query to one endpoint, and the middleware directs writes to the primary and reads to replicas.")

## Routing option 1: in the application

The application code decides, per query, whether to hit the primary or a replica. Alice places an order — the order service sends the `INSERT` to the primary. The primary replicates it. Alice views her order history — the service sends that `SELECT` to a replica.

### The problem: replication lag

With asynchronous replication, replicas trail the primary — by milliseconds normally, seconds or minutes under heavy write load, replica load, or network trouble. So: Alice places an order (write to primary), then immediately opens her order list (read from a replica that has not received the new row yet) and her order is missing. This is a failure of **read-after-write consistency**.

Fixes:

- **Route latency-sensitive reads to the primary** when up-to-the-second accuracy matters.
- **Read-your-own-writes from the primary** — for a short window after a user writes, send *that user's* reads to the primary so they always see their own change.
- **Check replica sync status** — many databases expose the replica's position in the primary's log; if it is caught up, use it, otherwise fall back to the primary or wait and retry.

## Routing option 2: middleware

A dedicated layer sits between the application and the database cluster. The application sends every query to the middleware, which routes transparently — `SELECT` to a replica, writes to the primary, with rules that can also key on user, schema, or SQL pattern. It speaks the database's own wire protocol (the MySQL protocol, say), so any compatible client connects to it unchanged.

- **Pros** — the application knows nothing about the topology; routing logic is in one place; migrations and replica changes are hidden from clients.
- **Cons** — the middleware is itself a distributed system to configure, monitor, and make highly available, or it becomes a new single point of failure; and it adds a network hop, so it has to be fast.

## Why reach for it

- **Read scalability** — read load spreads across replicas.
- **Primary relief** — the primary spends its resources on writes.
- **Read availability** — a replica failing leaves other replicas and the primary to serve reads.
- **Simpler than [sharding](/citadel/interview/data-sharding)** — as long as one primary can absorb all the writes and hold the whole dataset, replicas scale reads without the complexity of splitting data.

## It scales reads, and only reads

The pattern's ceiling is the primary: every write still goes through one machine, and every replica must apply every write, so it does nothing for write throughput or total data size. When those become the limit, [sharding](/citadel/interview/data-sharding) is next. And every replica read is a bet that lag is tolerable for that query — the same [latency-versus-consistency](/citadel/interview/latency-consistency) trade, made per read.
