---
title: Min Cost to Connect All Points - Minimum Spanning Tree on a Complete Graph
description: The cheapest set of edges connecting all points with Manhattan-distance weights — a minimum spanning tree, built by Prim growing a frontier or Kruskal sorting every edge.
date: 2022-02-12
draft: false
slug: /dsa/min-cost-to-connect-all-points
tags:
  - Competitive Programming
  - Graphs
  - Minimum Spanning Tree
---

Every pair of points is an edge weighted by Manhattan distance. Connecting all points at minimum total cost is exactly a minimum spanning tree. Prim grows one tree outward, always adding the cheapest edge to a new point; Kruskal sorts all edges and adds any that does not form a cycle.

## Description

Given `points` on a 2-D plane, the cost to connect two points is the Manhattan distance $|x_1 - x_2| + |y_1 - y_2|$. Return the minimum cost to connect all points so that there is exactly one path between any two.

**Example**

```
Input:  points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
Output: 20
```

**Constraints**

- $1 \le \text{points.length} \le 1000$
- $-10^6 \le x_i, y_i \le 10^6$; all points distinct.

## Prerequisites

- Minimum spanning tree; Prim's and Kruskal's algorithms.
- A min-heap (Prim) or Union-Find (Kruskal).

## Approach 1: Prim's algorithm

### Intuition

Start from point 0. Keep a min-heap of `(cost, point)` for edges crossing from the tree to the outside. Repeatedly pop the cheapest edge to an unvisited point, add its cost, mark it visited, and push its edges to all still-unvisited points.

### Algorithm

1. `visited = set()`, `heap = [(0, 0)]`, `total = 0`.
2. While `len(visited) < n`: pop `(cost, i)`. If `i` already visited, continue. Add `cost` to `total`, mark `i`. For every `j` not visited, push `(manhattan(i, j), j)`.
3. Return `total`.

```python
import heapq

def minCostConnectPoints(points: list[list[int]]) -> int:
    n = len(points)
    visited = set()
    heap = [(0, 0)]
    total = 0

    while len(visited) < n:
        cost, i = heapq.heappop(heap)
        if i in visited:
            continue
        visited.add(i)
        total += cost
        xi, yi = points[i]
        for j in range(n):
            if j not in visited:
                xj, yj = points[j]
                heapq.heappush(heap, (abs(xi - xj) + abs(yi - yj), j))

    return total
```

### Complexity

- **Time:** $O(n^2 \log n)$ — $n^2$ edges pushed, each heap op logarithmic.
- **Space:** $O(n^2)$ for the heap in the worst case.

## Approach 2: Prim's algorithm with a distance array

### Intuition

Avoid the heap: keep `min_dist[j]` = cheapest known edge from the tree to `j`. Each round, pick the unvisited `j` with the smallest `min_dist`, add it, and relax `min_dist[k]` for all unvisited `k`. This is the $O(n^2)$ dense-graph Prim.

### Algorithm

1. `min_dist = [inf] * n`, `min_dist[0] = 0`, `in_tree = [False] * n`, `total = 0`.
2. Repeat `n` times: pick unvisited `u` with smallest `min_dist[u]`; `in_tree[u] = True`; `total += min_dist[u]`; for each unvisited `v`, `min_dist[v] = min(min_dist[v], manhattan(u, v))`.
3. Return `total`.

```python
def minCostConnectPoints(points: list[list[int]]) -> int:
    n = len(points)
    min_dist = [float("inf")] * n
    min_dist[0] = 0
    in_tree = [False] * n
    total = 0

    for _ in range(n):
        u = min((i for i in range(n) if not in_tree[i]), key=lambda i: min_dist[i])
        in_tree[u] = True
        total += min_dist[u]
        xu, yu = points[u]
        for v in range(n):
            if not in_tree[v]:
                xv, yv = points[v]
                min_dist[v] = min(min_dist[v], abs(xu - xv) + abs(yu - yv))

    return total
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n)$.

## Approach 3: Kruskal's algorithm

### Intuition

Generate all $\binom{n}{2}$ edges, sort by weight, and add each edge whose endpoints are in different components (Union-Find). Stop after `n - 1` edges.

### Algorithm

1. Build `edges = [(dist, i, j)]` for all `i < j`; sort.
2. DSU over `n`. For each edge in order: if `find(i) != find(j)`, union and add `dist` to `total`, `used += 1`. Stop when `used == n - 1`.

```python
def minCostConnectPoints(points: list[list[int]]) -> int:
    n = len(points)
    edges = []
    for i in range(n):
        xi, yi = points[i]
        for j in range(i + 1, n):
            xj, yj = points[j]
            edges.append((abs(xi - xj) + abs(yi - yj), i, j))
    edges.sort()

    parent = list(range(n))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    total = used = 0
    for w, i, j in edges:
        ri, rj = find(i), find(j)
        if ri != rj:
            parent[ri] = rj
            total += w
            used += 1
            if used == n - 1:
                break
    return total
```

### Complexity

- **Time:** $O(n^2 \log n)$ — dominated by sorting $n^2$ edges.
- **Space:** $O(n^2)$ for the edge list.

## Common Pitfalls

- **Skipping the "already visited" check when popping in Prim.** Stale heap entries for now-visited points must be discarded, or you add an edge twice.
- **Adding `n` edges instead of `n - 1`.** A spanning tree on `n` nodes has exactly `n - 1` edges; the first Prim pop contributes cost 0.
- **Recomputing distances with floats.** Manhattan distance is integer; keep it exact.
- **Kruskal on the full edge list for large `n`.** At `n = 1000` that is ~500k edges — fine, but the $O(n^2)$ array-Prim is lighter on memory.

## The keystone

"Connect everything as cheaply as possible" is a minimum spanning tree, and the two classic builds only differ in vantage point: Prim grows a single blob, Kruskal merges many. Kruskal is [Redundant Connection](/citadel/dsa/redundant-connection)'s cycle test applied to sorted edges.
