---
title: Cassandra and ScyllaDB - Wide-Column at Scale
description: A masterless store designed for one thing - absorbing writes and staying up. No primary node to fail, tunable consistency per query, and an LSM storage engine that makes writes cheap. The price is that you model tables around the exact queries you will run, because there are no joins and secondary indexes are a trap.
date: 2024-04-03
draft: false
slug: /tech/cassandra
tags:
  - Tools
  - Databases
---

Cassandra came out of Facebook (2008), combining Amazon [Dynamo](/citadel/system-design/key-value-store)'s masterless distribution with Google Bigtable's data model. It's built for a specific shape of problem: enormous write volume, geographic distribution, and a hard requirement that the database keep accepting writes even when nodes and whole data centres are down. Discord's message store, Netflix's viewing history, and Apple's fleet of tens of thousands of Cassandra nodes are the canonical users.

What you give up for that is relational flexibility. There are no joins, transactions are limited, and you design your tables backward from your queries. Get the data model wrong and Cassandra performs terribly; get it right and it scales close to linearly.

## Masterless ring

Every node is equal — there is no primary, no config server, no single point of failure. Nodes form a ring; data is partitioned across it by **consistent hashing** with **virtual nodes** (each physical node owns many small ranges, so load is even and a failure spreads across many peers). Nodes gossip to share membership and state. To add capacity you start a new node; it claims ranges and streams data from its neighbours.

Clients can connect to any node, which acts as the **coordinator** for that request — it forwards to the replicas that own the key and assembles the response.

## Data model: query-first

- **Keyspace** — a namespace with a replication strategy (like a database).
- **Table** — has a **primary key** split into two parts:
  - **Partition key** — hashed to decide which node(s) store the row. *All rows with the same partition key live together on the same nodes.*
  - **Clustering columns** — sort the rows *within* a partition on disk.

```sql
CREATE TABLE messages (
  channel_id  bigint,
  bucket      int,          -- e.g. day, to bound partition size
  message_id  timeuuid,
  author_id   bigint,
  content     text,
  PRIMARY KEY ((channel_id, bucket), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

This table answers exactly one query well: "recent messages in a channel," because they're one partition, pre-sorted by `message_id` descending. It cannot efficiently answer "all messages by this author" — that's a different access pattern and needs a *different table* holding the same data keyed differently. **You denormalize and write to multiple tables**; storage is cheap, and reads must be single-partition point-or-range lookups.

The rules that follow:

- **Every query hits one partition** (or a known small set). A query that would scan the ring is an anti-pattern.
- **Bound partition size.** A partition that grows without limit (all of a channel's messages forever) becomes a hotspot and eventually unreadable. Add a time bucket to the partition key.
- **Secondary indexes are a trap.** A `CREATE INDEX` on a non-partition-key column makes queries fan out to *every* node. Use a manually-maintained lookup table or a materialized view instead.

## Tunable consistency

There's no locking and no consensus on the write path. Instead, per query you choose how many replicas must respond:

- `ONE` — one replica; fastest, may be stale.
- `QUORUM` — a majority of replicas (`RF/2 + 1`).
- `LOCAL_QUORUM` — a majority *within the local data centre* (avoids cross-DC latency).
- `ALL` — every replica; slowest, least available.

If read consistency + write consistency > replication factor (`R + W > RF`), a read is guaranteed to see the latest write — `QUORUM` reads and writes with `RF = 3` gives that. Choose weaker for throughput, stronger for correctness, per statement.

Behind the scenes: **hinted handoff** (a coordinator stores writes destined for a down node and replays them later), **read repair** (a read that finds stale replicas pushes the fresh value), and periodic **anti-entropy repair** (`nodetool repair`, comparing Merkle trees) keep replicas converging. Running repair regularly is operationally mandatory.

## Storage engine: LSM

Cassandra's local engine is a **log-structured merge tree**:

1. A write appends to the **commit log** (durability) and updates an in-memory **memtable**.
2. When the memtable fills, it's flushed to an immutable, sorted **SSTable** on disk.
3. A read may need to check several SSTables plus the memtable; a **bloom filter** per SSTable skips ones that definitely don't have the key, and a partition index locates it within one.
4. **Compaction** merges SSTables in the background, discarding overwritten values and expired data.

This makes writes cheap (append + memory) and sequential. The costs: reads can touch multiple SSTables, and **tombstones** — deletes are markers, not removals, kept around until compaction and past a `gc_grace_seconds` window so a deleted value doesn't resurrect from a lagging replica. A workload that deletes a lot, or writes many `null`s, or does range queries over ranges full of tombstones, gets slow and can even error (`TombstoneOverwhelmingException`). Compaction strategy matters: **STCS** (size-tiered, write-optimized), **LCS** (leveled, read-optimized, more I/O), **TWCS** (time-windowed, ideal for time-series with TTL).

## Write and read paths, briefly

- **Write**: coordinator → replicas → each replica writes commit log + memtable, acks; coordinator returns when `W` replicas acked.
- **Read**: coordinator → `R` replicas → each returns its newest value (memtable + relevant SSTables via bloom filter + index); coordinator returns the newest, triggers read repair on stale ones.

`LWT` (lightweight transactions, `IF NOT EXISTS`) provide compare-and-set using Paxos across replicas — correct but much slower, so use sparingly (a uniqueness check, not a general transaction).

## ScyllaDB

A drop-in-compatible rewrite in C++ with a **shard-per-core** architecture (each CPU core owns a slice of data and its own memory, event loop, and I/O queue — no shared state, no locks) built on the Seastar framework. No JVM means no garbage-collection pauses, which were Cassandra's worst latency spikes. Same data model, same CQL, same ring — often several times the throughput per node and much tighter tail latency. Discord's [migration from Cassandra to ScyllaDB](/citadel/system-design/discord) was largely about killing GC pauses and reducing node count.

## The one idea to keep

Cassandra is masterless and write-optimized: no primary to fail, an LSM engine that makes writes a memory append plus a sequential flush, and per-query tunable consistency (`QUORUM` reads and writes with `RF=3` for read-your-writes) instead of locking. The cost is you model tables backward from queries — one table per access pattern, every query confined to one bounded partition, no joins, secondary indexes avoided — and you must run repair and watch tombstones. ScyllaDB is the same design without the JVM's GC pauses.
