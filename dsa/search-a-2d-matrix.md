---
title: Search a 2D Matrix - Treat the Grid as One Sorted Array
description: Searching a row-sorted matrix where each row starts above the previous row's end — a staircase walk is O(m+n), and one binary search over m times n cells is logarithmic.
date: 2021-07-21
draft: false
slug: /dsa/search-a-2d-matrix
tags:
  - Competitive Programming
  - Binary Search
  - Matrix
---

Because the first element of each row exceeds the last of the previous row, reading the matrix row by row gives one strictly increasing sequence. So it is a 1-D binary search plus arithmetic to map a flat index back to `(row, col)`.

## Description

Given an `m x n` matrix where each row is sorted left to right and each row's first element is greater than the previous row's last element, return whether `target` is present.

**Example**

```
Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
Output: true

Input:  matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
Output: false
```

**Constraints**

- $1 \le m, n \le 100$
- $-10^4 \le \text{matrix}[i][j], \text{target} \le 10^4$

## Prerequisites

- [Binary search](/citadel/dsa/binary-search).
- The flat-index mapping `k -> (k // n, k % n)`.

## Approach 1: Brute Force

### Intuition

Scan every cell.

### Algorithm

1. For each row, for each element, return `true` on a match.
2. Return `false` after the whole matrix.

```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    for row in matrix:
        for x in row:
            if x == target:
                return True
    return False
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(1)$.

## Approach 2: Staircase Search

### Intuition

Start at the top-right corner. That value is the largest in its row and the smallest in its column, so one comparison eliminates a whole row or a whole column.

### Algorithm

1. `r = 0`, `c = n - 1`.
2. While `r < m` and `c >= 0`: if `matrix[r][c] == target`, return `true`; if it is greater, `c -= 1`; else `r += 1`.
3. Return `false`.

```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    r, c = 0, len(matrix[0]) - 1
    while r < len(matrix) and c >= 0:
        v = matrix[r][c]
        if v == target:
            return True
        if v > target:
            c -= 1
        else:
            r += 1
    return False
```

### Complexity

- **Time:** $O(m + n)$.
- **Space:** $O(1)$.

## Approach 3: Binary Search (Two Pass)

### Intuition

Binary-search the rows by their first element to find the only row that could hold `target`, then binary-search that row.

### Algorithm

1. Binary-search for the last row whose first element is `<= target`.
2. If no such row, return `false`.
3. Binary-search that row for `target`.

```python
from bisect import bisect_right

def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    first_col = [row[0] for row in matrix]
    r = bisect_right(first_col, target) - 1
    if r < 0:
        return False
    row = matrix[r]
    i = bisect_right(row, target) - 1
    return i >= 0 and row[i] == target
```

### Complexity

- **Time:** $O(\log m + \log n)$.
- **Space:** $O(m)$ for the extracted column (avoidable with index math).

## Approach 4: Binary Search (One Pass)

### Intuition

Index cells `0 .. m*n - 1` in reading order; cell `k` is at `(k // n, k % n)`. Run one binary search on `k`.

### Algorithm

1. `lo = 0`, `hi = m * n - 1`.
2. While `lo <= hi`: `mid = (lo + hi) // 2`; `val = matrix[mid // n][mid % n]`. Equal → `true`. Less → `lo = mid + 1`. Greater → `hi = mid - 1`.
3. Return `false`.

```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    m, n = len(matrix), len(matrix[0])
    lo, hi = 0, m * n - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        val = matrix[mid // n][mid % n]
        if val == target:
            return True
        if val < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return False
```

### Complexity

- **Time:** $O(\log(m \cdot n)) = O(\log m + \log n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Dividing by `m` instead of `n` in the index mapping.** The row is `mid // n`, the column `mid % n`. Swapping `m` and `n` only shows up on non-square matrices.
- **Approach 3 with `bisect_left` or without the `- 1`.** You want the last row whose first element is `<= target`; the wrong variant picks the next row when `target` equals a row's first element.
- **Staircase from the wrong corner.** Top-right or bottom-left work (each comparison removes a row or column); top-left and bottom-right do not.

## The keystone

Data sorted under a total order — even laid out in 2-D — becomes a 1-D binary search after one coordinate transform. The `k -> (k // width, k % width)` mapping recurs in [Valid Sudoku](/citadel/dsa/valid-sudoku)'s box index and grid-as-array traversals.
