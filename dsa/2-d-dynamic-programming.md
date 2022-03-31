---
title: 2-D Dynamic Programming - Two Indices, One Table
description: A guide to the 2-D DP section of NeetCode 150 — the state is a pair of prefixes or a grid cell, the recurrence reads a couple of neighbouring cells, and the space rolls down to one or two rows.
date: 2022-03-31
draft: false
slug: /dsa/2-d-dynamic-programming
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

Add a second axis to [1-D DP](/citadel/dsa/1-d-dynamic-programming) and the state becomes `dp[i][j]` — how far you are into two strings, a position in a grid, or an amount paired with an item count. The recurrence still reads only a handful of nearby cells, and the space still compresses, now to one or two rows.

## The method

1. **State.** `dp[i][j]` = the answer for `s1`'s first `i` and `s2`'s first `j` characters (or grid cell `(i, j)`).
2. **Recurrence.** Usually a choice between the diagonal (`dp[i-1][j-1]`), up (`dp[i-1][j]`), and left (`dp[i][j-1]`), combined with `+`, `max`, `min`, or `or`.
3. **Base.** Row 0 and column 0 = one string is empty, or the grid edge.
4. **Order.** Bottom-up by increasing `i, j` (or by interval length for interval DP); or top-down `@lru_cache` on `(i, j)`.
5. **Compress.** Depends only on the previous row → keep `prev` and `cur`.

## The families

**Grid paths.**

- [Unique Paths](/citadel/dsa/unique-paths) — `dp[r][c] = dp[r-1][c] + dp[r][c-1]`; also a single binomial coefficient.

**Two-string prefix tables** — the biggest group, all the same shape.

- [Longest Common Subsequence](/citadel/dsa/longest-common-subsequence) — match extends the diagonal, else `max` of up/left.
- [Edit Distance](/citadel/dsa/edit-distance) — match copies the diagonal, else `1 + min` of the three.
- [Distinct Subsequences](/citadel/dsa/distinct-subsequences) — always skip `s`; on a match also add the diagonal (counting, so `+`).
- [Interleaving String](/citadel/dsa/interleaving-string) — boolean reachability from up or left, gated by the matching source character.
- [Regular Expression Matching](/citadel/dsa/regular-expression-matching) — `*` branches into "zero copies" and "one more copy".

**Knapsack with two dimensions.**

- [Coin Change II](/citadel/dsa/coin-change-ii) — `dp[coin][amount]`; loop order picks combinations over permutations.
- [Target Sum](/citadel/dsa/target-sum) — `(index, running sum)`, or reduce to one subset-sum.

**State machine over time.**

- [Best Time to Buy and Sell Stock With Cooldown](/citadel/dsa/best-time-to-buy-and-sell-stock-with-cooldown) — three states (hold / sold / rest), one sweep.

**DAG / grid memoized DFS.**

- [Longest Increasing Path in a Matrix](/citadel/dsa/longest-increasing-path-in-a-matrix) — strict increase ⇒ acyclic ⇒ memoize `longest(cell)`.

**Interval DP.**

- [Burst Balloons](/citadel/dsa/burst-balloons) — `dp[l][r]` picks which balloon in `(l, r)` bursts *last*; iterate by interval length, $O(n^3)$.

## Recognising it

- Two sequences compared, or "transform / match / interleave one string into another" → two-string prefix table.
- A grid with "paths", "min cost to traverse", "longest path" → grid DP or memoized DFS.
- "count / minimise over choices" where each choice advances one of two counters → `dp[i][j]`.
- An operation that merges or removes an element and reshapes its neighbours → interval DP.

## The template

```python
from functools import lru_cache

def solve(a, b):
    m, n = len(a), len(b)

    @lru_cache(None)
    def dp(i, j):
        if i == m or j == n:
            return BASE
        if a[i] == b[j]:
            return MATCH(dp(i + 1, j + 1))
        return COMBINE(dp(i + 1, j), dp(i, j + 1))

    return dp(0, 0)
```

## Where this goes next

[Greedy](/citadel/dsa/greedy) is the other side of the coin: when a locally optimal choice is provably globally optimal, you skip the table entirely and make one pass.
