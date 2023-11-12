---
title: Latency Numbers - From an L1 Cache Hit to a Cross-Continent Round Trip
description: The time an operation takes spans ten orders of magnitude, from a nanosecond cache reference to a hundred-millisecond hop across the Atlantic. A ladder of the numbers worth memorising, and the design choice each rung forces.
date: 2023-11-12
draft: false
slug: /random-thoughts/latency-number
tags:
  - Notes
  - Latency
  - Performance
---

Reading a value from RAM takes about 100 nanoseconds. Sending a packet from California to the Netherlands and waiting for the reply takes about 100 milliseconds — a million times longer. Almost every system-design decision is a bet on which end of that range you land on: keep the data close and cheap to reach, or pay for the trip.

The numbers below were popularised by Jeff Dean and get rebuilt every few years as hardware moves, so treat them as orders of magnitude, not measurements. What you want in your head is the *shape* of the ladder: each rung is roughly 10× or 100× the one below it, and knowing which rung an operation sits on tells you whether it's worth worrying about.

## The ladder

One nanosecond (ns) is $10^{-9}$ s. A microsecond (µs) is $10^{-6}$ s — a thousand ns. A millisecond (ms) is $10^{-3}$ s — a thousand µs, a million ns.

| Operation | Ballpark | What it is |
| --- | --- | --- |
| L1 cache reference | ~1 ns | On-die memory, the smallest and fastest tier. The CPU manages it; application code doesn't. |
| L2 cache reference | ~10 ns | A larger, slightly slower backup for L1, still on or beside the core. |
| Main memory (RAM) read | ~100 ns | An in-memory store like Redis serves data that's already resident in this range. |
| Send 1 KB over a 1 Gbps network | ~10 µs | LAN transfer only — e.g. Memcached shipping 1 KB to an app server. No processing time on either end. |
| Read from an SSD | ~100 µs | No moving parts, so far faster than a spinning-disk seek. RocksDB on SSD reads in this range. |
| Database commit (RDBMS) | ~1 ms | Not just a write: index updates plus a transaction-log flush for durability. About 1 ms in PostgreSQL. |
| Round trip, California ↔ Netherlands | ~100 ms | A cross-continent internet hop and back. The speed of light sets the floor. |
| Monitoring refresh or retry interval | 1–10 s | Not hardware — an application choice. Grafana dashboards default to a 5–10 s refresh. |

## The gaps between the rungs

The jumps are what matter, because they compound:

- RAM (~100 ns) is about **1,000× faster** than an SSD read (~100 µs).
- An SSD read (~100 µs) is about **10× faster** than a database commit (~1 ms).
- A database commit (~1 ms) is about **100× faster** than a cross-continent round trip (~100 ms).

Stack those and you go from a RAM read to a global round trip in three steps, a factor of roughly a million. Nothing you do in application code closes a gap that size; you can only choose which side of it to stand on.

## What the ladder decides

**Caching in RAM works** because it moves a lookup from the disk or network rungs up to the ~100 ns rung — a jump of three or four orders of magnitude, not a tuning tweak. See [caching placement and strategies](/citadel/interview/caching) and [the memory hierarchy behind these tiers](/citadel/interview/computer-memories).

**CDNs exist** to shorten the round trip. If a user in Amsterdam hits an origin in California, the ~100 ms is unavoidable physics; put a copy of the asset in Amsterdam and the same request costs a few milliseconds. See [how a CDN is structured](/citadel/interview/cdn).

**Synchronous database calls add up.** Each commit is ~1 ms, and a request that makes ten of them in sequence has spent 10 ms before any of your own logic runs. Batch them, run them concurrently, or move them off the request path.

**Latency has a dollar value.** In a widely cited internal finding, Amazon measured that every additional 100 ms of latency cost it about 1% in sales. Slow isn't only annoying; it shows up in the revenue line.

## The three cliffs

If you keep one thing, keep the cliffs: CPU cache to RAM is about 100×, RAM to disk or LAN is another 100–1,000×, and your data centre to another continent is another 1,000× on top. Good design is mostly the discipline of not falling down a cliff you didn't need to approach — and, when you must, [deciding what consistency you'll trade for the latency you save](/citadel/interview/latency-consistency).
