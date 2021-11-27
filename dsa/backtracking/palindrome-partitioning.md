---
title: Palindrome Partitioning - Cut Only Where the Prefix Reads the Same
description: Every way to split a string so each piece is a palindrome — recurse over cut positions, take a prefix only when it is a palindrome, optionally precompute the palindrome table.
date: 2021-11-27
draft: false
slug: /dsa/palindrome-partitioning
tags:
  - Competitive Programming
  - Backtracking
  - Dynamic Programming
---

A partition is a set of cut points. At each step you choose how long the next piece is; you may only choose a length whose prefix is a palindrome, then recurse on the rest. The palindrome check is the only thing standing between this and plain subset enumeration.

## Description

Given a string `s`, partition it so that every substring of the partition is a palindrome. Return all possible partitionings.

**Example**

```
Input:  s = "aab"
Output: [["a","a","b"],["aa","b"]]

Input:  s = "a"
Output: [["a"]]
```

**Constraints**

- $1 \le \text{s.length} \le 16$
- `s` contains only lowercase English letters.

## Prerequisites

- The include/exclude recursion tree ([Subsets](/citadel/dsa/subsets)), here recast as "where does the next cut go".
- Two-pointer palindrome test.

## Approach 1: Backtracking

### Intuition

Walk a start index across the string. For each `end` from `start`, if `s[start:end+1]` is a palindrome, add it to the current path and recurse from `end + 1`. When `start` reaches the end of the string, record the path.

### Algorithm

1. `res = []`, `part = []`.
2. `dfs(start)`: if `start == len(s)`, append `part[:]`; return.
3. For `end` from `start` to `len(s) - 1`: if `s[start:end+1]` is a palindrome, append it, `dfs(end + 1)`, pop.
4. `isPalindrome(l, r)`: shrink `l`/`r` inward while characters match.

```python
def partition(s: str) -> list[list[str]]:
    res = []
    part = []

    def is_pal(l, r):
        while l < r:
            if s[l] != s[r]:
                return False
            l += 1
            r -= 1
        return True

    def dfs(start):
        if start == len(s):
            res.append(part[:])
            return
        for end in range(start, len(s)):
            if is_pal(start, end):
                part.append(s[start:end + 1])
                dfs(end + 1)
                part.pop()

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ — up to $2^{n-1}$ partitions, each costing $O(n)$ to build and check.
- **Space:** $O(n)$ recursion depth, excluding the output.

## Approach 2: Backtracking with a precomputed palindrome table

### Intuition

The naive version re-runs the two-pointer check for the same substring many times. Precompute `dp[i][j] = True` when `s[i:j+1]` is a palindrome, then the recursion's inner test is an $O(1)$ lookup.

### Algorithm

1. Build `dp` with the standard expansion: `dp[i][j]` is true when `s[i] == s[j]` and (`j - i < 2` or `dp[i+1][j-1]`). Fill by increasing substring length, or by decreasing `i` then increasing `j`.
2. Run the same `dfs(start)` as Approach 1, replacing `is_pal(start, end)` with `dp[start][end]`.

```python
def partition(s: str) -> list[list[str]]:
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    for i in range(n - 1, -1, -1):
        for j in range(i, n):
            if s[i] == s[j] and (j - i < 2 or dp[i + 1][j - 1]):
                dp[i][j] = True

    res = []
    part = []

    def dfs(start):
        if start == n:
            res.append(part[:])
            return
        for end in range(start, n):
            if dp[start][end]:
                part.append(s[start:end + 1])
                dfs(end + 1)
                part.pop()

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ still, but the constant factor drops — no repeated palindrome scans. Table build is $O(n^2)$.
- **Space:** $O(n^2)$ for `dp`.

## Approach 3: Recursion returning sub-partitions

### Intuition

Define `solve(i)` as *all* partitionings of the suffix `s[i:]`. For each palindromic prefix `s[i:j+1]`, prepend it to every partitioning returned by `solve(j + 1)`. Memoize on `i`.

### Algorithm

1. `solve(i)`: if `i == n`, return `[[]]` (one empty partitioning).
2. For each `j >= i` with `s[i:j+1]` a palindrome: for each `tail` in `solve(j + 1)`, collect `[s[i:j+1]] + tail`.
3. Cache `solve(i)` in a dict.

```python
from functools import lru_cache

def partition(s: str) -> list[list[str]]:
    n = len(s)

    def is_pal(l, r):
        while l < r:
            if s[l] != s[r]:
                return False
            l += 1
            r -= 1
        return True

    @lru_cache(None)
    def solve(i):
        if i == n:
            return [[]]
        out = []
        for j in range(i, n):
            if is_pal(i, j):
                for tail in solve(j + 1):
                    out.append([s[i:j + 1]] + tail)
        return out

    return [list(p) for p in solve(0)]
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ — the output size dominates; memoization removes recomputation of shared suffixes.
- **Space:** $O(n \cdot 2^n)$ for the cached partition lists.

## Common Pitfalls

- **Recursing from `end` instead of `end + 1`.** The next piece must begin *after* the one you just took, or you re-consume the last character.
- **Slicing bounds.** `s[start:end + 1]` is the substring from `start` through `end` inclusive; forgetting the `+ 1` drops the last character and breaks both the palindrome check and the recorded piece.
- **Recording `part` not `part[:]`.** `part` keeps mutating; store a copy at the leaf.
- **Building the `dp` table in the wrong order.** `dp[i][j]` depends on `dp[i+1][j-1]`, so `i` must decrease and `j` increase (or iterate by length).

## The keystone

Palindrome Partitioning is subset enumeration with a feasibility gate on each choice: a piece is allowed only if it is a palindrome. Precomputing that gate is the same move as carrying a trie in [Word Search II](/citadel/dsa/word-search-ii) — turn a repeated check into a table lookup so the recursion stays cheap. Next, [Letter Combinations of a Phone Number](/citadel/dsa/letter-combinations-of-a-phone-number) drops the gate entirely and just enumerates a product.
