---
title: Unique Paths - Every Cell Is the Sum of Above and Left
description: Counting lattice paths from the top-left to the bottom-right of a grid moving only right or down — each cell's count is the sum of the cell above and the cell to its left.
date: 2022-04-06
draft: false
slug: /dsa/unique-paths
tags:
  - Competitive Programming
  - Dynamic Programming
  - Combinatorics
---

You reach cell `(r, c)` only from `(r-1, c)` or `(r, c-1)`, so `paths(r, c) = paths(r-1, c) + paths(r, c-1)`. The top row and left column are all 1 (one straight path). Or skip the grid entirely: the answer is a single binomial coefficient.

## Description

A robot starts at the top-left of an `m x n` grid and can only move right or down. Return the number of distinct paths to the bottom-right corner.

**Example**

```
Input:  m = 3, n = 7
Output: 28

Input:  m = 3, n = 2
Output: 3
```

**Constraints**

- $1 \le m, n \le 100$
- The answer fits in a 32-bit integer.

## Prerequisites

- The two-predecessor grid recurrence.
- Rolling a 2-D table down to one row.
- The paths-as-choices bijection: $\binom{m+n-2}{m-1}$.

## Approach 1: 2-D DP

### Intuition

`dp[r][c]` = paths to `(r, c)`. First row and first column are 1; every other cell sums its top and left neighbours.

### Algorithm

1. `dp = [[1] * n for _ in range(m)]`.
2. For `r` from `1` to `m - 1`, `c` from `1` to `n - 1`: `dp[r][c] = dp[r-1][c] + dp[r][c-1]`.
3. Return `dp[m-1][n-1]`.

```python
def uniquePaths(m: int, n: int) -> int:
    dp = [[1] * n for _ in range(m)]
    for r in range(1, m):
        for c in range(1, n):
            dp[r][c] = dp[r - 1][c] + dp[r][c - 1]
    return dp[m - 1][n - 1]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: 1-D rolling row

### Intuition

Computing row `r` needs only row `r - 1`. Keep one array `row`; `row[c] += row[c - 1]` in place folds "above" (old `row[c]`) and "left" (updated `row[c-1]`) together.

### Algorithm

1. `row = [1] * n`.
2. Repeat `m - 1` times: for `c` from `1` to `n - 1`: `row[c] += row[c - 1]`.
3. Return `row[n - 1]`.

```python
def uniquePaths(m: int, n: int) -> int:
    row = [1] * n
    for _ in range(m - 1):
        for c in range(1, n):
            row[c] += row[c - 1]
    return row[n - 1]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(n)$.

## Approach 3: Combinatorics

### Intuition

Every path is a sequence of `m - 1` downs and `n - 1` rights in some order — choose which of the `m + n - 2` moves are downs: $\binom{m+n-2}{m-1}$.

### Algorithm

1. Compute `comb(m + n - 2, m - 1)` with an integer-exact product (or `math.comb`).

```python
from math import comb

def uniquePaths(m: int, n: int) -> int:
    return comb(m + n - 2, m - 1)
```

### Complexity

- **Time:** $O(\min(m, n))$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Initialising the whole grid to 0.** The first row and column must be 1; only interior cells are computed.
- **Rolling-row order.** Iterate `c` left to right so `row[c-1]` is already this row's value when you read it.
- **Combinatorics overflow.** Fine in Python; in fixed-width languages build the product with alternating multiply/divide to stay in range.
- **Confusing `m` and `n`.** `m` rows, `n` columns; the binomial is symmetric so `comb(m+n-2, m-1) == comb(m+n-2, n-1)`, but the grid loops are not.

## The keystone

The "sum of the neighbours you could have come from" recurrence is the spine of grid DP, and its space always rolls to one row. [Longest Common Subsequence](/citadel/dsa/longest-common-subsequence) is the same table shape with a match/no-match rule instead of a plain sum.
