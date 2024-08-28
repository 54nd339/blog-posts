---
title: Point Location in O(log n) - Persistent Sweep-Line
description: Answering "which region of a planar subdivision contains this query point" after O(n log n) preprocessing, by sweeping a vertical line and freezing a persistent ordered set of edges at every x where it changes.
date: 2024-08-28
draft: false
slug: /cp/point-location
tags:
  - Competitive Programming
  - Geometry
  - Sweep Line
---

Given a planar subdivision — the plane cut into regions by non-crossing segments — and a stream of query points, report which region each point falls in, in $O(\log n)$ per query. The classic solution is a **persistent** sweep line: as a vertical line moves left to right, the set of edges it crosses (ordered by height) changes only at $O(n)$ special $x$-values; store a snapshot of that ordered set at each, and a query becomes "binary search the $x$, then binary search the $y$".

## The problem

Preprocess $n$ segments (forming a subdivision, or an [arrangement](/citadel/cp/planar-graph-faces)) so that, for a query point $Q$, you can name the region (face) containing it — or the segment above/below it — in $O(\log n)$.

## The idea: slabs

Drop a vertical line through every segment endpoint. Between two consecutive lines is a **slab** in which no segment starts or ends, so the segments crossing the slab have a fixed top-to-bottom order. There are $O(n)$ slabs; within a slab, the query point's region is found by binary-searching its $y$ against the ordered segments.

Storing each slab's ordered list independently is $O(n^2)$ space. The fix: consecutive slabs differ by only $O(1)$ edges each (a few segments end, a few begin at that vertical line). Use a **persistent balanced BST**: process events left to right, applying each slab's $O(1)$ insertions/deletions to a persistent structure and keeping the root pointer for every slab. Total space $O(n \log n)$.

**Query $(x, y)$:** binary-search $x$ to find the slab (its persistent root), then descend that BST comparing $y$ against each edge via an [orientation test](/citadel/cp/basic-geometry) at $x$. The edge just below $Q$ identifies the face (with a precomputed "face below each edge in each slab" label).

## Algorithm outline

```
1. Collect all endpoint x-coordinates; sort and unique -> slab boundaries x_0 < x_1 < ... < x_k.
2. events[i] = segments whose left endpoint is at x_i (to insert)
             + segments whose right endpoint is at x_i (to remove).
3. persistent_bst root = empty
   for i in 0..k:
       apply removals then insertions for x_i to `root` (persistently);
       slab_root[i] = root                      # order valid for the slab (x_i, x_{i+1})
   The BST comparator orders two segments by their y at the slab's midpoint
   (compare exactly via cross products, not a computed y).
4. Query(qx, qy):
       i = upper_bound(slab_x, qx) - 1          # which slab
       walk slab_root[i]: at each node with segment s,
           go right if qy is above s at x=qx (orient test), else left;
       the last "segment below" is the answer edge -> its stored face label.
```

## Alternatives

- **Trapezoidal map / randomized incremental** (Seidel): insert segments in random order, maintaining a trapezoidal decomposition and a search DAG. Expected $O(n \log n)$ build, $O(\log n)$ query, $O(n)$ space — the method most libraries use. More code than the slab method but better constants and no persistence.
- **Kirkpatrick's hierarchy:** triangulate, then repeatedly remove an independent set of low-degree vertices and re-triangulate, forming $O(\log n)$ levels. $O(n)$ space, $O(\log n)$ query, but heavy to implement.
- **For convex subdivisions or a single convex polygon:** just the $O(\log n)$ angular binary search from [point in polygon](/citadel/cp/point-in-polygon) — no preprocessing structure needed.
- **Offline** (all queries known): sort queries by $x$ and sweep once with an ordinary balanced BST — $O((n + q)\log n)$, far simpler, and usually enough in contests.

## Complexity

- **Build:** $O(n \log n)$ time, $O(n \log n)$ space (persistent BST) or $O(n)$ (trapezoidal map).
- **Query:** $O(\log n)$ — one $x$ binary search, one $y$ search down the frozen structure.

## Common pitfalls

- **Exact comparator.** The BST orders segments by $y$ at an $x$ inside the slab; compute that comparison with cross products on integer coordinates, never a floating $y$ — a mis-order silently returns the wrong face.
- **Slab midpoint vs endpoints.** Compare segments at the slab's *interior*; at a boundary $x$ two segments may share a $y$ (they meet at an endpoint) and the order is ambiguous.
- **Vertical segments.** They live entirely on a slab boundary. Handle them as a special case (or perturb), or the sweep order is undefined.
- **Query on a boundary.** A point exactly on an edge or vertex — decide the tie policy up front.
- **Overkill.** If the queries are offline, the one-pass sweep with a plain `SortedList` is dramatically simpler and fast enough. Reach for persistence only when queries are online and $n, q$ are large.

## The keystone

Vertical lines through the endpoints cut the plane into $O(n)$ slabs, each with a fixed top-to-bottom edge order; a persistent balanced BST stores all those orders in $O(n \log n)$ space by sharing the $O(1)$ changes between neighbouring slabs. A query is then "binary-search the slab, binary-search the height" — $O(\log n)$. Offline, a single sweep replaces the persistence.
