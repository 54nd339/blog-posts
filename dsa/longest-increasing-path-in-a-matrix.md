---
title: Longest Increasing Path in a Matrix - DFS With a Memo per Cell
description: The longest strictly increasing path through 4-adjacent cells of a grid — DFS from each cell caching its answer, since strictly increasing values make the graph acyclic and each cell final once.
date: 2022-04-24
draft: false
slug: /dsa/longest-increasing-path-in-a-matrix
tags:
  - Competitive Programming
  - Dynamic Programming
  - Graphs
---

Draw an edge from a cell to each strictly larger neighbour. Strictly increasing values mean no cycles — a DAG — so `longest(cell)` is well defined: `1 + max(longest(neighbour))` over larger neighbours. Cache it; every cell is computed once.

## Description

Given an `m x n` integer matrix, return the length of the longest strictly increasing path. You may move up, down, left, or right, not diagonally, and may not revisit a cell.

**Example**

```
Input:  matrix = [[9,9,4],[6,6,8],[2,1,1]]
Output: 4   (1 → 2 → 6 → 9)

Input:  matrix = [[3,4,5],[3,2,6],[2,2,1]]
Output: 4   (3 → 4 → 5 → 6)
```

**Constraints**

- $1 \le m, n \le 200$
- $0 \le \text{matrix}[i][j] \le 2^{31} - 1$

## Prerequisites

- The strictly-increasing constraint makes the neighbour graph a DAG (no explicit visited set needed).
- Memoized DFS, or Kahn's algorithm on the DAG.

## Approach 1: DFS with memoization

### Intuition

`dfs(r, c)` returns the longest increasing path starting at `(r, c)`. For each neighbour with a strictly greater value, recurse; the answer is `1 + max` of those (or `1` if none). Store in `memo[r][c]`.

### Algorithm

1. `memo = {}`.
2. `dfs(r, c)`: return `memo[(r, c)]` if present. `best = 1`; for each in-bounds neighbour with `matrix[nr][nc] > matrix[r][c]`, `best = max(best, 1 + dfs(nr, nc))`. Store and return `best`.
3. Return `max(dfs(r, c) for all cells)`.

```python
def longestIncreasingPath(matrix: list[list[int]]) -> int:
    rows, cols = len(matrix), len(matrix[0])
    memo = {}

    def dfs(r, c):
        if (r, c) in memo:
            return memo[(r, c)]
        best = 1
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                best = max(best, 1 + dfs(nr, nc))
        memo[(r, c)] = best
        return best

    return max(dfs(r, c) for r in range(rows) for c in range(cols))
```

### Complexity

- **Time:** $O(m \cdot n)$ — each cell computed once, constant work per cell.
- **Space:** $O(m \cdot n)$ for the memo and recursion.

## Approach 2: Topological sort (peel the DAG)

### Intuition

Compute each cell's out-degree = number of strictly larger neighbours. Cells with out-degree 0 are local maxima (path length 1). Peel them like Kahn's algorithm: removing a cell decrements its smaller neighbours' out-degrees; each peeled layer is one longer path.

### Algorithm

1. Compute `outdeg[r][c]` = count of larger neighbours.
2. Queue all cells with `outdeg == 0`. `length = 0`.
3. While the queue is non-empty: `length += 1`; for each of the current layer's cells, for each *smaller* neighbour, decrement its out-degree and enqueue at 0.
4. Return `length`.

```python
from collections import deque

def longestIncreasingPath(matrix: list[list[int]]) -> int:
    rows, cols = len(matrix), len(matrix[0])
    outdeg = [[0] * cols for _ in range(rows)]
    dirs = ((1, 0), (-1, 0), (0, 1), (0, -1))

    for r in range(rows):
        for c in range(cols):
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                    outdeg[r][c] += 1

    q = deque((r, c) for r in range(rows) for c in range(cols) if outdeg[r][c] == 0)
    length = 0
    while q:
        length += 1
        for _ in range(len(q)):
            r, c = q.popleft()
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] < matrix[r][c]:
                    outdeg[nr][nc] -= 1
                    if outdeg[nr][nc] == 0:
                        q.append((nr, nc))
    return length
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Adding a visited set to the DFS.** Unnecessary and misleading — strict increase guarantees you never loop back; a visited set would even be wrong across different starting cells (it would block legitimate re-entry) unless it is the memo.
- **`>=` instead of `>`.** Equal neighbours are not part of a *strictly* increasing path; `>=` creates cycles and breaks the memo.
- **Not taking the max over all start cells.** The longest path can begin anywhere.
- **Kahn direction.** Peel from local maxima (out-degree 0 toward larger cells) and relax *smaller* neighbours.

## The keystone

A grid with a monotonic constraint is a DAG, and longest-path on a DAG is just memoized DFS (or layer-peeling). The strict inequality is what buys you "no visited set" — the same reason [Course Schedule](/citadel/dsa/course-schedule) can order an acyclic graph.
