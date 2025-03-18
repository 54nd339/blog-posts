---
title: Database Sharding - Splitting Rows Across Servers
description: When one database server can no longer hold the data or absorb the writes, sharding spreads rows across many. Vertical versus horizontal partitioning, the range, hash, and directory routing schemes, and the simpler options to exhaust first.
date: 2025-03-18
draft: false
slug: /interview/data-sharding
tags:
  - System Design
  - Interview Prep
  - Databases
---

A single database server has a ceiling: the largest disk you can attach, the write throughput of one primary, the working set that fits in its RAM. **Sharding** breaks through that ceiling by splitting one logical table's rows across many independent servers, each holding a slice of the data and taking a slice of the load.

It also gives up things a single database hands you for free — joins, foreign keys, global ordering. This covers the partitioning idea it rests on, the three schemes for deciding which row goes where, the costs, and the cheaper options worth trying first.

## Partitioning: vertical and horizontal

**Partitioning** divides a large table into smaller independent pieces.

- **Vertical partitioning** splits by *columns*: every row is present in each piece, but the columns are divided — a hot `user_profile` table separated from a rarely read `user_activity_log`, or large BLOB columns split off so common queries scan less.
- **Horizontal partitioning** splits by *rows*: each piece — a **shard** — has the full schema but only a subset of the rows, and the shards live on different servers. This is sharding.

## Why shard

- **Horizontal scale for writes and reads.** Add servers as data and traffic grow, rather than buying a bigger single box and hitting its hard limit. Unlike [read replicas](/citadel/interview/read-replica-pattern), sharding scales the write path too.
- **Smaller working set per shard.** Each query touches one shard's worth of data and index, so it scans less; independent shards can also run queries in parallel.
- **Fault isolation.** One shard's server failing takes out that slice of data, not the whole dataset.
- **Geographic placement.** Shard by region so data sits near its users and inside its jurisdiction (GDPR residency).

## What it costs

- **The application must be shard-aware.** Every query needs the shard key so it can be routed; talking to "the database" becomes talking to the right one of many.
- **Hotspots.** A poorly chosen key or scheme sends disproportionate data or traffic to one shard while others idle, undoing the benefit.
- **Cross-shard joins.** A `JOIN` spanning shards is expensive or unsupported, and usually ends up done in application code by fetching from each shard and combining.
- **No cross-shard referential integrity.** Foreign-key enforcement across shards is impractical; it moves to the application.
- **Re-sharding is painful.** Adding shards or rebalancing means moving large volumes of live data, with real risk of downtime or degraded performance.
- **Global `ORDER BY` on a non-key column** requires gathering from every shard and sorting at an aggregation layer.

## Routing schemes

The **shard key** is the column (or columns) whose value decides the shard; the **algorithm** maps that value to a specific shard.

**Range-based.** Contiguous key ranges map to shards — user IDs 1–1,000,000 on shard 1, 1,000,001–2,000,000 on shard 2. Simple, and range queries on the key hit one shard. But uneven data skews distribution, and monotonically increasing IDs make the newest shard a write hotspot while older shards go cold.

**Hash-based.** A hash of the key picks the shard, commonly `hash(key) % shard_count`. Distribution is even given a good hash, so hotspots are rare. The cost: range queries on the key now scatter across all shards, natural ordering is lost, and changing `shard_count` rehashes and moves most of the data. **Consistent hashing** limits how much data moves when the shard count changes, though applying it to a database has its own complications.

**Directory-based.** A lookup service maps keys (or key ranges, or other attributes) to shards. The application consults the directory first, then connects to the named shard. Most flexible — add a shard or move data by editing the directory — but the directory is an extra hop on every query and a single point of failure unless it is itself made highly available.

## Try the cheaper options first

Sharding is a common interview topic and an over-eager first reach. Before it, check:

- **Vertical scaling** — more CPU, RAM, faster disks on the current server. If the working set fits in RAM on a modern box, you may be done.
- **[Read replicas](/citadel/interview/read-replica-pattern)** — if the bottleneck is read load, replicas are far simpler and often sufficient.
- **[Caching](/citadel/interview/caching)**, indexing, and denormalising the hottest queries.

Shard when data volume outgrows one server, write throughput outgrows one primary, reads outgrow what replicas can serve, or data must be split geographically. Those, covered together in [scaling an application](/citadel/interview/scalability), are the signals that the complexity is finally worth it.
