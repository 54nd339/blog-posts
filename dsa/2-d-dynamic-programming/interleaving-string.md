---
title: Interleaving String - Walk a Grid of Consumed Prefixes
description: Whether a third string is an interleaving of two others preserving each one's order — a 2-D reachability DP where cell (i, j) means s1's first i and s2's first j characters build s3's first i plus j.
date: 2022-04-23
draft: false
slug: /dsa/interleaving-string
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

`dp[i][j]` is `True` when the first `i` characters of `s1` and the first `j` of `s2` can be interleaved to form the first `i + j` of `s3`. You reach `(i, j)` from `(i-1, j)` if `s1[i-1] == s3[i+j-1]`, or from `(i, j-1)` if `s2[j-1] == s3[i+j-1]`.

## Description

Given strings `s1`, `s2`, `s3`, return `true` if `s3` is formed by interleaving `s1` and `s2` (each character used once, each source's order preserved).

**Example**

```
Input:  s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
Output: true

Input:  s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
Output: false
```

**Constraints**

- $0 \le \text{s1.length}, \text{s2.length} \le 100$
- $0 \le \text{s3.length} \le 200$

## Prerequisites

- Length check: `len(s1) + len(s2) == len(s3)` or it is instantly false.
- 2-D reachability DP, rollable to one row.

## Approach 1: 2-D DP

### Intuition

`dp[i][j]` reachable iff `(dp[i-1][j] and s1[i-1] == s3[i+j-1])` or `(dp[i][j-1] and s2[j-1] == s3[i+j-1])`. `dp[0][0] = True`.

### Algorithm

1. If lengths do not add up, return `False`.
2. `dp = [[False] * (n + 1) for _ in range(m + 1)]`, `dp[0][0] = True`.
3. Fill row 0 and column 0 by matching a single source against `s3`'s prefix, then the interior with the rule above.
4. Return `dp[m][n]`.

```python
def isInterleave(s1: str, s2: str, s3: str) -> bool:
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    for i in range(m + 1):
        for j in range(n + 1):
            if i > 0 and dp[i - 1][j] and s1[i - 1] == s3[i + j - 1]:
                dp[i][j] = True
            if j > 0 and dp[i][j - 1] and s2[j - 1] == s3[i + j - 1]:
                dp[i][j] = True
    return dp[m][n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: 1-D rolling row

### Intuition

`dp[i][j]` needs `dp[i-1][j]` (previous row, same column) and `dp[i][j-1]` (current row, previous column). A single array updated left to right, with `dp[j]` playing "from above" before it is overwritten, suffices.

### Algorithm

1. Length check. `dp = [False] * (n + 1)`, `dp[0] = True`; initialise the rest of row 0 from `s2` vs `s3`.
2. For `i` from `1` to `m`: `dp[0] = dp[0] and s1[i-1] == s3[i-1]`. For `j` from `1` to `n`: `dp[j] = (dp[j] and s1[i-1] == s3[i+j-1]) or (dp[j-1] and s2[j-1] == s3[i+j-1])`.
3. Return `dp[n]`.

```python
def isInterleave(s1: str, s2: str, s3: str) -> bool:
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False
    dp = [False] * (n + 1)
    dp[0] = True
    for j in range(1, n + 1):
        dp[j] = dp[j - 1] and s2[j - 1] == s3[j - 1]
    for i in range(1, m + 1):
        dp[0] = dp[0] and s1[i - 1] == s3[i - 1]
        for j in range(1, n + 1):
            dp[j] = (dp[j] and s1[i - 1] == s3[i + j - 1]) or \
                    (dp[j - 1] and s2[j - 1] == s3[i + j - 1])
    return dp[n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(n)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i, j)` = can `s1[i:]` and `s2[j:]` interleave into `s3[i+j:]`. Take from `s1` if `s1[i] == s3[i+j]`, or from `s2` if `s2[j] == s3[i+j]`.

### Algorithm

Top-down mirror of Approach 1, cached on `(i, j)`; base case `i == m and j == n` → `True`.

```python
from functools import lru_cache

def isInterleave(s1: str, s2: str, s3: str) -> bool:
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False

    @lru_cache(None)
    def f(i, j):
        if i == m and j == n:
            return True
        k = i + j
        if i < m and s1[i] == s3[k] and f(i + 1, j):
            return True
        if j < n and s2[j] == s3[k] and f(i, j + 1):
            return True
        return False

    return f(0, 0)
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Skipping the length check.** Without `m + n == len(s3)` the index `s3[i+j-1]` goes out of range or silently misvalidates.
- **Greedy character matching.** When `s3[k]` equals both `s1[i]` and `s2[j]`, you must try both branches; a greedy pick fails cases like `s1="ab", s2="a", s3="aab"` handled wrong.
- **Row-0 / column-0 initialisation.** They encode "use only one source"; forgetting them breaks every path that starts along an edge.
- **1-D update using a stale `dp[j-1]`.** Iterate `j` ascending so `dp[j-1]` is already this row's value.

## The keystone

"Can two ordered sequences merge into a third" is a grid walk where each step consumes one character from one source — the same `(i, j)` prefix table as [Longest Common Subsequence](/citadel/dsa/longest-common-subsequence), with a boolean reachability rule instead of a length.
