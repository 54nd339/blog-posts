---
title: Spiral Matrix - Four Walls That Close In
description: Reading a matrix in spiral order — keep top, bottom, left, and right boundaries, walk one edge at a time, and pull that boundary inward after each edge.
date: 2022-06-14
draft: false
slug: /dsa/spiral-matrix
tags:
  - Competitive Programming
  - Math & Geometry
  - Matrix
---

Track four boundaries: `top`, `bottom`, `left`, `right`. Walk the top row left→right and drop `top`; the right column top→bottom and drop `right`; the bottom row right→left and raise `bottom`; the left column bottom→top and raise `left`. Repeat while the boundaries have not crossed.

## Description

Given an `m x n` matrix, return all its elements in spiral order (clockwise, starting at the top-left).

**Example**

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [1,2,3,6,9,8,7,4,5]

Input:  [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
Output: [1,2,3,4,8,12,11,10,9,5,6,7]
```

**Constraints**

- $1 \le m, n \le 10$
- $-100 \le \text{matrix}[i][j] \le 100$

## Prerequisites

- Four moving boundaries and the order of the four edge walks.
- Re-checking `top <= bottom` and `left <= right` between edges to stop at the center.

## Approach 1: Shrinking boundaries

### Intuition

Each of the four edge walks appends a line of the spiral and then moves its boundary one step inward. Between the horizontal and vertical walks, re-test the boundaries so a thin final row or column is not double-counted.

### Algorithm

1. `top, bottom = 0, m - 1`; `left, right = 0, n - 1`; `res = []`.
2. While `top <= bottom` and `left <= right`:
   - For `c` from `left` to `right`: append `matrix[top][c]`. `top += 1`.
   - For `r` from `top` to `bottom`: append `matrix[r][right]`. `right -= 1`.
   - If `top <= bottom`: for `c` from `right` down to `left`: append `matrix[bottom][c]`. `bottom -= 1`.
   - If `left <= right`: for `r` from `bottom` down to `top`: append `matrix[r][left]`. `left += 1`.
3. Return `res`.

```python
def spiralOrder(matrix: list[list[int]]) -> list[int]:
    if not matrix:
        return []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1
    res = []

    while top <= bottom and left <= right:
        for c in range(left, right + 1):
            res.append(matrix[top][c])
        top += 1

        for r in range(top, bottom + 1):
            res.append(matrix[r][right])
        right -= 1

        if top <= bottom:
            for c in range(right, left - 1, -1):
                res.append(matrix[bottom][c])
            bottom -= 1

        if left <= right:
            for r in range(bottom, top - 1, -1):
                res.append(matrix[r][left])
            left += 1

    return res
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(1)$ beyond the output.

## Approach 2: Direction vector with visited marking

### Intuition

Walk with a `(dr, dc)` heading that cycles right → down → left → up. Turn when the next cell is out of bounds or already visited.

### Algorithm

1. `dirs = [(0,1),(1,0),(0,-1),(-1,0)]`, `d = 0`, `r = c = 0`, `seen = set()`.
2. Repeat `m * n` times: append `matrix[r][c]`, mark `(r, c)`. Compute the next cell; if it is off-grid or seen, `d = (d + 1) % 4`. Step with `dirs[d]`.

```python
def spiralOrder(matrix: list[list[int]]) -> list[int]:
    m, n = len(matrix), len(matrix[0])
    dirs = [(0, 1), (1, 0), (0, -1), (-1, 0)]
    d = 0
    r = c = 0
    seen = set()
    res = []
    for _ in range(m * n):
        res.append(matrix[r][c])
        seen.add((r, c))
        nr, nc = r + dirs[d][0], c + dirs[d][1]
        if not (0 <= nr < m and 0 <= nc < n) or (nr, nc) in seen:
            d = (d + 1) % 4
            nr, nc = r + dirs[d][0], c + dirs[d][1]
        r, c = nr, nc
    return res
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$ for the visited set.

## Common Pitfalls

- **Skipping the mid-loop boundary re-checks.** For a single leftover row or column, the third and fourth walks would re-emit cells already taken — guard them with `if top <= bottom` and `if left <= right`.
- **Updating boundaries at the wrong time.** Move a boundary immediately after its edge walk, not at the end of the iteration.
- **Assuming a square matrix.** `m` and `n` differ; keep row and column boundaries separate.
- **Loop condition.** Use `while top <= bottom and left <= right`; `<` alone drops the center cell of an odd-sized matrix.

## The keystone

Boundary variables that march inward turn a traversal with changing direction into four straight-line loops. The same four-wall shrink drives the layer method in [Rotate Image](/citadel/dsa/rotate-image).
