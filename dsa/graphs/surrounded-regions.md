---
title: Surrounded Regions - Save What Touches the Border, Flip the Rest
description: Capturing regions of O fully enclosed by X — instead of testing each region for enclosure, flood from the border O cells to mark the survivors, then flip everything else.
date: 2022-01-12
draft: false
slug: /dsa/surrounded-regions
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

A region of `O`s is captured unless it touches the border. Testing each region for a border connection is fiddly; instead flood inward from every border `O`, marking it safe. After the flood, every unmarked `O` is surrounded — flip it.

## Description

Given an `m x n` board of `'X'` and `'O'`, flip to `'X'` every `'O'` that is not connected (4-directionally) to an `'O'` on the border. Modify the board in place.

**Example**

```
Input:  [["X","X","X","X"],
         ["X","O","O","X"],
         ["X","X","O","X"],
         ["X","O","X","X"]]
Output: [["X","X","X","X"],
         ["X","X","X","X"],
         ["X","X","X","X"],
         ["X","O","X","X"]]
```

**Constraints**

- $1 \le m, n \le 200$
- Each cell is `'X'` or `'O'`.

## Prerequisites

- Border-seeded flood fill.
- Using a temporary sentinel to mark "keep" cells, then a final sweep.

## Approach 1: Depth-First Search from the border

### Intuition

DFS from every `'O'` on the four edges, marking each reachable `'O'` with a temporary `'#'`. Then sweep the board: `'O'` → `'X'` (it was surrounded), `'#'` → `'O'` (it was safe).

### Algorithm

1. For every border cell that is `'O'`, run `dfs`.
2. `dfs(r, c)`: return unless in bounds and `board[r][c] == 'O'`. Set `board[r][c] = '#'`. Recurse into the four neighbours.
3. Sweep: for every cell, `'O'` becomes `'X'`, `'#'` becomes `'O'`.

```python
def solve(board: list[list[str]]) -> None:
    rows, cols = len(board), len(board[0])

    def dfs(r, c):
        if r < 0 or c < 0 or r >= rows or c >= cols or board[r][c] != "O":
            return
        board[r][c] = "#"
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        dfs(r, 0)
        dfs(r, cols - 1)
    for c in range(cols):
        dfs(0, c)
        dfs(rows - 1, c)

    for r in range(rows):
        for c in range(cols):
            if board[r][c] == "O":
                board[r][c] = "X"
            elif board[r][c] == "#":
                board[r][c] = "O"
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$ recursion worst case.

## Approach 2: Breadth-First Search from the border

### Intuition

Same plan, queue instead of recursion — safer for a 200×200 board where a deep DFS could overflow the stack.

### Algorithm

1. Enqueue every border `'O'`, mark it `'#'`.
2. Drain the queue: for each `'O'` neighbour, mark `'#'` and enqueue.
3. Sweep as in Approach 1.

```python
from collections import deque

def solve(board: list[list[str]]) -> None:
    rows, cols = len(board), len(board[0])
    q = deque()

    for r in range(rows):
        for c in (0, cols - 1):
            if board[r][c] == "O":
                board[r][c] = "#"
                q.append((r, c))
    for c in range(cols):
        for r in (0, rows - 1):
            if board[r][c] == "O":
                board[r][c] = "#"
                q.append((r, c))

    while q:
        r, c = q.popleft()
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] == "O":
                board[nr][nc] = "#"
                q.append((nr, nc))

    for r in range(rows):
        for c in range(cols):
            board[r][c] = "X" if board[r][c] == "O" else ("O" if board[r][c] == "#" else board[r][c])
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$ for the queue.

## Approach 3: Union-Find

### Intuition

Add a virtual node `BORDER`. Union every border `'O'` with `BORDER`, and union each `'O'` with its `'O'` neighbours. Finally, flip every `'O'` whose root differs from `find(BORDER)`.

### Algorithm

1. DSU over `rows*cols + 1`; the extra slot is `BORDER`.
2. For each `'O'`: if on the border, union with `BORDER`; union with right / down `'O'` neighbours.
3. For each `'O'` cell, if `find(cell) != find(BORDER)`, set it to `'X'`.

```python
def solve(board: list[list[str]]) -> None:
    rows, cols = len(board), len(board[0])
    BORDER = rows * cols
    parent = list(range(rows * cols + 1))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(a, b):
        parent[find(a)] = find(b)

    for r in range(rows):
        for c in range(cols):
            if board[r][c] != "O":
                continue
            idx = r * cols + c
            if r in (0, rows - 1) or c in (0, cols - 1):
                union(idx, BORDER)
            if r + 1 < rows and board[r + 1][c] == "O":
                union(idx, (r + 1) * cols + c)
            if c + 1 < cols and board[r][c + 1] == "O":
                union(idx, idx + 1)

    root = find(BORDER)
    for r in range(rows):
        for c in range(cols):
            if board[r][c] == "O" and find(r * cols + c) != root:
                board[r][c] = "X"
```

### Complexity

- **Time:** $O(m \cdot n \cdot \alpha(mn))$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Flipping during the flood.** Mark survivors with a temporary sentinel; do the `O`/`X` decision only in the final sweep, or you corrupt cells you still need to read.
- **Forgetting to restore `'#'` to `'O'`.** The sentinel must go back to `'O'` at the end.
- **Iterating only two edges.** All four borders seed the flood.
- **Deep recursion on a large all-`O` board.** Prefer BFS or Union-Find at 200×200.

## The keystone

"Enclosed unless it touches the boundary" inverts cleanly: flood *from* the boundary, and whatever the flood misses is enclosed. That is the same border-seed used in [Pacific Atlantic Water Flow](/citadel/dsa/pacific-atlantic-water-flow). Next, [Course Schedule](/citadel/dsa/course-schedule) moves off the grid to a dependency graph.
