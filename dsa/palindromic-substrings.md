---
title: Palindromic Substrings - Count Every Expansion
description: Counting all palindromic substrings of a string — the expand-around-center scan where each successful outward step is one more palindrome, summed over all centers.
date: 2022-03-20
draft: false
slug: /dsa/palindromic-substrings
tags:
  - Competitive Programming
  - Dynamic Programming
  - Two Pointers
---

Same machinery as [Longest Palindromic Substring](/citadel/dsa/longest-palindromic-substring): every palindrome has a center, and there are `2n - 1` of them. Here, instead of tracking the widest span, add 1 to a counter for every position where the two pointers still match as they expand.

## Description

Given a string `s`, return the number of palindromic substrings. Substrings at different start or end indices count separately even if identical.

**Example**

```
Input:  s = "abc"
Output: 3   ("a", "b", "c")

Input:  s = "aaa"
Output: 6   ("a" x3, "aa" x2, "aaa")
```

**Constraints**

- $1 \le \text{s.length} \le 1000$
- `s` is lowercase English.

## Prerequisites

- Odd and even centers.
- Counting matches during outward expansion.

## Approach 1: Expand around center

### Intuition

From each center, every time the outward pointers match you have found one more palindrome; keep stepping until they don't. Sum over all `n` odd centers and `n - 1` even centers.

### Algorithm

1. `count_from(l, r)`: while `l >= 0`, `r < n`, `s[l] == s[r]`: `total += 1`, `l -= 1`, `r += 1`.
2. For each `i`: `count_from(i, i)` and `count_from(i, i + 1)`.
3. Return `total`.

```python
def countSubstrings(s: str) -> int:
    n = len(s)
    total = 0

    def count_from(l, r):
        nonlocal total
        while l >= 0 and r < n and s[l] == s[r]:
            total += 1
            l -= 1
            r += 1

    for i in range(n):
        count_from(i, i)
        count_from(i, i + 1)

    return total
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Dynamic programming table

### Intuition

`dp[i][j]` is `True` when `s[i:j+1]` is a palindrome; count every `True` cell.

### Algorithm

1. `dp[i][i] = True` (count `n`).
2. For length `L` from `2` to `n`, each `i` with `j = i + L - 1`: `dp[i][j] = s[i] == s[j] and (L < 3 or dp[i+1][j-1])`; if `True`, `count += 1`.
3. Return `count`.

```python
def countSubstrings(s: str) -> int:
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    count = 0
    for i in range(n):
        dp[i][i] = True
        count += 1

    for L in range(2, n + 1):
        for i in range(n - L + 1):
            j = i + L - 1
            if s[i] == s[j] and (L < 3 or dp[i + 1][j - 1]):
                dp[i][j] = True
                count += 1

    return count
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n^2)$.

## Approach 3: Brute force

### Intuition

Test every substring for the palindrome property.

### Algorithm

1. For each `i <= j`, check `s[i:j+1] == s[i:j+1][::-1]`; increment on a match.

```python
def countSubstrings(s: str) -> int:
    n = len(s)
    count = 0
    for i in range(n):
        for j in range(i, n):
            sub = s[i:j + 1]
            if sub == sub[::-1]:
                count += 1
    return count
```

### Complexity

- **Time:** $O(n^3)$.
- **Space:** $O(1)$ extra.

## Common Pitfalls

- **Counting only maximal palindromes.** Every nested palindrome counts — `"aaa"` yields 6, not 3. The expansion counts each step, which is exactly right.
- **Skipping even centers.** `"aa"` contributes a palindrome centred in the gap.
- **`nonlocal` / closure for the counter.** In Python the inner function needs `nonlocal total`, or return the per-center count and sum outside.
- **DP order.** Same length-ascending (or `i`-descending) requirement as the longest-palindrome table.

## The keystone

Once "expand around center" is in hand, the difference between "longest" and "how many" is whether you track a max or increment a sum. Both are $O(n^2)$/$O(1)$. Next, [Decode Ways](/citadel/dsa/decode-ways) leaves palindromes behind for a counting DP over string prefixes.
