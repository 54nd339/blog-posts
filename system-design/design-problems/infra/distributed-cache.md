---
title: Designing a Distributed Cache - Sharding, Eviction, and the Thundering Herd
description: A cache tier that outgrows one machine has to answer where each key lives, what to evict when memory fills, and what happens the instant a hot key expires and ten thousand requests all miss at once. The failure modes have names, and each has a standard defence.
date: 2025-07-24
draft: false
slug: /system-design/distributed-cache
tags:
  - System Design
  - Design Problems
---

A cache is a small, fast store in front of a large, slow one. One Redis process holds maybe a few hundred gigabytes. When the working set is bigger than that, or the request rate is more than one process can serve, the cache itself has to be distributed — and now it has the same problems as any distributed store, plus a few that are specific to caching.

The three that matter: how do you place keys across nodes so adding capacity doesn't reshuffle everything, what do you throw away when a node is full, and what do you do about the stampede that happens when a popular key vanishes and every request behind it misses simultaneously.

## What it has to do

- `get` / `set` / `delete` with sub-millisecond latency, high hit ratio.
- Scale horizontally — add nodes for capacity or throughput.
- Tolerate node loss without a correctness bug (a cache miss is acceptable; stale data returned as fresh is not).
- Keep entries roughly consistent with the source of truth.

## Placement

Three topologies:

- **Client-side sharding** (memcached-style). The client library hashes the key and talks directly to the owning node. No proxy hop, but every client needs the node list and the same hashing logic.
- **Proxy** (Twemproxy, Envoy). Clients talk to a proxy that shards. One place to manage routing; one extra hop and a component to scale.
- **Cluster-aware** (Redis Cluster). Nodes know the full key-space map and redirect a misrouted request; the client caches the map.

However you route, shard with **consistent hashing** so adding a node moves `1/N` of the keys, not all of them. A plain `hash(key) % N` re-homes almost every key on a resize, which for a cache means a near-total miss storm against the database — the resize itself causes an outage. Virtual nodes per physical node keep the load even. (Detail under [hash tables](/citadel/data-structures/hash-tables).)

## Eviction

Memory is finite; something has to go. The policies:

- **LRU** (least recently used) — evict the entry untouched longest. The default; matches temporal locality. Real implementations approximate it (Redis samples a handful of keys and evicts the oldest of the sample) because exact LRU needs a linked list touched on every access.
- **LFU** (least frequently used) — evict the entry accessed *least often*, with a decay so a formerly-hot key eventually ages out. Better when popularity is stable and skewed; the default in Redis 4+ (`allkeys-lfu`).
- **TTL-based** — every entry has an expiry; evict expired entries first, then fall back to LRU/LFU.
- **FIFO / random** — rarely the right choice, but random is cheap and surprisingly hard to beat under uniform access.

Set a **max-memory policy** explicitly. The dangerous default on some setups is "return errors when full" — a cache that stops accepting writes turns into a cache that slowly goes stale.

## Read and write patterns

- **Cache-aside** (lazy). App checks cache; on a miss, reads the DB and populates the cache. Simple, resilient (cache down ⇒ just slower), but the first request for any key always misses, and there's a window where the cache is stale after a DB write.
- **Read-through** — the cache library itself loads from the DB on a miss. Same behaviour, cleaner app code, couples the cache to the data source.
- **Write-through** — writes go to cache and DB synchronously. Cache is never stale; every write pays both latencies.
- **Write-back** — writes go to cache, flushed to the DB asynchronously. Fast writes, absorbs bursts, but a node loss before flush loses data — only acceptable for data you can afford to lose (counters, sessions).

Invalidation is the other half: on a DB write, either delete the cache key (next read repopulates) or update it. Deleting is safer — an update can race with a concurrent read that repopulates with the old value.

## The failure modes with names

**Cache stampede / thundering herd / dogpile.** A hot key expires. Between that instant and the first request repopulating it, thousands of concurrent requests all miss and all hit the database with the same query. Defences:

- **Request coalescing** — the first miss for a key acquires a short lock (or a `SETNX` flag); other requests for that key wait briefly for it to repopulate instead of also querying. `singleflight` in Go is this.
- **Early recomputation** — refresh a key *before* it expires, probabilistically, when a request arrives and the key is close to its TTL. The herd never forms because the key never fully goes cold.
- **Stale-while-revalidate** — serve the expired value and refresh in the background.

**Cache penetration.** Requests for keys that exist in neither cache nor database — often malicious, probing random IDs. Each one misses the cache and hits the DB for nothing. Defences: cache the negative result (`key → NOT_FOUND` with a short TTL), and front the cache with a [Bloom filter](/citadel/data-structures/bloom-filters) of valid keys to reject impossible ones without any lookup.

**Cache avalanche.** A large set of keys given the same TTL all expire at the same second, and the miss storm hits every key at once. Fix: add jitter — `ttl + random(0, spread)` — so expiries spread out. Same idea for a cache node restart: warm it gradually rather than pointing full traffic at a cold node.

**Hot key.** One key (a celebrity's profile, a viral post) gets so much traffic that its single owning node saturates. Fixes: replicate that key to multiple nodes and pick one at random per request; add a small client-side local cache for the hottest keys (a two-level cache); or split the value across sub-keys.

## Consistency and node loss

A distributed cache usually does **not** replicate for correctness — a lost node just means those keys miss and repopulate from the DB. Replication (Redis replica + Sentinel, or Cluster's primary/replica pairs) is for *availability*: keep serving reads for a shard while its primary is being replaced, at the cost of memory and the small chance of serving a just-stale replica.

## The one idea to keep

A distributed cache is consistent hashing plus an eviction policy plus a set of named stampede defences. Shard so a resize moves `1/N` of keys, not all of them. Pick LRU or LFU deliberately and set a max-memory policy that isn't "error out." Then plan for the moment a hot key expires: coalesce the requests, or refresh the key before it dies, and always jitter your TTLs so a thousand keys don't expire on the same tick.
