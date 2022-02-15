---
title: Swim in Rising Water - Minimise the Maximum Cell on a Path
description: The earliest time you can cross a grid where cell heights are entry times — a shortest-path variant that minimises the largest value on the route, by Dijkstra, binary search, or Kruskal.
date: 2022-02-15
draft: false
slug: /dsa/swim-in-rising-water
tags:
  - Competitive Programming
  - Graphs
  - Shortest Path
---

You can enter cell `(r, c)` at time `grid[r][c]`, and you want to reach the bottom-right corner as early as possible. The cost of a path is its *maximum* cell, not its sum. Dijkstra works if you relax with `max(dist, neighbour_height)` instead of a sum; binary search and union-find also fit.

## Description

Given an `n x n` grid where `grid[r][c]` is the elevation, water level rises by 1 each time step. You can swim from a cell to a 4-adjacent one if both elevations are at most the current time. Starting at `(0, 0)`, return the least time to reach `(n-1, n-1)`.

**Example**

```
Input:  grid = [[0,2],[1,3]]
Output: 3

Input:  grid = [[0,1,2,3,4],[24,23,22,21,5],[12,13,14,15,16],[11,17,18,19,20],[10,9,8,7,6]]
Output: 16
```

**Constraints**

- $1 \le n \le 50$
- `grid` is a permutation of `0 .. n*n - 1`.

## Prerequisites

- Dijkstra generalised to a "minimise the maximum edge" objective.
- Binary search on the answer; Union-Find for incremental connectivity.

## Approach 1: Dijkstra (minimise the path maximum)

### Intuition

Keep `time[cell]` = the smallest possible path-maximum to reach it. Pop the cell with the smallest such value; relax a neighbour with `max(current_time, grid[neighbour])`. First pop of the target is the answer.

### Algorithm

1. `heap = [(grid[0][0], 0, 0)]`, `visited = set()`.
2. Pop `(t, r, c)`. If `(r, c)` is the target, return `t`. If visited, skip; mark it. For each in-bounds neighbour not visited, push `(max(t, grid[nr][nc]), nr, nc)`.

```python
import heapq

def swimInWater(grid: list[list[int]]) -> int:
    n = len(grid)
    heap = [(grid[0][0], 0, 0)]
    visited = set()

    while heap:
        t, r, c = heapq.heappop(heap)
        if (r, c) == (n - 1, n - 1):
            return t
        if (r, c) in visited:
            continue
        visited.add((r, c))
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < n and 0 <= nc < n and (nr, nc) not in visited:
                heapq.heappush(heap, (max(t, grid[nr][nc]), nr, nc))

    return -1
```

### Complexity

- **Time:** $O(n^2 \log n)$.
- **Space:** $O(n^2)$.

## Approach 2: Binary search + DFS

### Intuition

For a fixed time `T`, a simple DFS/BFS over cells with `grid <= T` tells you whether the target is reachable. Reachability is monotonic in `T`, so binary search the smallest feasible `T`.

### Algorithm

1. `lo = grid[0][0]`, `hi = n*n - 1`.
2. While `lo < hi`: `mid = (lo + hi) // 2`; if `reachable(mid)`, `hi = mid`, else `lo = mid + 1`.
3. `reachable(T)`: DFS from `(0,0)` through cells `<= T`; return whether `(n-1, n-1)` is hit.

```python
def swimInWater(grid: list[list[int]]) -> int:
    n = len(grid)

    def reachable(T):
        if grid[0][0] > T:
            return False
        seen = {(0, 0)}
        stack = [(0, 0)]
        while stack:
            r, c = stack.pop()
            if (r, c) == (n - 1, n - 1):
                return True
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if 0 <= nr < n and 0 <= nc < n and (nr, nc) not in seen and grid[nr][nc] <= T:
                    seen.add((nr, nc))
                    stack.append((nr, nc))
        return False

    lo, hi = grid[0][0], n * n - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if reachable(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Complexity

- **Time:** $O(n^2 \log n)$ — $\log(n^2)$ searches, each an $O(n^2)$ flood.
- **Space:** $O(n^2)$.

## Approach 3: Kruskal / Union-Find

### Intuition

Add cells in increasing elevation order. When you add cell with value `t`, union it with any already-added neighbour. The answer is the value `t` at which `(0,0)` and `(n-1,n-1)` first share a component.

### Algorithm

1. Sort cells by elevation.
2. For `t` from 0 upward, "activate" the cell whose value is `t`; union with active neighbours.
3. After each activation, if `find(0) == find(n*n - 1)`, return `t`.

```python
def swimInWater(grid: list[list[int]]) -> int:
    n = len(grid)
    pos = {grid[r][c]: (r, c) for r in range(n) for c in range(n)}
    parent = list(range(n * n))
    active = [False] * (n * n)

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(a, b):
        parent[find(a)] = find(b)

    for t in range(n * n):
        r, c = pos[t]
        active[r * n + c] = True
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < n and 0 <= nc < n and active[nr * n + nc]:
                union(r * n + c, nr * n + nc)
        if find(0) == find(n * n - 1):
            return t
    return n * n - 1
```

### Complexity

- **Time:** $O(n^2 \alpha(n^2))$.
- **Space:** $O(n^2)$.

## Common Pitfalls

- **Relaxing with a sum in Dijkstra.** The path cost is `max`, not `+`; use `max(t, grid[nr][nc])`.
- **Binary search bounds.** `lo` must start at `grid[0][0]` (you cannot leave earlier than that), `hi` at `n*n - 1`.
- **Union-Find: unioning with inactive neighbours.** Only merge with cells already activated (elevation `<= t`).
- **Returning on enqueue vs on pop in Dijkstra.** Return when the target is *popped*, so its recorded time is final.

## The keystone

"Minimise the worst step on a path" is a shortest-path skeleton with `max` swapped in for `+` — Dijkstra, binary-search-on-answer, and Kruskal all adapt. The monotonic-reachability insight is the same one that powers [Koko Eating Bananas](/citadel/dsa/koko-eating-bananas).
