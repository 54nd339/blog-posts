---
title: Rotate Image - Transpose, Then Reverse Each Row
description: Rotating an n by n matrix 90 degrees clockwise in place — swap across the main diagonal to transpose, then reverse every row, or rotate four cells at a time layer by layer.
date: 2022-06-12
draft: false
slug: /dsa/rotate-image
tags:
  - Competitive Programming
  - Math & Geometry
  - Matrix
---

A 90° clockwise rotation is a transpose (reflect over the main diagonal) followed by reversing each row. Both steps are in-place and touch each cell once. The alternative rotates cells in groups of four, one concentric ring at a time.

## Description

Given an `n x n` 2-D `matrix`, rotate it 90 degrees clockwise in place — do not allocate another matrix.

**Example**

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [[7,4,1],[8,5,2],[9,6,3]]
```

**Constraints**

- $1 \le n \le 20$
- $-1000 \le \text{matrix}[i][j] \le 1000$

## Prerequisites

- Transpose by swapping `matrix[i][j]` with `matrix[j][i]` for `j > i`.
- Reversing a list in place; or the four-way cyclic swap.

## Approach 1: Transpose then reverse rows

### Intuition

Transposing turns rows into columns. Reversing each transposed row then places what was the bottom-left at the top-left — a clockwise turn.

### Algorithm

1. For `i` in `range(n)`, `j` in `range(i + 1, n)`: swap `matrix[i][j]` and `matrix[j][i]`.
2. For each row, reverse it in place.

```python
def rotate(matrix: list[list[int]]) -> None:
    n = len(matrix)
    for i in range(n):
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    for row in matrix:
        row.reverse()
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Layer-by-layer four-way swap

### Intuition

Process concentric square rings from outside in. Within a ring, rotate four cells at a time: top-left ← bottom-left ← bottom-right ← top-right ← (saved top-left).

### Algorithm

1. For `layer` from `0` to `n // 2 - 1`: `first = layer`, `last = n - 1 - layer`.
2. For `i` from `first` to `last - 1`: `offset = i - first`; save `top = matrix[first][i]`; move `matrix[last - offset][first]` → top-left, `matrix[last][last - offset]` → bottom-left, `matrix[i][last]` → bottom-right, `top` → top-right.

```python
def rotate(matrix: list[list[int]]) -> None:
    n = len(matrix)
    for layer in range(n // 2):
        first, last = layer, n - 1 - layer
        for i in range(first, last):
            offset = i - first
            top = matrix[first][i]
            matrix[first][i] = matrix[last - offset][first]
            matrix[last - offset][first] = matrix[last][last - offset]
            matrix[last][last - offset] = matrix[i][last]
            matrix[i][last] = top
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Transposing with a full `j` range.** Start the inner loop at `j = i + 1`; going over all `j` swaps every pair twice and undoes the transpose.
- **Reversing columns instead of rows.** Clockwise = transpose then reverse *rows*. Reversing rows first then transposing gives a *counter*-clockwise rotation.
- **Layer method index arithmetic.** The four expressions must form a clean cycle; an off-by-one on `last - offset` corrupts the ring.
- **Allocating a new matrix.** The problem forbids it; both approaches above stay in place.

## The keystone

Many grid transforms factor into two simple in-place passes (a reflection and a reversal). Recognising a rotation as "transpose ∘ reverse" avoids fiddly four-index bookkeeping. [Spiral Matrix](/citadel/dsa/spiral-matrix) uses the same shrinking-boundary idea as the layer method.
