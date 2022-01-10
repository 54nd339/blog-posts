---
title: Pacific Atlantic Water Flow - Search Inland From Both Oceans
description: Cells from which rain can reach both oceans on a height grid — instead of searching outward from every cell, flood inward from each ocean border to non-decreasing heights and intersect.
date: 2022-01-10
draft: false
slug: /dsa/pacific-atlantic-water-flow
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

Searching from each cell toward both oceans is `O((mn)^2)`. Reverse it: start at the ocean borders and walk *uphill* (to neighbours of height greater than or equal to the current). A cell reachable in the Pacific flood and the Atlantic flood is one where water flows to both.

## Description

Given an `m x n` matrix `heights`, the Pacific borders the top and left edges, the Atlantic borders the bottom and right edges. Water flows from a cell to an orthogonal neighbour of height less than or equal to it. Return all coordinates from which water can reach both oceans.

**Example**

```
Input:  heights = [[1,2,2,3,5],
                    [3,2,3,4,4],
                    [2,4,5,3,1],
                    [6,7,1,4,5],
                    [5,1,1,2,4]]
Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```

**Constraints**

- $1 \le m, n \le 200$
- $0 \le \text{heights}[i][j] \le 10^5$

## Prerequisites

- Reverse thinking: flood from the destination, not the source.
- Two independent DFS/BFS floods and a set intersection.

## Approach 1: Depth-First Search from the borders

### Intuition

Build two visited sets, `pac` and `atl`. Seed `pac` with the top row and left column, `atl` with the bottom row and right column. From each seed, DFS to any neighbour whose height is greater than or equal to the current cell's (water could flow back down to us). The answer is `pac ∩ atl`.

### Algorithm

1. `pac`, `atl` = empty sets.
2. `dfs(r, c, visited, prev_height)`: return if out-of-bounds, already visited, or `heights[r][c] < prev_height`. Add `(r, c)` to `visited`. Recurse into the four neighbours with `prev_height = heights[r][c]`.
3. For each `c`: `dfs(0, c, pac, ...)` and `dfs(rows-1, c, atl, ...)`. For each `r`: `dfs(r, 0, pac, ...)` and `dfs(r, cols-1, atl, ...)`.
4. Return `[[r, c] for (r, c) in pac & atl]`.

```python
def pacificAtlantic(heights: list[list[int]]) -> list[list[int]]:
    rows, cols = len(heights), len(heights[0])
    pac, atl = set(), set()

    def dfs(r, c, visited, prev):
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or (r, c) in visited or heights[r][c] < prev):
            return
        visited.add((r, c))
        h = heights[r][c]
        dfs(r + 1, c, visited, h)
        dfs(r - 1, c, visited, h)
        dfs(r, c + 1, visited, h)
        dfs(r, c - 1, visited, h)

    for c in range(cols):
        dfs(0, c, pac, heights[0][c])
        dfs(rows - 1, c, atl, heights[rows - 1][c])
    for r in range(rows):
        dfs(r, 0, pac, heights[r][0])
        dfs(r, cols - 1, atl, heights[r][cols - 1])

    return [[r, c] for (r, c) in pac & atl]
```

### Complexity

- **Time:** $O(m \cdot n)$ — each cell is visited at most once per ocean.
- **Space:** $O(m \cdot n)$ for the two sets and recursion.

## Approach 2: Breadth-First Search from the borders

### Intuition

Same reversal, iterative. Seed a queue with all border cells for one ocean; pop a cell and enqueue any unvisited neighbour of height greater than or equal to it. Repeat for the other ocean, then intersect.

### Algorithm

1. `bfs(starts)`: put all `starts` in `visited` and a queue. While the queue is non-empty, pop `(r, c)`; for each neighbour in bounds, unvisited, with `heights[nr][nc] >= heights[r][c]`, mark and enqueue. Return `visited`.
2. `pac = bfs(top row + left column)`, `atl = bfs(bottom row + right column)`.
3. Return `pac & atl` as coordinate pairs.

```python
from collections import deque

def pacificAtlantic(heights: list[list[int]]) -> list[list[int]]:
    rows, cols = len(heights), len(heights[0])

    def bfs(starts):
        visited = set(starts)
        q = deque(starts)
        while q:
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if (0 <= nr < rows and 0 <= nc < cols and (nr, nc) not in visited
                        and heights[nr][nc] >= heights[r][c]):
                    visited.add((nr, nc))
                    q.append((nr, nc))
        return visited

    pac = bfs([(0, c) for c in range(cols)] + [(r, 0) for r in range(rows)])
    atl = bfs([(rows - 1, c) for c in range(cols)] + [(r, cols - 1) for r in range(rows)])
    return [[r, c] for (r, c) in pac & atl]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Searching in the natural direction.** Flooding downhill from every cell is quadratic; flood *uphill* from the oceans once.
- **Wrong comparison.** From the border inward, you move to neighbours with height **≥** the current cell (water flows back the other way). Using `<=` walks the wrong slope.
- **Seeding only two edges.** The Pacific is top **and** left; the Atlantic is bottom **and** right. Corners belong to both.
- **Not deduping seeds.** Corner cells appear in two border lists; put seeds in a set first or the visited set absorbs the duplication anyway.

## The keystone

"Which sources can reach a target" often flips to "which cells does a flood from the target reach", turning `N` searches into one. That reversal, plus intersecting two floods, also underlies [Surrounded Regions](/citadel/dsa/surrounded-regions).
