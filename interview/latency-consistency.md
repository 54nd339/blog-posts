---
title: Latency vs Consistency - The CAP Trade-Off in Practice
description: Making every node agree on the latest data takes coordination, and coordination takes time. The CAP theorem framing, a replication example that makes the trade concrete, and where real systems choose to sit.
date: 2024-07-31
draft: false
slug: /interview/latency-consistency
tags:
  - System Design
  - Interview Prep
  - Distributed Systems
---

For every node in a distributed system to be certain it has the latest data, the nodes must coordinate — agree before an operation counts as done — and that agreement takes time. So the faster a system answers, the less it can promise that the answer is current. This is the latency-consistency trade, and it is not a bug to fix but a dial to set.

This covers the terms, the CAP theorem that frames the choice, a replication example that makes it concrete, and how different applications set the dial.

## The terms

**Latency** is the wait before a response after an action. It has money attached: Amazon reported every 100 ms of added latency cost about 1% of sales.

**Consistency** here means every node showing the same data at the same time. Two flavours:

- **Strong** — every read returns the most recent write, or an error. Updates are visible everywhere immediately.
- **Eventual** — with no new writes, all replicas *eventually* converge to the last value. Replicas may disagree for a while. This is the **BASE** posture (basically available, soft state, eventual consistency) common to NoSQL stores.

## CAP

The **CAP theorem** says a distributed data store can guarantee at most two of:

- **Consistency (C)** — every read gets the most recent write or an error (here, a strong form: linearizability).
- **Availability (A)** — every request gets a non-error response, no guarantee it is current.
- **Partition tolerance (P)** — the system keeps operating even when the network drops or delays messages between nodes.

Network partitions are guaranteed to happen, so **P is not optional** for a real distributed system. That leaves the real choice, made *during a partition*:

- **CP** — stay consistent, sacrifice availability: refuse or error on requests it cannot serve correctly.
- **AP** — stay available, sacrifice strong consistency: nodes may return different versions until the partition heals.

CAP is a coarse lens — Martin Kleppmann's "Please stop calling databases CP or AP" argues the two terms hide too much nuance — but it names the fundamental tension cleanly.

## The trade, concretely

![Three timelines of an API write replicated from a primary data node to two secondaries. In the first, the API service's wait spans until all three nodes acknowledge — best consistency, highest latency. In the second, it ends after the primary and one secondary acknowledge — medium, medium. In the third, it ends as soon as the primary persists — worst consistency, lowest latency.](../images/latency-consistency.png "When is a replicated write 'done'? Waiting for more acknowledgements buys consistency and costs latency; the choice is a quorum size.")

An API writes to a primary node that replicates to two secondaries. When is the write "successful"?

- **Wait for all three.** Any later read from any node sees the write. Best consistency, highest latency — the client waits on the slowest of three acknowledgements.
- **Wait for primary + one secondary.** One replica may briefly lag. Medium consistency, medium latency.
- **Wait for the primary only.** Client gets a fast response; a read from a lagging secondary right after can be stale. Lowest latency, weakest consistency.

The choice is a quorum size, and it can be set per operation.

## Where systems land

- **Banking and payments** — strong consistency, nearly always. A wrong balance or a half-applied transfer is unacceptable; extra latency is the acceptable price.
- **Social feeds, product views** — eventual consistency is fine. A like or comment taking a few seconds to fan out globally harms nothing; availability and speed win.
- **Inventory** — mixed. Browsing can read slightly stale replicas, but the checkout decrement needs strong consistency so two customers do not both buy the last unit.

## Managing the trade

The tension does not go away, but it can be softened:

- **[Caching](/citadel/interview/caching)** cuts read latency, and adds its own consistency and [invalidation](/citadel/interview/cache-pitfalls) problem.
- **Asynchronous processing** — push work that needs no immediate answer (a confirmation email) to the background so the user-facing action returns fast.
- **Tunable consistency** — many distributed databases let you pick the quorum per query or per table.
- **[Optimistic concurrency](/citadel/interview/race-condition)** — let operations proceed and check for conflicts at commit, which beats locking when contention is low.

## The skill is choosing on purpose

There is no setting that is fast, consistent, and available all at once under a partition, so good design is not finding one — it is deciding, per piece of data, which property you will give up and being explicit about it. The replication quorum, the isolation level, the cache TTL are all the same decision at different layers: how stale is acceptable here, in exchange for how much speed. The [read replica](/citadel/interview/read-replica-pattern) and [sharding](/citadel/interview/data-sharding) posts show this trade in the data tier specifically.
