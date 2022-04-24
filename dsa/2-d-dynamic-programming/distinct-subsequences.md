---
title: Distinct Subsequences - Count the Ways One String Hides in Another
description: Counting how many distinct subsequences of s equal t — a 2-D DP where matching characters let you either use or skip the current s character, and mismatches force a skip.
date: 2022-04-24
draft: false
slug: /dsa/distinct-subsequences
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

`dp[i][j]` = number of ways the first `i` characters of `s` contain the first `j` characters of `t` as a subsequence. Always you may skip `s[i-1]`: `dp[i-1][j]`. If `s[i-1] == t[j-1]` you may *also* match it: add `dp[i-1][j-1]`.

## Description

Given strings `s` and `t`, return the number of distinct subsequences of `s` that equal `t`. The answer fits in a 32-bit signed integer.

**Example**

```
Input:  s = "rabbbit", t = "rabbit"
Output: 3

Input:  s = "babgbag", t = "bag"
Output: 5
```

**Constraints**

- $1 \le \text{s.length}, \text{t.length} \le 1000$
- Lowercase English letters.

## Prerequisites

- The 2-D prefix table with an "always skip, sometimes also match" transition.
- Base case: matching the empty `t` has exactly one way (delete everything).

## Approach 1: 2-D DP

### Intuition

`dp[i][0] = 1` for all `i` (one way to form the empty string). `dp[0][j] = 0` for `j > 0`. Then `dp[i][j] = dp[i-1][j] + (dp[i-1][j-1] if s[i-1] == t[j-1] else 0)`.

### Algorithm

1. `dp = [[0] * (n + 1) for _ in range(m + 1)]`; set `dp[i][0] = 1`.
2. For `i` from `1` to `m`, `j` from `1` to `n`: `dp[i][j] = dp[i-1][j]`; if `s[i-1] == t[j-1]`, `dp[i][j] += dp[i-1][j-1]`.
3. Return `dp[m][n]`.

```python
def numDistinct(s: str, t: str) -> int:
    m, n = len(s), len(t)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = 1
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            dp[i][j] = dp[i - 1][j]
            if s[i - 1] == t[j - 1]:
                dp[i][j] += dp[i - 1][j - 1]
    return dp[m][n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: 1-D rolling array

### Intuition

`dp[i][j]` needs `dp[i-1][j]` and `dp[i-1][j-1]` — the previous row only. Use one array and iterate `j` **descending** so `dp[j-1]` still holds the old row's value.

### Algorithm

1. `dp = [0] * (n + 1)`, `dp[0] = 1`.
2. For each character `ch` of `s`: for `j` from `n` down to `1`: if `ch == t[j-1]`, `dp[j] += dp[j-1]`.
3. Return `dp[n]`.

```python
def numDistinct(s: str, t: str) -> int:
    n = len(t)
    dp = [0] * (n + 1)
    dp[0] = 1
    for ch in s:
        for j in range(n, 0, -1):
            if ch == t[j - 1]:
                dp[j] += dp[j - 1]
    return dp[n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(n)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i, j)` = ways `s[i:]` contains `t[j:]`. If `j == n`, 1. If `i == m`, 0. `res = f(i+1, j)`; if `s[i] == t[j]`, `res += f(i+1, j+1)`.

### Algorithm

Top-down mirror of Approach 1, cached on `(i, j)`.

```python
from functools import lru_cache

def numDistinct(s: str, t: str) -> int:
    m, n = len(s), len(t)

    @lru_cache(None)
    def f(i, j):
        if j == n:
            return 1
        if i == m:
            return 0
        res = f(i + 1, j)
        if s[i] == t[j]:
            res += f(i + 1, j + 1)
        return res

    return f(0, 0)
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **`dp[i][0]` not 1.** Deleting every character of `s` is the one way to spell the empty `t`.
- **1-D iteration direction.** Ascending `j` would use the *current* row's `dp[j-1]`, double-counting. Go descending.
- **Adding `dp[i-1][j-1]` on a mismatch.** Only a matching character opens the "use it" option; a mismatch is skip-only.
- **Treating it like LCS.** LCS takes a `max`; here you *sum* the two contributions because you are counting, not measuring.

## The keystone

Counting embeddings of one string in another is the LCS table with `+` instead of `max` and an asymmetric transition (skip `s` freely, consume `t` only on a match). [Edit Distance](/citadel/dsa/edit-distance) is the same table again, now minimising a cost over insert/delete/replace.
