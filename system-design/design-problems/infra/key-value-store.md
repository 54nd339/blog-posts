---
title: Designing a Distributed Key-Value Store - The Dynamo Playbook
description: Spread a hash table across a thousand machines that fail routinely, and every classic decision resurfaces - how to partition keys, how many copies to keep, whether a read waits for all of them, and what to do when two writes conflict. Amazon's Dynamo paper is the template most of these stores still follow.
date: 2025-07-24
draft: false
slug: /system-design/key-value-store
tags:
  - System Design
  - Design Problems
---

A single-node key-value store is a hash map with a network port. `put(key, value)`, `get(key)`, done. The design problem starts when the data no longer fits on one machine and the machine is no longer reliable — which, at a thousand nodes, means something is always down.

Amazon's 2007 **Dynamo** paper laid out one coherent set of answers: partition with consistent hashing, replicate to a handful of nodes, let the client tune how many replicas a read or write waits for, and resolve the conflicts that this inevitably creates with version vectors. Cassandra, Riak, Voldemort, and DynamoDB itself are all variations on it. This walks the decisions.

## What it has to do

- `get(key)` and `put(key, value)` at very high throughput, low latency.
- **Always writeable** — an `put` should succeed even during failures and network partitions. (This is the [CAP](/citadel/interview/latency-consistency) choice: availability over strong consistency.)
- **Incremental scalability** — add one node at a time, and only a small slice of data should move.
- **No single point of failure** — every node is equal; there is no coordinator to lose.

## Partitioning: consistent hashing

Hash keys and nodes onto the same ring; a key belongs to the first node clockwise from it. Adding a node steals only the arc between it and its predecessor — about `1/N` of the keys — instead of remapping everything, which is what naive `hash(key) % N` would do. Each physical node holds many **virtual nodes** (points on the ring) so load evens out and a departing node's share spreads across many peers rather than dumping on one neighbour. This is covered in depth under [hash tables](/citadel/data-structures/hash-tables).

## Replication

Store each key on the `N` nodes clockwise from its ring position (the **preference list**). `N = 3` is typical. The first node is the coordinator for that key; if it's down, the next live node on the list takes over.

Because the preference list skips over temporarily dead nodes, a write can land on a node that isn't a "natural" owner. That node holds the data with a **hint** saying who it really belongs to, and hands it back when the rightful owner returns — **hinted handoff**. The cluster stays writeable through failures at the cost of temporary misplacement.

## Tunable consistency: N, R, W

The client (or the store's config) picks two more numbers:

- **W** — how many replicas must acknowledge a write before it's considered successful.
- **R** — how many replicas a read contacts and waits for.

The coordinator sends the operation to all `N` replicas but returns as soon as `W` (or `R`) respond.

- **`R + W > N`** guarantees a read overlaps at least one node that saw the latest write — read-your-writes, at the cost of latency (you wait for more nodes).
- **`R + W ≤ N`** is faster but a read can miss a recent write.
- Common presets: `N=3, W=2, R=2` (balanced), `W=1` (fast writes, weak durability), `R=1` (fast reads, may be stale).

There's no global consensus and no locking. That's the whole point — it's why the store stays available during a partition — and it's why conflicts happen.

## Conflict resolution

Two clients write the same key on opposite sides of a partition. Both succeed. Now there are two values. The store can't pick a winner by wall-clock time — clocks drift, and "last write wins" silently discards data.

Dynamo attaches a **version vector** (a vector clock) to each value: a map of `node → counter`, incremented by the coordinating node on each write. When a read finds two versions:

- if one vector **dominates** the other (every counter ≥, at least one greater), it's strictly newer — return it;
- if neither dominates, the writes were **concurrent** — return *both* and let the application reconcile (merge two shopping carts, take the union, ask the user).

Stores that don't want to burden the app offer **last-write-wins** with a synchronized-ish timestamp, accepting silent loss for keys where it doesn't matter.

## Anti-entropy: making replicas converge

Hinted handoff covers short outages. For a node that was down long enough to miss many writes, replicas compare **Merkle trees**: each node builds a hash tree over its key range, two nodes exchange trees top-down, and only the subtrees whose hashes differ get walked and repaired. A full key-by-key comparison would be prohibitive; the tree makes divergence cheap to locate.

**Read repair** is the opportunistic version: when a read contacts `R` nodes and finds stale copies, the coordinator pushes the fresh value to them in the background.

## Membership and failure detection

No coordinator, so nodes learn about each other by **gossip**: each node periodically picks a random peer and exchanges its view of the ring and of who's alive. Failure detection is local and lazy — node A marks node B "down" only when its own requests to B time out, not on a global vote, because a global membership consensus would reintroduce the single point of coordination the design avoids.

## Storage engine

Underneath the distribution, each node needs a local store. The common choice is an **LSM tree** (log-structured merge tree): writes go to an in-memory table and a commit log, flush to sorted immutable files (SSTables), and background compaction merges them. LSM favours write throughput and sequential I/O — the right trade for a store expected to absorb heavy writes. B-trees are the alternative when reads dominate. See [heaps and trees](/citadel/data-structures/trees) for the underlying structures and [how an SSD works](/citadel/system-design/ssds) for why sequential writes matter.

## The one idea to keep

A distributed key-value store is a hash table that has accepted it will be partitioned, so it drops locking and global consensus entirely. Consistent hashing decides where keys live, `N` replicas keep them alive, and the client dials `R` and `W` to trade latency against staleness — with `R + W > N` buying read-your-writes. The cost of never blocking is that concurrent writes both win, so the store keeps version vectors and hands genuine conflicts back to the application, while Merkle-tree anti-entropy quietly pulls lagging replicas back into line.
