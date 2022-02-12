---
title: Network Delay Time - Dijkstra to the Farthest Node
description: Time for a signal from one node to reach all others on a weighted directed graph — single-source shortest paths by Dijkstra, then the maximum finishing distance, or unreachable.
date: 2022-02-12
draft: false
slug: /dsa/network-delay-time
tags:
  - Competitive Programming
  - Graphs
  - Shortest Path
---

A signal reaches every node along its shortest path, so run Dijkstra from `k`. The answer is the largest of those shortest-path distances — the last node to hear the signal. If any node is unreachable, return `-1`.

## Description

`times[i] = [u, v, w]` is a directed edge: a signal takes `w` time from `u` to `v`. Given `n` nodes labelled `1..n` and a source `k`, return the time for all nodes to receive a signal sent from `k`, or `-1` if some node never does.

**Example**

```
Input:  times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
Output: 2

Input:  times = [[1,2,1]], n = 2, k = 2
Output: -1
```

**Constraints**

- $1 \le k \le n \le 100$
- $1 \le \text{times.length} \le 6000$; $0 \le w \le 100$; edges distinct.

## Prerequisites

- Dijkstra's algorithm with a min-heap (non-negative weights).
- Bellman-Ford as an alternative (and its `V-1` relaxation bound).

## Approach 1: Dijkstra

### Intuition

Pop the closest unfinalized node, finalize its distance, relax its outgoing edges. Non-negative weights guarantee that the first time a node is popped, its distance is final.

### Algorithm

1. Build `adj[u] = [(v, w)]`.
2. `dist = {}`, `heap = [(0, k)]`.
3. While `heap`: pop `(d, u)`. If `u in dist`, skip. Set `dist[u] = d`. For each `(v, w)` in `adj[u]` with `v not in dist`, push `(d + w, v)`.
4. If `len(dist) < n`, return `-1`; else return `max(dist.values())`.

```python
import heapq
from collections import defaultdict

def networkDelayTime(times: list[list[int]], n: int, k: int) -> int:
    adj = defaultdict(list)
    for u, v, w in times:
        adj[u].append((v, w))

    dist = {}
    heap = [(0, k)]
    while heap:
        d, u = heapq.heappop(heap)
        if u in dist:
            continue
        dist[u] = d
        for v, w in adj[u]:
            if v not in dist:
                heapq.heappush(heap, (d + w, v))

    return max(dist.values()) if len(dist) == n else -1
```

### Complexity

- **Time:** $O(E \log V)$.
- **Space:** $O(V + E)$.

## Approach 2: Bellman-Ford

### Intuition

Relax every edge `n - 1` times. After pass `i`, all shortest paths using at most `i` edges are correct; a shortest path has at most `n - 1` edges.

### Algorithm

1. `dist = [inf] * (n + 1)`, `dist[k] = 0`.
2. Repeat `n - 1` times: for each `(u, v, w)`, if `dist[u] + w < dist[v]`, set `dist[v] = dist[u] + w`.
3. `ans = max(dist[1..n])`; return `-1` if it is `inf`.

```python
def networkDelayTime(times: list[list[int]], n: int, k: int) -> int:
    dist = [float("inf")] * (n + 1)
    dist[k] = 0
    for _ in range(n - 1):
        changed = False
        for u, v, w in times:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                changed = True
        if not changed:
            break
    ans = max(dist[1:])
    return ans if ans != float("inf") else -1
```

### Complexity

- **Time:** $O(V \cdot E)$.
- **Space:** $O(V)$.

## Approach 3: Floyd-Warshall

### Intuition

All-pairs shortest paths by dynamic programming over "allowed intermediate nodes". Overkill for a single source, but trivial to write and fine at `n = 100`.

### Algorithm

1. `d[i][j] = inf`, `d[i][i] = 0`, `d[u][v] = w` for each edge.
2. For each `mid`, `i`, `j`: `d[i][j] = min(d[i][j], d[i][mid] + d[mid][j])`.
3. Answer is `max(d[k][j])` over `j`, or `-1` if infinite.

```python
def networkDelayTime(times: list[list[int]], n: int, k: int) -> int:
    INF = float("inf")
    d = [[INF] * (n + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        d[i][i] = 0
    for u, v, w in times:
        d[u][v] = min(d[u][v], w)

    for mid in range(1, n + 1):
        for i in range(1, n + 1):
            for j in range(1, n + 1):
                if d[i][mid] + d[mid][j] < d[i][j]:
                    d[i][j] = d[i][mid] + d[mid][j]

    ans = max(d[k][1:])
    return ans if ans != INF else -1
```

### Complexity

- **Time:** $O(V^3)$.
- **Space:** $O(V^2)$.

## Common Pitfalls

- **Re-finalizing a popped node in Dijkstra.** Guard with `if u in dist: continue`; stale heap entries are normal.
- **1- vs 0-indexed nodes.** Labels are `1..n`; size arrays `n + 1` and ignore index 0.
- **Returning a distance when a node is unreachable.** Check that *all* `n` nodes were reached before taking the max.
- **Bellman-Ford with too few passes.** You need `n - 1` relaxation rounds (early-exit on a clean pass is a valid optimisation).

## The keystone

"Time for a broadcast to reach everyone" is single-source shortest paths plus a max. Dijkstra is the tool whenever weights are non-negative; [Cheapest Flights Within K Stops](/citadel/dsa/cheapest-flights-within-k-stops) is what happens when you also bound the number of edges, which pulls you back toward Bellman-Ford.
