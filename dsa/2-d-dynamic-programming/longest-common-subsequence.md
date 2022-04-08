---
title: Longest Common Subsequence - Match the Ends or Drop One
description: The length of the longest subsequence common to two strings — if the current characters match, add one and recurse on both shorter prefixes, otherwise take the better of dropping one character from either.
date: 2022-04-08
draft: false
slug: /dsa/longest-common-subsequence
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

`dp[i][j]` is the LCS length of `text1[:i]` and `text2[:j]`. If `text1[i-1] == text2[j-1]`, those characters can both end the LCS: `dp[i][j] = dp[i-1][j-1] + 1`. Otherwise one of them is unused: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

## Description

Given two strings `text1` and `text2`, return the length of their longest common subsequence, or `0` if there is none. A subsequence keeps relative order but need not be contiguous.

**Example**

```
Input:  text1 = "abcde", text2 = "ace"
Output: 3   ("ace")

Input:  text1 = "abc", text2 = "def"
Output: 0
```

**Constraints**

- $1 \le \text{text1.length}, \text{text2.length} \le 1000$
- Lowercase English letters.

## Prerequisites

- The 2-D prefix-vs-prefix table.
- Rolling two rows; the match / no-match branch.

## Approach 1: 2-D DP

### Intuition

Fill `dp` of size `(m+1) x (n+1)`, zero-initialised (an empty prefix shares nothing). Each cell either extends the diagonal on a match or takes the better of up / left.

### Algorithm

1. `dp = [[0] * (n + 1) for _ in range(m + 1)]`.
2. For `i` from `1` to `m`, `j` from `1` to `n`: if `text1[i-1] == text2[j-1]`, `dp[i][j] = dp[i-1][j-1] + 1`; else `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.
3. Return `dp[m][n]`.

```python
def longestCommonSubsequence(text1: str, text2: str) -> int:
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: Two rolling rows

### Intuition

`dp[i]` needs only `dp[i-1]`. Keep `prev` and `cur` of length `n + 1`.

### Algorithm

1. `prev = [0] * (n + 1)`.
2. For each `i`: `cur = [0] * (n + 1)`; fill `cur[j]` from `prev[j-1]`, `prev[j]`, `cur[j-1]`; then `prev = cur`.
3. Return `prev[n]`.

```python
def longestCommonSubsequence(text1: str, text2: str) -> int:
    m, n = len(text1), len(text2)
    prev = [0] * (n + 1)
    for i in range(1, m + 1):
        cur = [0] * (n + 1)
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                cur[j] = prev[j - 1] + 1
            else:
                cur[j] = max(prev[j], cur[j - 1])
        prev = cur
    return prev[n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(n)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i, j)` = LCS of the suffixes `text1[i:]` and `text2[j:]`. Match → `1 + f(i+1, j+1)`; mismatch → `max(f(i+1, j), f(i, j+1))`.

### Algorithm

Top-down mirror of Approach 1, cached on `(i, j)`.

```python
from functools import lru_cache

def longestCommonSubsequence(text1: str, text2: str) -> int:
    m, n = len(text1), len(text2)

    @lru_cache(None)
    def f(i, j):
        if i == m or j == n:
            return 0
        if text1[i] == text2[j]:
            return 1 + f(i + 1, j + 1)
        return max(f(i + 1, j), f(i, j + 1))

    return f(0, 0)
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Index shift.** `dp[i][j]` compares `text1[i-1]` with `text2[j-1]`; the row/column 0 stands for the empty prefix.
- **Confusing subsequence with substring.** Gaps are allowed here — do not reset on a mismatch (that would be *longest common substring*).
- **Rolling-row read order.** `cur[j]` needs `prev[j-1]`, `prev[j]`, and the already-updated `cur[j-1]`; overwrite `prev` only after the row is done.
- **Only one rolling row.** You still need `prev[j-1]` from the old row; a single in-place array loses it unless you stash the diagonal.

## The keystone

The match-the-ends-or-drop-one recurrence is the template for two-string DP: [Edit Distance](/citadel/dsa/edit-distance), [Distinct Subsequences](/citadel/dsa/distinct-subsequences), and [Interleaving String](/citadel/dsa/interleaving-string) are all this table with a different rule per cell.
