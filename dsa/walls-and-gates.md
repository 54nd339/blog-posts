---
title: Walls and Gates - Multi-Source BFS From Every Gate at Once
description: Filling each empty room with its distance to the nearest gate — seed a BFS queue with all gates simultaneously so the first visit to a room is along a shortest path.
date: 2021-12-23
draft: false
slug: /dsa/walls-and-gates
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

Running a BFS from every empty room would be `O((mn)^2)`. Instead, start one BFS from *all* gates at once. Because every gate is at distance 0 and the queue expands in lockstep, the first time BFS reaches a room, it is via the closest gate.

## Description

You are given an `m x n` grid: `-1` is a wall, `0` is a gate, `2147483647` (INF) is an empty room. Fill each empty room with the distance to its nearest gate. Leave a room INF if no gate can reach it.

**Example**

```
Input:  rooms = [[INF, -1,  0, INF],
                  [INF, INF, INF, -1],
                  [INF, -1, INF, -1],
                  [  0, -1, INF, INF]]
Output:         [[  3, -1,  0,  1],
                  [  2,  2,  1, -1],
                  [  1, -1,  2, -1],
                  [  0, -1,  3,  4]]
```

**Constraints**

- $1 \le m, n \le 250$

## Prerequisites

- Breadth-first search yields shortest paths on an unweighted graph.
- Multi-source BFS: seed the queue with all sources at distance 0.

## Approach 1: Multi-Source BFS

### Intuition

Enqueue every gate. Pop a cell, and for each empty neighbour still at INF, set it to `current + 1` and enqueue it. Each room is written exactly once — the first, shortest time it is reached.

### Algorithm

1. Push all `(r, c)` with `rooms[r][c] == 0` into a queue.
2. While the queue is non-empty: pop `(r, c)`. For each orthogonal neighbour `(nr, nc)` in bounds with `rooms[nr][nc] == INF`: set `rooms[nr][nc] = rooms[r][c] + 1`, enqueue it.

```python
from collections import deque

def wallsAndGates(rooms: list[list[int]]) -> None:
    INF = 2147483647
    rows, cols = len(rooms), len(rooms[0])
    q = deque()
    for r in range(rows):
        for c in range(cols):
            if rooms[r][c] == 0:
                q.append((r, c))

    while q:
        r, c = q.popleft()
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and rooms[nr][nc] == INF:
                rooms[nr][nc] = rooms[r][c] + 1
                q.append((nr, nc))
```

### Complexity

- **Time:** $O(m \cdot n)$ — every cell enters the queue at most once.
- **Space:** $O(m \cdot n)$ for the queue in the worst case.

## Approach 2: Depth-First Search from each gate

### Intuition

From each gate, DFS outward carrying a distance, and write it into a room only if it improves on what is there. Simpler to write, but a room can be rewritten several times.

### Algorithm

1. For each gate `(r, c)`, call `dfs(r, c, 0)`.
2. `dfs(r, c, d)`: return on out-of-bounds, wall, or `rooms[r][c] < d`. Set `rooms[r][c] = d`. Recurse into the four neighbours with `d + 1`.

```python
def wallsAndGates(rooms: list[list[int]]) -> None:
    rows, cols = len(rooms), len(rooms[0])

    def dfs(r, c, d):
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or rooms[r][c] < d):
            return
        rooms[r][c] = d
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            dfs(r + dr, c + dc, d + 1)

    for r in range(rows):
        for c in range(cols):
            if rooms[r][c] == 0:
                dfs(r, c, 0)
```

### Complexity

- **Time:** $O(m \cdot n \cdot g)$ worst case with `g` gates, since a room may be revisited once per gate.
- **Space:** $O(m \cdot n)$ recursion depth.

## Common Pitfalls

- **Running a separate BFS per room.** That is the `O((mn)^2)` trap the multi-source seed avoids.
- **The `rooms[r][c] < d` check in DFS.** The wall value `-1` must also stop the recursion; `-1 < d` handles it, but only because `d >= 0` — keep the wall check explicit if `d` could be 0 at a wall.
- **Overwriting a gate or wall.** Only rooms currently at INF (BFS) or strictly greater than the incoming distance (DFS) may be written.
- **Using a `visited` set in multi-source BFS.** Unnecessary — "still INF" *is* the visited test, and it doubles as the write.

## The keystone

Multi-source BFS answers "nearest of many targets" in one linear pass by collapsing all targets into a single distance-0 frontier. [Rotting Oranges](/citadel/dsa/rotting-oranges) is the same technique, where the frontier is every rotten orange and the answer is the number of BFS layers.
