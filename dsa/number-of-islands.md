---
title: Number of Islands - Flood Fill Every Unvisited Landmass
description: Counting connected groups of land cells in a grid — scan for an unvisited 1, flood the whole island so it is not counted again, and increment once per flood.
date: 2021-12-09
draft: false
slug: /dsa/number-of-islands
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

The grid is a graph: each land cell is a vertex, edges join orthogonal neighbours. An island is a connected component. Walk the grid; the first time you touch a new component, count it and erase the whole thing so you never re-enter it.

## Description

Given an `m x n` binary grid where `'1'` is land and `'0'` is water, return the number of islands. An island is land connected 4-directionally and surrounded by water or the grid edge.

**Example**

```
Input:  grid = [["1","1","0","0"],
                 ["1","0","0","1"],
                 ["0","0","1","1"]]
Output: 2
```

**Constraints**

- $1 \le m, n \le 300$
- Each cell is `'0'` or `'1'`.

## Prerequisites

- Grid traversal with the four orthogonal offsets.
- DFS / BFS on an implicit graph; marking visited to avoid revisits.

## Approach 1: Depth-First Search

### Intuition

Loop over every cell. On an unvisited `'1'`, run a DFS that sinks every reachable land cell (set it to `'0'` or record it visited), then add one to the count.

### Algorithm

1. `count = 0`.
2. For each `(r, c)`: if `grid[r][c] == '1'`, call `dfs(r, c)` and `count += 1`.
3. `dfs(r, c)`: return on out-of-bounds or `grid[r][c] != '1'`; set `grid[r][c] = '0'`; recurse into the four neighbours.

```python
def numIslands(grid: list[list[str]]) -> int:
    rows, cols = len(grid), len(grid[0])

    def dfs(r, c):
        if r < 0 or c < 0 or r >= rows or c >= cols or grid[r][c] != "1":
            return
        grid[r][c] = "0"
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                dfs(r, c)
                count += 1
    return count
```

### Complexity

- **Time:** $O(m \cdot n)$ — each cell is visited once.
- **Space:** $O(m \cdot n)$ worst case for the recursion stack (a grid that is one snaking island).

## Approach 2: Breadth-First Search

### Intuition

Same flood, iterative. On an unvisited `'1'`, push it to a queue, and drain the queue, sinking each cell and enqueuing its land neighbours.

### Algorithm

1. For each `(r, c)` with `grid[r][c] == '1'`: `count += 1`, sink it, enqueue.
2. While the queue is non-empty: pop `(r, c)`, for each neighbour that is `'1'`, sink it and enqueue.

```python
from collections import deque

def numIslands(grid: list[list[str]]) -> int:
    rows, cols = len(grid), len(grid[0])
    count = 0

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] != "1":
                continue
            count += 1
            grid[r][c] = "0"
            q = deque([(r, c)])
            while q:
                cr, cc = q.popleft()
                for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                    nr, nc = cr + dr, cc + dc
                    if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == "1":
                        grid[nr][nc] = "0"
                        q.append((nr, nc))
    return count
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(\min(m, n))$ for the queue (the frontier of a BFS on a grid).

## Approach 3: Union-Find

### Intuition

Give every land cell an id. Union each land cell with its right and down land neighbours. The answer is the number of distinct roots among land cells.

### Algorithm

1. Initialise DSU over `rows * cols`, `components` = number of land cells.
2. For each land cell, union with its right and down neighbour if that is land; decrement `components` on each successful union.
3. Return `components`.

```python
def numIslands(grid: list[list[str]]) -> int:
    rows, cols = len(grid), len(grid[0])
    parent = list(range(rows * cols))
    rank = [0] * (rows * cols)
    components = 0

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(a, b):
        nonlocal components
        ra, rb = find(a), find(b)
        if ra == rb:
            return
        if rank[ra] < rank[rb]:
            ra, rb = rb, ra
        parent[rb] = ra
        if rank[ra] == rank[rb]:
            rank[ra] += 1
        components -= 1

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] != "1":
                continue
            components += 1
            if r + 1 < rows and grid[r + 1][c] == "1":
                union(r * cols + c, (r + 1) * cols + c)
            if c + 1 < cols and grid[r][c + 1] == "1":
                union(r * cols + c, r * cols + c + 1)
    return components
```

### Complexity

- **Time:** $O(m \cdot n \cdot \alpha(mn))$ — near linear.
- **Space:** $O(m \cdot n)$ for the parent and rank arrays.

## Common Pitfalls

- **Not marking a cell visited before enqueuing.** In BFS, sink on *enqueue*, not on dequeue, or the same cell enters the queue from several neighbours and you overcount work (and risk a huge queue).
- **Counting inside the DFS.** Increment once per *call site* in the scan loop, not once per recursive call.
- **Diagonal moves.** Islands connect 4-directionally only; adding the four diagonals merges islands that should stay separate.
- **Mutating the input when you must not.** If the grid has to survive, use a separate `visited` set instead of overwriting `'1'` with `'0'`.

## The keystone

Counting connected components is the "hello world" of graph traversal: one outer scan for un-visited seeds, one flood per seed. [Max Area of Island](/citadel/dsa/max-area-of-island) is the same scan where the flood returns a size instead of nothing.
