---
title: Erasure Coding - Durability Without Full Replication
description: How object stores hit eleven nines of durability without keeping three full copies. Split data into k chunks, add m parity chunks, and survive the loss of any m of the k+m.
date: 2024-08-03
draft: false
slug: /interview/erasure-coding
tags:
  - System Design
  - Interview Prep
  - Storage
---

Cloud object stores advertise durability like "eleven nines" — 99.999999999%, roughly a one-in-ten-thousand chance of losing a single object over ten million years. The obvious way to get there is replication: keep three copies, lose two, still fine. But three copies means paying for 3&times; the storage. **Erasure coding** reaches the same durability at a fraction of that overhead.

## The idea

Split the data into `k` equal chunks, compute `m` extra **parity** chunks from them, and store all `k + m` chunks on different disks or nodes. The property that makes it work: the original data can be rebuilt from *any* `k` of the `k + m` chunks. So you can lose up to `m` chunks — data or parity, in any combination — and still recover everything.

![Erasure coding in four steps: split the file into four data chunks d1 to d4, compute two parity chunks p1 and p2 from them, distribute all six across six nodes, then reconstruct lost chunks d3 and d4 from the four surviving chunks. Below, a comparison bar shows 3-copy replication using 3 GB for 1 GB of data versus a (4+2) scheme using 1.5 GB.](../images/erasure-coding.png "A (4+2) scheme: 4 data chunks, 2 parity chunks, any 2 losses survivable. Storage cost 1.5x versus 3x for three-copy replication. Source: adapted from a system-design illustration.")

## A (4+2) scheme

Take `k = 4`, `m = 2`:

1. **Split** the file into four data chunks `d1, d2, d3, d4`.
2. **Compute** two parity chunks from them with a Reed-Solomon code. Illustratively — not the real arithmetic, which is over a finite field — parities look like weighted sums:
   - `p1 = d1 + 2·d2 − d3 + 4·d4`
   - `p2 = −d1 + 5·d2 + d3 − 3·d4`
3. **Distribute** all six chunks across six nodes.
4. **Lose** up to two. Say the nodes holding `d3` and `d4` crash.
5. **Reconstruct.** Four chunks remain — `d1, d2, p1, p2`. That is two equations (the parity definitions) in the two unknowns `d3` and `d4`; solve them and the data is whole again. Any four of the six suffice.

## Storage overhead

- **(4+2) erasure coding** — 6 chunks stored per 4 chunks of data, an overhead of `m/k = 2/4 = 50%`. One GB of data becomes 1.5 GB on disk.
- **3-copy replication** — three 1 GB copies per 1 GB of data, an overhead of 200%.

Same broad protection (survive two failures), a quarter of the extra storage.

## Durability

More overhead buys more durability, but erasure coding also wins at equal overhead. Take a per-node annual failure rate of 0.81%:

- **(k+m) erasure coding** reaches around **eleven nines** (99.999999999%).
- **3-copy replication**, same node failure rate, reaches around **six nines** (99.9999%).

The reason is combinatorial: replication fails when 3 specific copies die; a (4+2) scheme fails only when 3 of 6 spread chunks die, and there are far more ways to survive.

## The cost is compute and repair traffic

Erasure coding is not free. A write now computes parity, a read of a missing chunk triggers a reconstruction that pulls `k` chunks across the network, and a failed disk means recomputing its chunks from `k` others — more CPU and far more repair bandwidth than copying a replica. So systems tend to replicate small, hot, latency-sensitive objects and erasure-code the large, cold bulk — which is most of the bytes in an [object store](/citadel/interview/storage-systems), and why the advertised durability numbers are achievable at all.
