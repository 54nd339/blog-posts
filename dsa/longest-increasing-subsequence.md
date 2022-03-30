---
title: Longest Increasing Subsequence - Patience Beats the Quadratic DP
description: The length of the longest strictly increasing subsequence — an O(n squared) DP over "best ending at i", or an O(n log n) method maintaining the smallest possible tail for each length.
date: 2022-03-30
draft: false
slug: /dsa/longest-increasing-subsequence
tags:
  - Competitive Programming
  - Dynamic Programming
  - Binary Search
---

The quadratic DP asks, for each `i`, "how long is the best increasing subsequence ending exactly at `i`". The $O(n \log n)$ method keeps `tails[k]` = the smallest value that can end an increasing subsequence of length `k + 1`; each new element replaces the first tail it cannot extend.

## Description

Given an integer array `nums`, return the length of the longest strictly increasing subsequence (elements need not be contiguous).

**Example**

```
Input:  nums = [10,9,2,5,3,7,101,18]
Output: 4   ([2,3,7,101])

Input:  nums = [0,1,0,3,2,3]
Output: 4
```

**Constraints**

- $1 \le \text{nums.length} \le 2500$
- $-10^4 \le \text{nums}[i] \le 10^4$

## Prerequisites

- "Best ending at index `i`" DP.
- The `tails` array invariant and binary search for the insertion point.

## Approach 1: Quadratic DP

### Intuition

`dp[i]` = length of the longest increasing subsequence ending at `i`. `dp[i] = 1 + max(dp[j])` over all `j < i` with `nums[j] < nums[i]`, or `1` if none. Answer is `max(dp)`.

### Algorithm

1. `dp = [1] * n`.
2. For `i` from `1` to `n - 1`: for `j` from `0` to `i - 1`: if `nums[j] < nums[i]`, `dp[i] = max(dp[i], dp[j] + 1)`.
3. Return `max(dp)`.

```python
def lengthOfLIS(nums: list[int]) -> int:
    n = len(nums)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n)$.

## Approach 2: Patience sorting with binary search

### Intuition

Keep `tails`, where `tails[k]` is the smallest tail value of any increasing subsequence of length `k + 1`. `tails` stays sorted. For each `x`, binary search the leftmost tail `>= x`: if found, overwrite it (a length-`k+1` subsequence now ends smaller); if not, append `x` (the LIS just grew). The length of `tails` is the answer.

### Algorithm

1. `tails = []`.
2. For each `x`: `pos = bisect_left(tails, x)`. If `pos == len(tails)`, append `x`; else `tails[pos] = x`.
3. Return `len(tails)`.

```python
from bisect import bisect_left

def lengthOfLIS(nums: list[int]) -> int:
    tails = []
    for x in nums:
        pos = bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i)` = LIS length starting at `i`. `f(i) = 1 + max(f(j))` over `j > i` with `nums[j] > nums[i]`. Answer is `max(f(i))`.

### Algorithm

Top-down version of the quadratic DP, cached on `i`.

```python
from functools import lru_cache

def lengthOfLIS(nums: list[int]) -> int:
    n = len(nums)

    @lru_cache(None)
    def f(i):
        best = 1
        for j in range(i + 1, n):
            if nums[j] > nums[i]:
                best = max(best, 1 + f(j))
        return best

    return max(f(i) for i in range(n))
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`bisect_right` instead of `bisect_left`.** With `bisect_right`, equal elements extend the sequence, giving the longest *non-decreasing* subsequence. Strictly increasing needs `bisect_left`.
- **Reading `tails` as an actual subsequence.** Its contents are not a valid LIS — only its *length* is meaningful.
- **Initialising `dp` to 0.** Every element alone is a subsequence of length 1.
- **Forgetting `max(dp)`.** The LIS need not end at the last index.

## The keystone

The `tails` trick — keep the smallest tail per length, binary-search each insertion — turns an $O(n^2)$ subsequence DP into $O(n \log n)$. It generalises to problems reducible to LIS, like the box-stacking / envelope-nesting family. Next, [Partition Equal Subset Sum](/citadel/dsa/partition-equal-subset-sum) switches to a boolean knapsack.
