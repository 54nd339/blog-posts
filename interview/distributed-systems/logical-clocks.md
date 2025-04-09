---
title: Time, Clocks, and Ordering in Distributed Systems
description: Wall clocks on different machines disagree by milliseconds and sometimes jump backwards, so you cannot order events across servers by timestamp. Lamport clocks give a consistent total order; vector clocks detect whether two events were actually concurrent; hybrid clocks and TrueTime bring physical time back in, carefully.
date: 2025-04-09
draft: false
slug: /interview/logical-clocks
tags:
  - System Design
  - Interview Prep
  - Distributed Systems
---

Two servers each write a record and stamp it with `System.currentTimeMillis()`. Server A's clock is 40 ms ahead of Server B's. A user's write on B genuinely happened after their write on A, but B's smaller timestamp says otherwise. Now "last write wins" picks the wrong one, and you've silently lost data.

Wall clocks across machines disagree (NTP keeps them within milliseconds to tens of milliseconds, not closer), drift between syncs, and can step *backwards* when NTP corrects them or a leap second lands. So you can't use them to order events across servers. What you can do is track *causality* — which events could have influenced which — and that's what logical clocks give you.

## Happens-before

Define `a → b` ("a happens-before b") when:

- `a` and `b` are on the same node and `a` came first; or
- `a` is sending a message and `b` is receiving it; or
- transitively, `a → c` and `c → b`.

If neither `a → b` nor `b → a`, the events are **concurrent** — nothing connects them, and any order between them is equally valid. Causality is a partial order; ordering across concurrent events is a choice, not a fact.

Why it matters: debugging (what caused this?), consistency (apply writes in a causal order), garbage collection (is this object still reachable, given all nodes' views?), and conflict resolution (were these two writes concurrent, or did one see the other?).

## Lamport clocks

Each node keeps a counter `C`. Rules:

- Before any local event (including sending a message), `C = C + 1`.
- Attach `C` to every message sent.
- On receiving a message with timestamp `t`, set `C = max(C, t) + 1`.

Guarantee: if `a → b` then `C(a) < C(b)`. Tie-break equal counters by node ID and you get a **total order** consistent with causality — good enough to, say, order requests in a queue or resolve which of two operations "wins" deterministically.

The limitation: the converse doesn't hold. `C(a) < C(b)` does **not** imply `a → b` — they might be concurrent and just have different counter values. A Lamport clock can order events but can't tell you whether an order was forced by causality or is arbitrary.

## Vector clocks

To detect concurrency, each node keeps a **vector** `V` of counters, one per node. Rules:

- Before a local event, increment your own entry: `V[self] += 1`.
- Attach the whole vector to messages.
- On receive, take the element-wise max with the incoming vector, then increment your own entry.

Now compare two vectors:

- `V(a) < V(b)` (every entry `≤`, at least one `<`) ⇒ `a → b`.
- `V(b) < V(a)` ⇒ `b → a`.
- Neither ⇒ **concurrent** — and you *know* it, which a Lamport clock couldn't tell you.

This is exactly what [Dynamo-style stores](/citadel/system-design/key-value-store) use as **version vectors**: a read that finds two versions checks their vectors — if one dominates it's newer, if neither dominates the writes were concurrent and the store returns both for the application to merge.

The cost: the vector is O(number of nodes), which grows and needs pruning in a large or churny cluster. **Dotted version vectors** and server-side-only versioning trim this.

## Hybrid Logical Clocks

Lamport clocks are causally correct but drift arbitrarily far from wall-clock time, which makes "give me everything from the last hour" impossible. **HLC** combines both: a timestamp is `(physical_time, logical_counter)`. On a local event, take `max(last_physical, wall_clock())`; if the physical part didn't advance, bump the logical counter. On receive, take the max of both parts. The result stays within a bounded distance of true wall-clock time *and* respects causality — you can range-query by the physical part and still trust the ordering. CockroachDB and MongoDB use HLC.

## TrueTime

Google's Spanner takes the opposite approach: make physical clocks *trustworthy* with GPS and atomic clocks in every datacenter, and expose time as an **interval** `[earliest, latest]` with a guaranteed bound (a few milliseconds) on the true time. To commit a transaction at timestamp `t`, Spanner **waits out the uncertainty** — it holds the commit until `t` is definitely in the past everywhere (`t < TT.now().earliest`). That "commit wait" is a few milliseconds of deliberate latency in exchange for **external consistency**: if transaction T1 commits before T2 starts anywhere in the world, T1's timestamp is smaller. It works because Google can pay for the hardware to shrink the uncertainty; most systems can't.

## Practical fallout

- **Never** resolve conflicts by comparing wall-clock timestamps from different machines — that's how last-write-wins loses data.
- Use **monotonic** clocks (`System.nanoTime`, `CLOCK_MONOTONIC`) for measuring elapsed time and timeouts — they never jump backward. Use wall clocks only for displaying "when."
- If you need causal consistency, propagate a logical clock (Lamport or vector) with your messages and stored versions.
- Leases and timeouts must tolerate clock error: a lease "valid for 10s" should be treated as expired earlier by the holder and later by others, so two nodes never both think they hold it.

## The one idea to keep

You can't order events across machines by timestamp — clocks disagree and jump backward. Track causality instead: Lamport clocks (`C = max(local, received) + 1`) give a total order consistent with happens-before, but can't tell a forced order from an arbitrary one. Vector clocks can — if neither vector dominates, the events were genuinely concurrent, which is why version vectors drive conflict resolution in available stores. HLC and TrueTime are the ways to put bounded physical time back into the picture without reintroducing the drift bug.
