---
title: Discord's Message Storage - From MongoDB to Cassandra to ScyllaDB
description: Discord's message database went from a single MongoDB replica in 2015 to 177 Cassandra nodes holding trillions of rows to a ScyllaDB cluster. The bottleneck that forced each move, and the p99 latency numbers that justified the last one.
date: 2025-05-12
draft: false
slug: /system-design/discord
tags:
  - System Design
  - Case Study
  - Databases
---

In 2015 Discord stored its messages in a single MongoDB replica. By 2022 it was holding trillions of them and had rebuilt the message database twice on the way. Each rebuild followed the same script: a store that was fine at one scale became the bottleneck at the next, and the fix was a different storage engine, not a bigger server.

This walks through the three phases — what broke, what replaced it, and why — drawn from Discord's own engineering write-ups.

## The workload

Messages are the core of Discord, and the access pattern is unforgiving. Reads dominate: a busy channel has hundreds of people scrolling history at once, so one channel becomes a read hotspot. The dataset only grows — messages are rarely deleted — and every read has to stay fast enough to feel instant. The database has to absorb that with predictable, low tail latency while the row count climbs by orders of magnitude.

## Phase 1: MongoDB (2015)

Discord launched on a single MongoDB replica — the usual startup choice for schema flexibility and speed of setup. By November 2015 it held about 100 million messages.

The problem showed up as latency that would not stay predictable. Once the data and its indexes outgrew RAM, MongoDB had to hit disk for queries that used to be served from memory, and response times became erratic. A working set that no longer fits in memory is the recurring theme of this whole story.

## Phase 2: Cassandra (2017–2022)

Discord moved messages to Cassandra, a distributed NoSQL database built for horizontal scale and fault tolerance — add nodes, and data and load spread across them. It ran 12 nodes in 2017 holding billions of messages, and 177 nodes by early 2022 holding trillions.

At that size, new problems surfaced:

- **Unpredictable latency again**, this time from the storage engine's own maintenance. Cassandra uses an **LSM tree** (log-structured merge-tree): writes land in memory and are flushed to immutable **SSTables** on disk, which background **compaction** later merges. At trillions of rows, compaction was expensive to run and dragged on read performance while it ran.
- **Reads cost more than writes** in an LSM tree, because a read may have to check several SSTables. Combined with many concurrent readers on a hot channel, this produced hotspots.
- **Garbage-collection pauses.** Cassandra is written in Java, and [GC pauses](/citadel/interview/garbage-collection) showed up directly as latency spikes.

## Phase 3: ScyllaDB (2022)

Discord switched to ScyllaDB: a database that speaks Cassandra's API and uses the same data model, but is written in C++. No JVM means no GC pauses. Its "shard-per-core" design pins data to a specific core and avoids cross-core locking, which keeps tail latency tight under load.

The migration came with a broader redesign: a data-services tier written in Rust sitting between the API and the database, one service per set of partitions. Its main job is **request coalescing** — if a thousand users pull the same uncached row in the same instant, the data service issues *one* query to ScyllaDB and fans the result back out to all thousand. That alone flattens the read hotspots that a popular channel used to create, because the database only ever sees one in-flight request per hot row.

The results, at p99:

- **Read latency: 15 ms**, down from 40–125 ms on Cassandra.
- **Write latency: 5 ms**, down from 5–70 ms.

## What the rewrites had in common

Every phase ended the same way: the working set outgrew memory, or the engine's own housekeeping started competing with live traffic, and tail latency became unpredictable. Discord's answer was never to tune the existing system harder — it was to re-pick the storage engine for the new scale. The lesson isn't "use ScyllaDB"; it's that a database choice has a scale range, and staying inside it is an ongoing decision. (For the roughly-time-sorted IDs Discord attaches to every message, see [globally unique IDs](/citadel/system-design/unique-ids).)
