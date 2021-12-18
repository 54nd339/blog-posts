---
title: Max Area of Island - Flood Fill That Returns a Count
description: The largest connected group of land cells in a binary grid — the island-counting scan where each flood returns how many cells it sank, and you keep the maximum.
date: 2021-12-18
draft: false
slug: /dsa/max-area-of-island
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

Identical to [Number of Islands](/citadel/dsa/number-of-islands) except the flood reports its size. Scan for an unvisited land cell, flood it, take the returned area, keep the running max.

## Description

Given an `m x n` binary grid, return the area of the largest island (number of `1` cells connected 4-directionally). If there is no land, return `0`.

**Example**

```
Input:  grid = [[0,0,1,0,0],
                 [0,1,1,1,0],
                 [0,0,1,0,1]]
Output: 5
```

**Constraints**

- $1 \le m, n \le 50$
- Each cell is `0` or `1`.

## Prerequisites

- Grid DFS / BFS with visited marking.
- A flood that accumulates and returns a value.

## Approach 1: Depth-First Search

### Intuition

`dfs(r, c)` returns `0` for water / out-of-bounds / visited, otherwise `1` plus the sum of the four recursive calls. Mark the cell before recursing.

### Algorithm

1. For each cell, `best = max(best, dfs(r, c))`.
2. `dfs`: bounds / non-land check returns `0`; set `grid[r][c] = 0`; return `1 + dfs(up) + dfs(down) + dfs(left) + dfs(right)`.

```python
def maxAreaOfIsland(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])

    def dfs(r, c):
        if r < 0 or c < 0 or r >= rows or c >= cols or grid[r][c] == 0:
            return 0
        grid[r][c] = 0
        return 1 + dfs(r + 1, c) + dfs(r - 1, c) + dfs(r, c + 1) + dfs(r, c - 1)

    best = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                best = max(best, dfs(r, c))
    return best
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$ recursion worst case.

## Approach 2: Breadth-First Search

### Intuition

Flood with a queue, incrementing `area` for each cell dequeued (or enqueued). Return the max over all seeds.

### Algorithm

1. On an unvisited land cell: `area = 0`, sink and enqueue it.
2. Drain the queue: each pop does `area += 1` and enqueues land neighbours (sinking them on enqueue).
3. `best = max(best, area)`.

```python
from collections import deque

def maxAreaOfIsland(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    best = 0

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] != 1:
                continue
            grid[r][c] = 0
            q = deque([(r, c)])
            area = 0
            while q:
                cr, cc = q.popleft()
                area += 1
                for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                    nr, nc = cr + dr, cc + dc
                    if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                        grid[nr][nc] = 0
                        q.append((nr, nc))
            best = max(best, area)
    return best
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(\min(m, n))$ queue.

## Approach 3: Union-Find

### Intuition

Union adjacent land cells; track a `size` per root. The answer is the largest root size.

### Algorithm

1. DSU with `size[root]` initialised to `1` for land cells.
2. Union each land cell with its right / down land neighbour, adding sizes on merge.
3. Return `max(size[find(cell)])` over land cells (or track a running max during unions).

```python
def maxAreaOfIsland(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    parent = list(range(rows * cols))
    size = [1] * (rows * cols)
    best = 0

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(a, b):
        nonlocal best
        ra, rb = find(a), find(b)
        if ra == rb:
            return
        if size[ra] < size[rb]:
            ra, rb = rb, ra
        parent[rb] = ra
        size[ra] += size[rb]
        best = max(best, size[ra])

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] != 1:
                continue
            best = max(best, 1)
            idx = r * cols + c
            if r + 1 < rows and grid[r + 1][c] == 1:
                union(idx, (r + 1) * cols + c)
            if c + 1 < cols and grid[r][c + 1] == 1:
                union(idx, idx + 1)
    return best
```

### Complexity

- **Time:** $O(m \cdot n \cdot \alpha(mn))$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Returning `0` instead of the max for an all-water grid.** Initialise `best = 0` and it falls out naturally; don't seed it with a sentinel.
- **Forgetting the `1 +` in the DFS return.** The current cell counts too.
- **Double counting in BFS.** Sink each cell exactly once — on enqueue — so it contributes `+1` a single time.
- **Union-Find without size tracking.** Plain component count doesn't give area; you must carry `size` on the root.

## The keystone

The connected-components scan generalises by choosing what the flood *produces*: nothing (a count of islands), a size (this problem), a boolean reachability (Pacific/Atlantic). Same traversal, different accumulator. Next, [Clone Graph](/citadel/dsa/clone-graph) runs the same traversal on an explicit adjacency structure.
