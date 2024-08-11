---
title: Cache Failure Modes - Penetration, Stampede, and Eviction
description: A cache is meant to shield the database; under load it sometimes does the opposite. Cache penetration, avalanche and stampede, hot-key breakdown, cache and database drift, and how the eviction policy decides your hit rate.
date: 2024-08-11
draft: false
slug: /interview/cache-pitfalls
tags:
  - System Design
  - Interview Prep
  - Caching
---

A cache earns its place by absorbing reads that would otherwise hit the database. The failure modes below are the situations where it stops doing that — and a flood of traffic lands on the database all at once, often at the worst moment.

Each one has a standard name, a standard cause, and a standard fix. This assumes the [placement and read/write strategies](/citadel/interview/caching) are already in place; what follows is what breaks once that cache is under real load.

## Cache penetration

A request asks for a key that is **in neither the cache nor the database**. The cache misses, the database lookup finds nothing, nothing gets cached, and the next identical request repeats the whole trip. A bad actor hammering random IDs, or a bug generating them, turns this into unbounded database load for answers that are always "not found".

- **Cache the negative result.** When the database returns nothing, store an explicit null marker for that key with a short TTL. Repeat requests hit the marker and stop.
- **Bloom filter of valid keys.** Keep a Bloom filter — a compact probabilistic set — of every key that exists in the database. It never has false negatives, so a "definitely not present" answer lets you skip the cache and database entirely.

## Cache avalanche and stampede

A large batch of keys expires at close to the same instant — or the cache node itself goes down — and every request that was being served from cache now misses simultaneously. The database sees its normal read load plus the entire cache's worth of traffic in one spike.

- **Jitter the TTLs.** Add a small random offset to each expiry so a batch loaded together does not expire together.
- **Single-flight the rebuild.** On a miss for a given key, let one request go to the database and have the others wait for its result (a mutex or a "request coalescing" layer). One query refills the key instead of thousands.
- **Refresh ahead of expiry.** Detect popular entries nearing their TTL and reload them in the background before they lapse.
- **Survive a lost node.** Run the cache as a highly available cluster, and put a circuit breaker in front of the database so a cache outage degrades gracefully instead of taking the database with it.

## Cache breakdown (hot key)

The narrow version of a stampede: a single very popular key expires or is invalidated, and all of its concurrent readers miss at the same moment. The total request rate is fine; it is concentrated on one key.

- **Lock the rebuild.** A distributed lock (via Redis or ZooKeeper) so exactly one process repopulates that key while the rest briefly serve the previous value or wait.
- **Do not expire known hot keys on a timer.** Refresh them from the source on update instead, so there is never a moment when the key is simply absent.

## Cache and database drift

The two copies disagree. It happens when a writer updates the database directly without invalidating the cache, or when a write-back cache fails before flushing pending changes. Readers then get stale or plainly wrong data.

- **Match the write strategy to the consistency need.** For data that must not be served stale, [write-through](/citadel/interview/caching) updates both copies before acknowledging.
- **Invalidate on the write path.** Drive cache invalidation from the same event that changes the database — an application event, a change-data-capture stream, a trigger — rather than hoping a TTL covers it.
- **Keep a TTL as a backstop.** Even with active invalidation, a bounded TTL caps how long any missed invalidation can persist.

## Eviction: the policy sets your hit rate

Cache memory is finite. When it is full and a new entry arrives, the cache must **evict** one. If the policy does not match how the data is actually accessed, the hit rate collapses and most reads fall through to the database anyway.

| Policy | Evicts | Fits | Hurts when |
| --- | --- | --- | --- |
| **LRU** least recently used | the entry idle longest | general use with temporal locality | a big one-off scan evicts genuinely hot entries |
| **MRU** most recently used | the entry just used | cyclic scans where the newest is least needed again | normal workloads with reuse of recent data |
| **SLRU** segmented LRU | from a probationary segment first, then a protected one | mixed workloads; resists scan pollution | needs segment-size tuning; more complex |
| **LFU** least frequently used | the lowest access count | stable popularity over long periods | tastes change; a new hot key starts cold and is evicted early |
| **FIFO** first in, first out | the oldest by insertion | truly sequential access; minimal overhead | ignores reuse — early hot entries get dropped |
| **RR** random replacement | a random entry | a baseline; no bookkeeping | unpredictable; uses no access information |
| **TTL** time to live | on expiry, not on fullness | bounding staleness | does not manage space alone; pair with LRU/LFU |
| **Two-tier** small L1 + larger L2 | per tier, own policy each | hottest data in a tiny fast tier, warm data in a big one | two layers to keep consistent |

Choosing well is three steps: characterise the access pattern (recency-driven leans LRU, frequency-driven leans LFU, scan-heavy needs SLRU or an admission filter); pick the matching policy; then watch hit rate, miss rate, and eviction count and adjust the policy or the size. Redis exposes several of these as `maxmemory-policy`, including `allkeys-lru`, `allkeys-lfu`, and `volatile-ttl`.

## A cache is a shock absorber, not a system of record

Every fix here trades something — a little freshness, an extra lock, a background refresher, a second tier — for keeping load off the database when the cache has a bad moment. That is the right trade, because the database is the copy you cannot lose. Design the cache so that losing all of it means a slow few minutes, never lost data or a cascading outage.
