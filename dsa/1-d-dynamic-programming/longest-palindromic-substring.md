---
title: Longest Palindromic Substring - Grow Outward From Every Center
description: The longest contiguous palindrome in a string — treat each index and each gap as a center and expand while the characters match, keeping the widest span seen.
date: 2022-03-18
draft: false
slug: /dsa/longest-palindromic-substring
tags:
  - Competitive Programming
  - Dynamic Programming
  - Two Pointers
---

A palindrome is symmetric about its center. There are `2n - 1` possible centers — `n` single characters and `n - 1` gaps between characters. From each, push two pointers outward while they match; track the longest span.

## Description

Given a string `s`, return the longest substring of `s` that is a palindrome.

**Example**

```
Input:  s = "babad"
Output: "bab"   ("aba" is also valid)

Input:  s = "cbbd"
Output: "bb"
```

**Constraints**

- $1 \le \text{s.length} \le 1000$
- `s` contains only digits and English letters.

## Prerequisites

- Odd- and even-length palindromes need separate center types.
- Two-pointer outward expansion.

## Approach 1: Expand around center

### Intuition

For each `i`, expand `(i, i)` for odd-length palindromes and `(i, i + 1)` for even-length. Each expansion is $O(n)$; there are $O(n)$ centers.

### Algorithm

1. `expand(l, r)`: while `l >= 0`, `r < n`, `s[l] == s[r]`: `l -= 1`, `r += 1`. Return `(l + 1, r - 1)` — the last valid bounds.
2. For each `i`, run `expand(i, i)` and `expand(i, i + 1)`; keep the longer span.
3. Return `s[best_l : best_r + 1]`.

```python
def longestPalindrome(s: str) -> str:
    if not s:
        return ""
    start, end = 0, 0

    def expand(l, r):
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1
            r += 1
        return l + 1, r - 1

    for i in range(len(s)):
        for lo, hi in (expand(i, i), expand(i, i + 1)):
            if hi - lo > end - start:
                start, end = lo, hi

    return s[start:end + 1]
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Dynamic programming table

### Intuition

`dp[i][j]` is `True` when `s[i:j+1]` is a palindrome: `s[i] == s[j]` and (`j - i < 2` or `dp[i+1][j-1]`). Fill by increasing length; record the longest true span.

### Algorithm

1. `dp = [[False] * n for _ in range(n)]`; every `dp[i][i] = True`.
2. For length `L` from `2` to `n`, for each `i` with `j = i + L - 1`: set `dp[i][j]` per the rule; update the best span.
3. Return the best substring.

```python
def longestPalindrome(s: str) -> str:
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start, best = 0, 1
    for i in range(n):
        dp[i][i] = True

    for L in range(2, n + 1):
        for i in range(n - L + 1):
            j = i + L - 1
            if s[i] == s[j] and (L < 3 or dp[i + 1][j - 1]):
                dp[i][j] = True
                if L > best:
                    start, best = i, L

    return s[start:start + best]
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n^2)$.

## Approach 3: Brute force

### Intuition

Check every substring for the palindrome property, longest first.

### Algorithm

1. For length `L` from `n` down to `1`, for each start `i`: if `s[i:i+L]` reads the same reversed, return it.

```python
def longestPalindrome(s: str) -> str:
    n = len(s)
    for L in range(n, 0, -1):
        for i in range(n - L + 1):
            sub = s[i:i + L]
            if sub == sub[::-1]:
                return sub
    return ""
```

### Complexity

- **Time:** $O(n^3)$.
- **Space:** $O(1)$ extra (ignoring the slice).

## Common Pitfalls

- **Only checking odd centers.** `"cbbd"` needs the even center between the two `b`s.
- **Off-by-one on the returned bounds.** After the expansion loop overshoots, the valid palindrome is `s[l+1 : r]` (i.e. inclusive `l+1 .. r-1`).
- **Comparing spans with `>=` and clobbering ties.** Use `>` so the first (leftmost) longest wins, matching typical expectations.
- **DP fill order.** `dp[i][j]` needs `dp[i+1][j-1]`, so iterate by length or with `i` descending.

## The keystone

"Expand around center" is the two-pointer answer to almost every substring-palindrome question — $O(n^2)$ time, $O(1)$ space, no table. [Palindromic Substrings](/citadel/dsa/palindromic-substrings) is the identical scan where you *count* expansions instead of measuring the longest.
