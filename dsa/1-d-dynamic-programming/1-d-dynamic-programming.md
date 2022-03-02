---
title: 1-D Dynamic Programming - One Axis, One Recurrence
description: A guide to the 1-D DP section of NeetCode 150 — define the state as a single index or amount, find how it depends on a few earlier states, then roll the array down to a handful of variables.
date: 2022-03-02
draft: false
slug: /dsa/1-d-dynamic-programming
tags:
  - Competitive Programming
  - Dynamic Programming
  - Recursion
---

Every problem here has a state indexed by one number — a position in an array, a prefix length, a target amount — and a recurrence that reaches back a fixed distance. Write the recurrence, pick top-down memoization or bottom-up iteration, then, if the recurrence only reads the last one or two entries, drop the array for scalars.

## The method

1. **State.** "`dp[i]` = the answer for the first `i` elements" or "... for amount `i`".
2. **Recurrence.** Express `dp[i]` from smaller states. Usually a `min`/`max`/`+` over one or two predecessors, sometimes with a validity gate.
3. **Base case.** `dp[0]` (empty prefix) or `dp[0] = 0`.
4. **Direction.** Bottom-up if the dependencies are simple; top-down `@lru_cache` if the state space is sparse or the transitions are awkward.
5. **Compress.** Reading only `dp[i-1]`, `dp[i-2]` → two variables, $O(1)$ space.

## The families

**Fibonacci-shaped: `dp[i]` from `dp[i-1]` and `dp[i-2]`.**

- [Climbing Stairs](/citadel/dsa/climbing-stairs) — sum the two.
- [Min Cost Climbing Stairs](/citadel/dsa/min-cost-climbing-stairs) — `min` the two, add a toll.
- [House Robber](/citadel/dsa/house-robber) — `max(dp[i-1], dp[i-2] + nums[i])`.
- [House Robber II](/citadel/dsa/house-robber-ii) — run House Robber twice for the circular street.
- [Decode Ways](/citadel/dsa/decode-ways) — same shape, each term gated by digit validity.

**Expand around center (palindromes).**

- [Longest Palindromic Substring](/citadel/dsa/longest-palindromic-substring) — track the widest span.
- [Palindromic Substrings](/citadel/dsa/palindromic-substrings) — count every expansion.

**Knapsack on one number.**

- [Coin Change](/citadel/dsa/coin-change) — unbounded, minimise count.
- [Partition Equal Subset Sum](/citadel/dsa/partition-equal-subset-sum) — 0/1, boolean reachability of `total/2`.

**Best-so-far scans.**

- [Maximum Product Subarray](/citadel/dsa/maximum-product-subarray) — carry running max *and* min.
- [Word Break](/citadel/dsa/word-break) — prefix reachability with a dictionary gate.
- [Longest Increasing Subsequence](/citadel/dsa/longest-increasing-subsequence) — "best ending at `i`" ($O(n^2)$) or the `tails` array ($O(n \log n)$).

## Recognising it

- "count the ways to ...", "minimum / maximum ... over choices", "can you ..." with an array or string.
- The choice at each step is small (take/skip, 1-step/2-step, which coin) and only the recent past matters.
- Greedy gives a wrong answer on a small counterexample (e.g. Coin Change `[1,3,4]`, amount 6).

## The template

```python
from functools import lru_cache

def solve(arr):
    n = len(arr)

    @lru_cache(None)
    def dp(i):
        if i >= n:
            return BASE
        return COMBINE(dp(i + 1), dp(i + 2), arr[i])

    return dp(0)
```

Convert to a bottom-up loop and then to rolling variables once the shape is clear.

## Where this goes next

[2-D Dynamic Programming](/citadel/dsa/2-d-dynamic-programming) adds a second index — two strings, a grid, or "amount plus item count" — so the state is `dp[i][j]` and the compression is to one or two rows instead of one or two scalars.
