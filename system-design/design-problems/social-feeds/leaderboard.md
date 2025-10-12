---
title: Designing a Real-Time Leaderboard
description: Rank millions of players by a score that changes constantly, and answer both "top 10" and "what's my rank" in milliseconds. A Redis sorted set does it on one node; past that, you shard and accept that global rank becomes an estimate.
date: 2025-10-12
draft: false
slug: /system-design/leaderboard
tags:
  - System Design
  - Design Problems
---

A game shows the top 100 players, and it shows *you* your own rank — "you're #4,182,916 of 20 million" — and both update as scores change. The naive version is `SELECT COUNT(*) FROM scores WHERE score > mine` on every request, which is a full index scan per query and falls over immediately.

The problem has two distinct queries with different costs: **top-N** (cheap — read a sorted prefix) and **rank-of-a-player** (expensive — count everyone above them). A structure that keeps scores sorted answers the first for free and the second in log time. That structure is a sorted set, and the design question is what to do when it no longer fits on one machine.

## What it has to do

- `submit(player, score)` — set or increment a score.
- `top(n)` — the highest `n` players.
- `rank(player)` — 1-based position.
- `around(player, k)` — the `k` players just above and below.
- Global, and segmented (per country, per friend group), and time-windowed (daily / weekly / all-time).

## One node: the sorted set

Redis's **sorted set** (ZSET) stores members with a floating-point score and keeps them ordered. Internally it's a **skip list** plus a hash map, giving:

- `ZADD key score member` — insert/update in O(log n).
- `ZREVRANGE key 0 9 WITHSCORES` — top 10 in O(log n + 10).
- `ZREVRANK key member` — rank in O(log n).
- `ZRANGEBYSCORE` / `ZREVRANGEBYSCORE` — the "around me" window.

For tens of millions of members this is a few hundred MB and every operation is sub-millisecond. Most leaderboards never need more than this, plus a replica for failover and a periodic snapshot to a durable store.

The authoritative scores live in a normal database; the ZSET is an **index** you can rebuild from it. Write path: update the DB row, then `ZADD` the ZSET. If they can diverge (a crash between the two), a periodic reconciliation job replays the DB into a fresh ZSET.

## Ties

Two players on 5,000 points need a deterministic order. Pack a tiebreaker into the score: use `score - (timestamp / 1e13)` so an earlier achiever ranks higher, or store the score as a 64-bit value with the score in the high bits and `(MAX - timestamp)` in the low bits. Now `ZREVRANK` is fully ordered with no special-casing.

## Time windows

Daily and weekly boards are separate ZSETs with a TTL: `lb:daily:2026-03-14`, `lb:weekly:2026-W11`. A submit writes to all active windows. Expired keys are reaped automatically. Rolling windows ("last 24 hours") are harder — you either approximate with hourly buckets you sum, or keep per-event timestamps and re-aggregate, which is back to the expensive path.

## Scaling past one node

Say the member set or the write rate outgrows a single Redis. Two sharding strategies, both of which cost you *exact global rank*:

**Shard by score range.** ZSET A holds scores 0–999, B holds 1,000–1,999, etc. `top(n)` reads from the highest non-empty shard(s). `rank(player)` = (player's rank within its own shard) + (total counts of all higher shards, which each shard reports cheaply as `ZCARD`). This keeps rank *exact* but ranges get unbalanced as the score distribution shifts, needing rebalancing.

**Shard by player hash.** Player → shard by `hash(player_id) % S`. Writes spread evenly, no rebalancing. But now no shard knows the global order. `top(n)` becomes: ask every shard for its top `n`, merge — still exact for the top. `rank(player)` becomes an **estimate**: each shard maintains a coarse **score histogram** (buckets of score → count); to rank a player, sum the "count of scores above mine" across all shards' histograms. Accurate to the bucket width, and O(S) instead of O(anything huge). For "you're roughly #4.2 million," bucket-level accuracy is fine; the exact top of the board is still exact.

## Serving

- **Top-N is cached** — it changes slowly relative to how often it's read. Recompute every few seconds, serve from cache.
- **Rank-of-me** is computed per request but cheap on a ZSET; for the sharded histogram version, cache per player for a few seconds.
- **Around-me** is one `ZRANGEBYSCORE` window.

## Anti-cheat

The leaderboard is a target. Score submission must be **server-authoritative** — the client sends "I completed level 7," the server computes the score from validated game state, never trusts a client-sent number. Add rate limits on submissions, and an anomaly check (score deltas that are physically impossible) that flags accounts for review or a shadow leaderboard.

## The one idea to keep

A leaderboard is a sorted set: top-N is a cheap read of a sorted prefix, rank is a log-time count, and one Redis node handles tens of millions of players. Scaling out forces a choice — shard by score range to keep rank exact at the cost of rebalancing, or shard by player hash for even writes and accept that global rank becomes a histogram estimate while the actual top of the board stays exact. Whatever you do, compute scores on the server; a client-sent number is a cheat waiting to happen.
