---
title: Edit Distance - Insert, Delete, or Replace to the Minimum
description: The fewest single-character edits to turn one string into another — a 2-D DP where a matching pair costs nothing and a mismatch costs one plus the cheapest of insert, delete, or replace.
date: 2022-05-08
draft: false
slug: /dsa/edit-distance
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

`dp[i][j]` = edits to turn `word1[:i]` into `word2[:j]`. If the last characters match, `dp[i][j] = dp[i-1][j-1]`. Otherwise it is `1 + min` of delete (`dp[i-1][j]`), insert (`dp[i][j-1]`), replace (`dp[i-1][j-1]`).

## Description

Given `word1` and `word2`, return the minimum number of operations (insert, delete, or replace a single character) to convert `word1` into `word2`.

**Example**

```
Input:  word1 = "horse", word2 = "ros"
Output: 3   (horse → rorse → rose → ros)

Input:  word1 = "intention", word2 = "execution"
Output: 5
```

**Constraints**

- $0 \le \text{word1.length}, \text{word2.length} \le 500$
- Lowercase English letters.

## Prerequisites

- The three edit operations map to the three neighbouring cells.
- Base row/column = turning a string into the empty string (all deletions / insertions).

## Approach 1: 2-D DP

### Intuition

`dp[0][j] = j` (insert `j` characters), `dp[i][0] = i` (delete `i`). For the rest, a match copies the diagonal; a mismatch is `1 + min(left, up, diagonal)`.

### Algorithm

1. `dp = [[0] * (n + 1) for _ in range(m + 1)]`; `dp[i][0] = i`, `dp[0][j] = j`.
2. For `i` from `1` to `m`, `j` from `1` to `n`: if `word1[i-1] == word2[j-1]`, `dp[i][j] = dp[i-1][j-1]`; else `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`.
3. Return `dp[m][n]`.

```python
def minDistance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1])
    return dp[m][n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: Two rolling rows

### Intuition

`dp[i][*]` depends only on `dp[i-1][*]` and the current row's left neighbour. Keep `prev` and `cur`.

### Algorithm

1. `prev = list(range(n + 1))`.
2. For `i` from `1` to `m`: `cur = [i] + [0] * n`; fill `cur[j]` from `prev[j]`, `cur[j-1]`, `prev[j-1]`; then `prev = cur`.
3. Return `prev[n]`.

```python
def minDistance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)
    prev = list(range(n + 1))
    for i in range(1, m + 1):
        cur = [i] + [0] * n
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                cur[j] = prev[j - 1]
            else:
                cur[j] = 1 + min(prev[j], cur[j - 1], prev[j - 1])
        prev = cur
    return prev[n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(n)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i, j)` = edits for `word1[i:]` → `word2[j:]`. If one is exhausted, the answer is the other's remaining length. Match → `f(i+1, j+1)`; else `1 + min(f(i+1, j), f(i, j+1), f(i+1, j+1))`.

### Algorithm

Top-down mirror of Approach 1, cached on `(i, j)`.

```python
from functools import lru_cache

def minDistance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)

    @lru_cache(None)
    def f(i, j):
        if i == m:
            return n - j
        if j == n:
            return m - i
        if word1[i] == word2[j]:
            return f(i + 1, j + 1)
        return 1 + min(f(i + 1, j), f(i, j + 1), f(i + 1, j + 1))

    return f(0, 0)
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Base row/column left at 0.** `dp[i][0]` must be `i` and `dp[0][j]` must be `j` — the cost of an all-delete or all-insert conversion.
- **Only using two of the three predecessors.** Replace is the diagonal, insert is left, delete is up; dropping any one inflates the answer on some inputs.
- **Rolling-row: `cur[0]` must be `i`.** It represents deleting the first `i` characters of `word1`.
- **Taking the diagonal on a match without checking bounds.** Guard `i, j >= 1`.

## The keystone

Edit distance is the canonical two-string cost DP: match is free, and each of the three edits corresponds to one adjacent cell. Every alignment problem — diff, spell-check, sequence alignment — is a weighting of this exact recurrence.
