---
title: Data Structures for Competitive Programming - Range Queries and DSU
description: A map of the contest data-structure toolkit — the range-query ladder from prefix sums to segment trees, disjoint set union for connectivity, and the sqrt-family fallbacks when nothing else fits.
date: 2024-02-22
draft: false
slug: /cp/data-structures
tags:
  - Competitive Programming
  - Data Structures
  - Range Queries
---

Two questions drive almost every contest data structure. First: "aggregate this range of the array, and let me change elements too." Second: "are these two things connected, as I keep adding links." The first is a ladder of structures trading query power for update cost; the second is [disjoint set union](/citadel/cp/disjoint-set-union), which is nearly free.

## The range-query ladder

| structure | query | point update | range update | notes |
| --- | --- | --- | --- | --- |
| prefix sums | $O(1)$ | $O(n)$ | — | static only |
| [sqrt decomposition](/citadel/cp/sqrt-decomposition) | $O(\sqrt n)$ | $O(1)$ | $O(\sqrt n)$ | easiest to code for weird queries |
| [Fenwick tree](/citadel/cp/fenwick-tree) | $O(\log n)$ | $O(\log n)$ | $O(\log n)$† | tiny, fast, invertible ops only |
| [segment tree](/citadel/data-structures/segment-tree) | $O(\log n)$ | $O(\log n)$ | $O(\log n)$‡ | any associative op; lazy for range update |
| [sparse table](/citadel/cp/sparse-table) | $O(1)$ | — | — | static, idempotent op (min/gcd) |
| [sqrt tree](/citadel/cp/sqrt-tree) | $O(1)$ | $O(\sqrt n)$ | — | associative op, $O(1)$ query, rare |

† Fenwick does range-update/point-query with a difference array, or range-update/range-query with two Fenwicks. ‡ Segment tree needs lazy propagation for range updates.

**How to choose.** Static min/gcd/or → sparse table. Sums with point updates → Fenwick (smallest code). Anything with range assignment, range add, or a non-invertible merge (max, "count of X") → segment tree with lazy propagation. Query so exotic that no clean merge exists → sqrt decomposition and just recompute blocks.

## Connectivity: disjoint set union

[DSU](/citadel/cp/disjoint-set-union) answers "same component?" and "how many components?" under a stream of `union` operations, in near-$O(1)$ amortised time with union-by-size plus path compression. It is the backbone of [Kruskal's MST](/citadel/algorithms/MinimumSpanningTree), offline dynamic-connectivity, and [2-edge-connectivity](/citadel/cp/finding-bridges-online). It does not support deletion — for that you need the ["rollback to a checkpoint"](/citadel/cp/deleting-in-log-n) trick or link-cut trees.

## The amortised-extremes pair

[Minimum stack and minimum queue](/citadel/cp/min-stack-min-queue) keep the running min/max of a stack or queue in $O(1)$ per operation. The queue version (two min-stacks, or a monotone deque) is the engine behind sliding-window minimum and several DP optimisations.

## Balanced BSTs when order matters

When you need order statistics, split/merge by key, or an implicit sequence with range operations, a [treap](/citadel/cp/treap) (or its cousin the implicit treap) gives $O(\log n)$ expected for all of it with far less code than a red-black tree.

## Recognising the pattern

- "sum/min/max of a subarray" + updates → range-query ladder; pick by update type.
- "are $u$ and $v$ connected" as edges arrive → DSU.
- "minimum of every window of width $k$" → monotone deque (min-queue).
- "k-th smallest" / "insert, erase, rank" → order-statistics tree or treap.
- offline queries you can reorder → sort them, then a simpler structure often suffices (see [Mo's algorithm on sqrt decomposition](/citadel/cp/sqrt-decomposition)).

## Where this goes next

The segment tree deepens into lazy propagation, persistence, and 2-D — covered in the [dedicated segment tree post](/citadel/data-structures/segment-tree). DSU reappears throughout [Graphs](/citadel/cp/graphs), and the sqrt idea generalises from arrays to trees ([centroid decomposition](/citadel/cp/centroid-decomposition)) and to query batching (Mo's).
