---
title: Regular Expression Matching - Dot, Star, and a 2-D Table
description: Full-string matching of a pattern with dot and star against a text — a 2-D DP over prefixes where star means zero occurrences or one more of its preceding element.
date: 2022-05-12
draft: false
slug: /dsa/regular-expression-matching
tags:
  - Competitive Programming
  - Dynamic Programming
  - Recursion
---

`dp[i][j]` = does `p[:j]` match `s[:i]`. A plain character or `.` consumes one text character when it matches. A `*` is the hard case: `p[j-2]*` matches zero copies (`dp[i][j-2]`) or, if `p[j-2]` matches `s[i-1]`, one more copy (`dp[i-1][j]`).

## Description

Given a string `s` and a pattern `p` supporting `.` (any single character) and `*` (zero or more of the preceding element), return `true` if `p` matches the **entire** string `s`.

**Example**

```
Input:  s = "aa", p = "a*"
Output: true

Input:  s = "mississippi", p = "mis*is*p*."
Output: false
```

**Constraints**

- $1 \le \text{s.length} \le 20$; $1 \le \text{p.length} \le 20$
- `s` is lowercase letters; `p` has lowercase letters, `.`, and `*`; every `*` has a valid preceding element.

## Prerequisites

- `*` always pairs with the token before it; treat `x*` as one unit.
- The zero-or-more branch, and the "consume one text char, stay on the same pattern unit" branch.

## Approach 1: 2-D DP (bottom-up)

### Intuition

`dp[0][0] = True`. Fill `dp[0][j]` for patterns like `a*b*c*` that can match the empty string. Then for each `(i, j)`:

- If `p[j-1]` is `.` or `p[j-1] == s[i-1]`: `dp[i][j] = dp[i-1][j-1]`.
- If `p[j-1] == '*'`: `dp[i][j] = dp[i][j-2]` (zero copies); OR, if `p[j-2]` matches `s[i-1]`, also `dp[i-1][j]`.

### Algorithm

1. `dp = [[False] * (n + 1) for _ in range(m + 1)]`, `dp[0][0] = True`.
2. For `j` from `1` to `n`: if `p[j-1] == '*'`, `dp[0][j] = dp[0][j-2]`.
3. For `i` from `1` to `m`, `j` from `1` to `n`: apply the rules above.
4. Return `dp[m][n]`.

```python
def isMatch(s: str, p: str) -> bool:
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True

    for j in range(1, n + 1):
        if p[j - 1] == "*":
            dp[0][j] = dp[0][j - 2]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j - 1] == "*":
                dp[i][j] = dp[i][j - 2]
                if p[j - 2] == "." or p[j - 2] == s[i - 1]:
                    dp[i][j] = dp[i][j] or dp[i - 1][j]
            elif p[j - 1] == "." or p[j - 1] == s[i - 1]:
                dp[i][j] = dp[i - 1][j - 1]

    return dp[m][n]
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Approach 2: Recursion with memoization

### Intuition

`f(i, j)` = does `p[j:]` match `s[i:]`. Look ahead: if `p[j+1] == '*'`, either skip `p[j:j+2]` (`f(i, j+2)`) or, on a first-char match, consume one text character (`f(i+1, j)`). Otherwise require a single-char match and recurse `f(i+1, j+1)`.

### Algorithm

1. `f(i, j)`: if `j == n`, return `i == m`.
2. `first = i < m and (p[j] == s[i] or p[j] == '.')`.
3. If `j + 1 < n and p[j+1] == '*'`: return `f(i, j + 2) or (first and f(i + 1, j))`.
4. Else: return `first and f(i + 1, j + 1)`.
5. Memoize on `(i, j)`.

```python
from functools import lru_cache

def isMatch(s: str, p: str) -> bool:
    m, n = len(s), len(p)

    @lru_cache(None)
    def f(i, j):
        if j == n:
            return i == m
        first = i < m and (p[j] == s[i] or p[j] == ".")
        if j + 1 < n and p[j + 1] == "*":
            return f(i, j + 2) or (first and f(i + 1, j))
        return first and f(i + 1, j + 1)

    return f(0, 0)
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Treating `*` as a standalone wildcard.** It modifies the *preceding* token; `x*` is one unit meaning "zero or more `x`".
- **Skipping the `dp[0][j]` initialisation.** Patterns like `a*b*` match `""`; without this, any input requiring that fails.
- **Star's "consume" branch checking the wrong character.** It compares `p[j-2]` (the element before `*`) against `s[i-1]`, not `p[j-1]`.
- **Partial match.** The pattern must match the *entire* string — the answer is `dp[m][n]`, and the base case is `i == m`, not `i <= m`.

## The keystone

`*` forces a two-way branch — "this unit contributes nothing" vs "this unit eats one more character" — and memoizing `(text index, pattern index)` tames the exponential search. This closes [2-D Dynamic Programming](/citadel/dsa/2-d-dynamic-programming); wildcard matching (`?` and `*`) is the same table with simpler rules.
