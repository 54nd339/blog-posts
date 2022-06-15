---
title: Set Matrix Zeroes - Use the First Row and Column as Notepad
description: Zeroing every row and column that contains a zero, in place — record which rows and columns to clear in the matrix's own first row and column, with one extra flag for their overlap.
date: 2022-06-15
draft: false
slug: /dsa/set-matrix-zeroes
tags:
  - Competitive Programming
  - Math & Geometry
  - Matrix
---

You need to remember which rows and columns contain a zero without a separate `O(m + n)` structure. The trick: use row 0 and column 0 as the marker arrays. Their own zero-status needs one scalar flag each, because writing markers would overwrite it.

## Description

Given an `m x n` matrix, if an element is 0, set its entire row and column to 0. Do it in place.

**Example**

```
Input:  [[1,1,1],[1,0,1],[1,1,1]]
Output: [[1,0,1],[0,0,0],[1,0,1]]

Input:  [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
Output: [[0,0,0,0],[0,4,5,0],[0,3,1,0]]
```

**Constraints**

- $1 \le m, n \le 200$
- $-2^{31} \le \text{matrix}[i][j] \le 2^{31} - 1$

## Prerequisites

- Marking rows/columns in the first row/column.
- A separate flag for whether the first row (or column) itself must be zeroed.

## Approach 1: First row/column as markers, O(1) extra

### Intuition

First check whether row 0 and column 0 originally contain a zero (two booleans). Then, for each interior cell that is 0, set `matrix[i][0] = 0` and `matrix[0][j] = 0`. Second pass: zero any interior cell whose row marker or column marker is 0. Finally, zero row 0 and/or column 0 based on the saved booleans.

### Algorithm

1. `first_row_zero = any(matrix[0][j] == 0 for j in range(n))`; `first_col_zero = any(matrix[i][0] == 0 for i in range(m))`.
2. For `i` in `1..m-1`, `j` in `1..n-1`: if `matrix[i][j] == 0`, set `matrix[i][0] = matrix[0][j] = 0`.
3. For `i` in `1..m-1`, `j` in `1..n-1`: if `matrix[i][0] == 0` or `matrix[0][j] == 0`, `matrix[i][j] = 0`.
4. If `first_row_zero`, zero all of row 0. If `first_col_zero`, zero all of column 0.

```python
def setZeroes(matrix: list[list[int]]) -> None:
    m, n = len(matrix), len(matrix[0])
    first_row_zero = any(matrix[0][j] == 0 for j in range(n))
    first_col_zero = any(matrix[i][0] == 0 for i in range(m))

    for i in range(1, m):
        for j in range(1, n):
            if matrix[i][j] == 0:
                matrix[i][0] = 0
                matrix[0][j] = 0

    for i in range(1, m):
        for j in range(1, n):
            if matrix[i][0] == 0 or matrix[0][j] == 0:
                matrix[i][j] = 0

    if first_row_zero:
        for j in range(n):
            matrix[0][j] = 0
    if first_col_zero:
        for i in range(m):
            matrix[i][0] = 0
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(1)$.

## Approach 2: Two sets of indices

### Intuition

First pass collects the row indices and column indices that contain a zero. Second pass zeroes any cell whose row or column is in a set.

### Algorithm

1. `zero_rows = set()`, `zero_cols = set()`.
2. Scan all cells; on a 0, add `i` to `zero_rows`, `j` to `zero_cols`.
3. Scan again; zero `matrix[i][j]` if `i in zero_rows or j in zero_cols`.

```python
def setZeroes(matrix: list[list[int]]) -> None:
    m, n = len(matrix), len(matrix[0])
    zero_rows, zero_cols = set(), set()
    for i in range(m):
        for j in range(n):
            if matrix[i][j] == 0:
                zero_rows.add(i)
                zero_cols.add(j)
    for i in range(m):
        for j in range(n):
            if i in zero_rows or j in zero_cols:
                matrix[i][j] = 0
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m + n)$.

## Common Pitfalls

- **Zeroing as you scan.** A zero written in pass one would be read as an "original" zero later, cascading across the whole matrix. Mark first, apply second.
- **Forgetting the first row/column flags.** Once you write markers into row 0 and column 0, you can no longer tell whether they were originally zero — capture that before writing.
- **Applying the first row/column before the interior.** Zero the interior using the markers first; then handle row 0 and column 0 last.
- **Iterating the marker application over index 0.** Keep the interior loops at `1..m-1` / `1..n-1`; row 0 and column 0 are handled by the flags.

## The keystone

When you are denied extra space, the data structure itself can hold your bookkeeping — here, the border cells double as the "which rows/cols to clear" arrays, with two scalars patching the self-reference. This "store metadata in unused capacity" idea recurs in in-place hashing and cycle-marking tricks.
