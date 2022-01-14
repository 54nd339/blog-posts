---
title: Advanced Graphs - Weights, Spanning Trees, and Eulerian Trails
description: A guide to the Advanced Graphs section of NeetCode 150 — Dijkstra for weighted shortest paths, Prim and Kruskal for spanning trees, Bellman-Ford for hop budgets, and Hierholzer for trails using every edge.
date: 2022-01-14
draft: false
slug: /dsa/advanced-graphs
tags:
  - Competitive Programming
  - Graphs
  - Shortest Path
---

[Graphs](/citadel/dsa/graphs) covered unweighted traversal. This section adds edge weights and asks harder questions: cheapest path, cheapest way to connect everything, a trail that uses every edge. Four named algorithms cover all six problems.

## The algorithms

- **Dijkstra** — a min-heap of `(distance, node)`; pop the closest unfinalized node, relax its edges. Non-negative weights only. Shortest path from one source.
- **Bellman-Ford** — relax every edge `V - 1` times. Slower, but handles negative weights and, crucially, lets you *bound the number of edges* by capping the rounds.
- **Prim / Kruskal** — minimum spanning tree. Prim grows one tree with a heap; Kruskal sorts all edges and adds any that does not close a cycle (union-find).
- **Hierholzer** — Eulerian path/circuit: walk until stuck, unwind while prepending, for a trail using every edge exactly once.

## Problem → algorithm

- [Network Delay Time](/citadel/dsa/network-delay-time) — Dijkstra from the source, answer is the max finalized distance.
- [Cheapest Flights Within K Stops](/citadel/dsa/cheapest-flights-within-k-stops) — Bellman-Ford with `k + 1` rounds from a snapshot; the round count *is* the stop budget.
- [Min Cost to Connect All Points](/citadel/dsa/min-cost-to-connect-all-points) — MST on a complete graph of Manhattan distances; Prim ($O(n^2)$) or Kruskal.
- [Swim in Rising Water](/citadel/dsa/swim-in-rising-water) — Dijkstra with `max` instead of `+` (minimise the path's largest cell); also binary-search-on-answer or Kruskal.
- [Reconstruct Itinerary](/citadel/dsa/reconstruct-itinerary) — Eulerian path; Hierholzer with lexical tie-breaking.
- [Alien Dictionary](/citadel/dsa/alien-dictionary) — not weighted at all; topological sort on edges extracted from adjacent word pairs (carried over from [Graphs](/citadel/dsa/graphs)).

## Dijkstra, the template

```python
import heapq

def dijkstra(adj, src, n):
    dist = {}
    heap = [(0, src)]
    while heap:
        d, u = heapq.heappop(heap)
        if u in dist:
            continue
        dist[u] = d
        for v, w in adj[u]:
            if v not in dist:
                heapq.heappush(heap, (d + w, v))
    return dist
```

Swap the relaxation (`d + w` → `max(d, w)`), add a dimension (`stops`), or change the pop target and you have most of this section.

## Recognising it

- "shortest / cheapest path", non-negative weights → Dijkstra.
- "... using at most K edges/stops", or negative weights → Bellman-Ford.
- "connect all nodes at minimum total cost" → MST (Prim/Kruskal).
- "minimise the maximum edge on a path" → Dijkstra with `max`, or Kruskal until connected.
- "use every edge/ticket exactly once" → Eulerian path, Hierholzer.

## Where this goes next

The recurring move here — a state is *position plus a budget or constraint*, and you relax over that expanded state — is exactly the setup for [1-D Dynamic Programming](/citadel/dsa/1-d-dynamic-programming), where the graph is implicit and the "edges" are transitions between subproblems.
