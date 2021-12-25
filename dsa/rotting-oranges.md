---
title: Rotting Oranges - Count the BFS Layers
description: Minutes until no fresh orange remains as rot spreads to orthogonal neighbours — multi-source BFS from every rotten orange, where the answer is the number of frontier expansions.
date: 2021-12-25
draft: false
slug: /dsa/rotting-oranges
tags:
  - Competitive Programming
  - Graphs
  - Matrix
---

Every rotten orange spreads at the same rate, so seed the BFS with all of them. Each full expansion of the frontier is one minute. Track fresh oranges; if any survive after the queue drains, it is impossible.

## Description

In an `m x n` grid, `0` is empty, `1` is a fresh orange, `2` is rotten. Each minute, a fresh orange orthogonally adjacent to a rotten one becomes rotten. Return the minimum minutes until no fresh orange remains, or `-1` if some fresh orange can never rot.

**Example**

```
Input:  grid = [[2,1,1],
                 [1,1,0],
                 [0,1,1]]
Output: 4

Input:  grid = [[2,1,1],[0,1,1],[1,0,1]]
Output: -1
```

**Constraints**

- $1 \le m, n \le 10$
- Each cell is `0`, `1`, or `2`.

## Prerequisites

- Multi-source BFS with level-by-level processing.
- Counting layers by draining the queue one snapshot at a time.

## Approach 1: Multi-Source BFS

### Intuition

Enqueue all rotten oranges and count fresh ones. Process the queue in waves: each wave rots every fresh neighbour of the current frontier and decrements the fresh count. The number of waves that actually rotted something is the answer.

### Algorithm

1. Scan the grid: enqueue every `2`, count every `1` into `fresh`.
2. `minutes = 0`. While the queue is non-empty and `fresh > 0`:
   - For each of the `len(queue)` cells in this wave: pop it; for each fresh neighbour, set it to `2`, `fresh -= 1`, enqueue it.
   - `minutes += 1`.
3. Return `minutes` if `fresh == 0` else `-1`.

```python
from collections import deque

def orangesRotting(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    q = deque()
    fresh = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                q.append((r, c))
            elif grid[r][c] == 1:
                fresh += 1

    minutes = 0
    while q and fresh > 0:
        for _ in range(len(q)):
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    q.append((nr, nc))
        minutes += 1

    return minutes if fresh == 0 else -1
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$ for the queue.

## Common Pitfalls

- **Incrementing `minutes` on an empty wave.** Guard the loop with `fresh > 0` (or check whether the wave rotted anything) so a grid that starts fully rotten returns `0`, not `1`.
- **Not snapshotting the wave size.** Capture `len(q)` before the inner loop; reading it live mixes this minute's spread with the next.
- **Forgetting the all-empty / no-fresh case.** `fresh == 0` at the start must return `0`.
- **Marking on dequeue instead of enqueue.** Rot (and decrement `fresh`) when you enqueue, or a cell can be counted by two neighbours in the same wave.

## The keystone

When a BFS frontier expands uniformly, the layer index *is* the time or distance — you get it for free by draining the queue one `len(q)` snapshot at a time. The same layer-counting turns [Word Ladder](/citadel/dsa/word-ladder) into a shortest-transformation count.
