---
title: Advanced Data Structures - The Specialized Toolkit for Large-Scale Systems
description: A catalogue of the structures that show up once systems get big and distributed - token and leaky buckets, consistent hashing, rsync, Merkle trees, Raft and Paxos, HyperLogLog, Count-Min Sketch, timing wheels, and operational transformation.
date: 2021-04-05
draft: false
slug: /algorithms/advanced
tags:
  - Algorithms
  - Data Structures
  - Distributed Systems
---

Beyond the textbook structures, large-scale systems lean on a specialized set of algorithms — for rate limiting, data distribution, integrity checking, consensus, cardinality estimation, timer management, and real-time collaboration. This is a reference to the ones that come up most, each in a paragraph: what it solves, how it works, where it's used.

## Rate limiting

### Token bucket

Controls an average request rate while allowing short bursts. A bucket accumulates tokens at a fixed rate `r` up to a capacity `b`; each request consumes a token, and a request with no token available is delayed or dropped. The bucket's fill level is the burst budget: idle for a while and you can spend `b` requests at once, then you're back to `r`. Simple to implement, and the standard choice for API rate limiting behind [load balancers](/citadel/interview/load-balancing).

### Leaky bucket

Enforces a strictly constant output rate. Requests enter a fixed-size FIFO queue; the queue drains ("leaks") at a fixed rate regardless of how bursty the input was; an arrival to a full queue is dropped. Where token bucket permits bursts up to `b`, leaky bucket smooths everything to the drain rate — better when a downstream service has a hard fixed capacity.

## Data distribution and synchronization

### Consistent hashing

Distributes keys across a changing set of servers with minimal disruption. Naive `hash(key) % N` remaps almost every key when `N` changes. Consistent hashing places both servers and keys on a hash ring (0 to $2^{32}-1$); a key belongs to the first server clockwise from it. Add or remove a server and only the keys in that one arc move — about $1/N$ of them. Each physical server is given many **virtual nodes** scattered around the ring so load stays even and a departing server's keys spread across many successors rather than dumping on one. Used by distributed caches, Cassandra and DynamoDB partitioning, and [Slack's channel servers](/citadel/system-design/slack); it's the mechanism behind [sharding that survives resharding](/citadel/interview/data-sharding).

### The rsync algorithm

Synchronizes a file between two machines by sending only the changed bytes. The receiver splits its copy into fixed-size blocks and sends the sender a weak rolling checksum (Adler-32) and a strong checksum (MD5) for each. The sender slides a window over its version, cheaply testing every offset with the rolling checksum and confirming hits with the strong one. It then sends only the blocks the receiver lacks, plus instructions to reuse the blocks it already has. Big files with small edits transfer in a fraction of the time. Used for backups, mirroring, and deploys.

### Merkle tree

Verifies and compares large datasets without reading all the data. Leaves are hashes of data blocks; each internal node is the hash of its children's hashes; the root is a single hash of the whole set. If two roots match, the datasets are almost certainly identical. If they differ, you descend only the subtrees whose hashes disagree, pinpointing the changed blocks in $O(\log n)$. Any tampered block changes every hash on its path to the root. Used by Git (commit and tree objects), Bitcoin (transactions per block), and Cassandra and DynamoDB anti-entropy repair between replicas.

## Distributed consensus

### Raft and Paxos

Get a group of servers to agree on a sequence of operations despite crashes and lost messages (not malicious nodes). This is what makes a replicated state machine fault-tolerant.

**Paxos**, from Leslie Lamport, is the original — provably correct, famously hard to understand and implement. It has proposers, acceptors, and learners, and runs a two-phase prepare/accept protocol.

**Raft** was designed to be understandable. It elects a single **leader**; the leader takes client commands, appends them to its log, and replicates entries to followers; an entry is **committed** once a majority has stored it, and Raft's rules guarantee a committed entry survives leader changes. The system keeps working as long as a majority of servers can communicate. Used by etcd, Consul, and CockroachDB for leader election and log replication.

## Probabilistic structures for big data

These trade a small, bounded error for large savings in space and time.

### HyperLogLog

Estimates **cardinality** — the number of distinct elements — of a huge multiset in a few kilobytes. Hash each element; the more distinct elements you've seen, the more likely one of their hashes starts with a long run of leading zeros (like waiting for a long streak of coin-flip heads). HLL uses a bank of registers, each tracking the longest zero-run for the elements routed to it, and combines them with a harmonic mean. It estimates billions of uniques with a percent or two of error, and two HLL sketches merge trivially to give the union's cardinality. Used for unique-visitor and unique-query counts over streams.

### Count-Min Sketch

Estimates the **frequency** of items in a stream in sub-linear space. Keep a `d`-by-`w` grid of counters and `d` hash functions. On each arrival, increment one counter per row (the column its hash selects). To query an item's count, hash it in every row and take the **minimum** of those `d` counters. Collisions can only inflate a count, never deflate it, so the estimate is an upper bound with a tunable error. Used to find heavy hitters — the most frequent IPs, URLs, or search terms — in real time.

## System mechanisms

### Hierarchical timing wheel

Manages millions of timers without a giant priority queue. A wheel is a circular array of slots, each a bucket of timers; a "hand" advances one slot per tick, firing that slot's timers. One wheel covers a limited range, so wheels are stacked by granularity (milliseconds, seconds, minutes); when the fast wheel completes a revolution it cascades one step on the next, redistributing those timers into the fast wheel as they come due. Adding or cancelling a timer is $O(1)$. Used for network timeouts and for Kafka's delayed-operation scheduler.

### Operational transformation

Lets many users edit one document live and converge to the same result. Each edit is an **operation** (insert `"x"` at position 5). Before applying an incoming operation, a server (or client) **transforms** it against the concurrent operations it has already applied, adjusting its parameters — a prior insert earlier in the document shifts this operation's position. The transformed operation is applied and broadcast. Used by [Google Docs](/citadel/system-design/google-docs); [CRDTs](/citadel/system-design/google-docs) are the alternative approach, pushing the merge into the data type so no central transform is needed.

## Database internals

- **LSM tree (log-structured merge-tree).** Optimizes write throughput. Writes go to an in-memory sorted table (often a skip list); when it fills, it's flushed to disk as an immutable **SSTable**; background **compaction** merges SSTables and drops superseded keys. A read may check the memtable and several SSTables (a Bloom filter per SSTable skips most). Behind Cassandra, RocksDB, LevelDB — and [the store migrations at Discord](/citadel/system-design/discord).
- **Inverted index.** Maps each term to the list of documents (and positions) containing it, so a keyword query is a lookup and a merge rather than a scan. The core of [search engines](/citadel/system-design/search-engine); Lucene, which powers Elasticsearch and Solr, is the reference implementation.

## Key takeaways

- Large-scale systems draw on a specialized toolkit beyond the standard data structures.
- **Rate limiting** (token / leaky bucket), **data distribution** (consistent hashing, rsync), and **integrity** (Merkle trees) each solve one recurring infrastructure problem.
- **Raft and Paxos** provide crash-fault-tolerant consensus for replicated logs.
- **HyperLogLog** and **Count-Min Sketch** trade bounded error for huge space savings on cardinality and frequency over streams.
- **Timing wheels** scale timer management; **operational transformation** (and CRDTs) enable real-time collaboration.
- Most of these accept an approximation or an eventual-consistency trade to buy performance at scale.
