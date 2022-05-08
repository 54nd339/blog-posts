---
title: Burst Balloons - Choose the Last Balloon in Each Range
description: Maximising coins from bursting balloons where each burst pays the product of its surviving neighbours — interval DP that picks which balloon in a range is burst last, so its neighbours are the range's borders.
date: 2022-05-08
draft: false
slug: /dsa/burst-balloons
tags:
  - Competitive Programming
  - Dynamic Programming
  - Interval DP
---

Bursting first is hard to reason about — the neighbours keep changing. Bursting *last* in a sub-range is clean: when balloon `k` is the last to go in `(left, right)`, its neighbours at that moment are exactly `left` and `right`, and the two sides `(left, k)` and `(k, right)` were already independently cleared.

## Description

You have `n` balloons with values `nums[i]`. Bursting balloon `i` earns `nums[i-1] * nums[i] * nums[i+1]` coins, where out-of-range neighbours count as 1. After bursting, the neighbours become adjacent. Return the maximum coins.

**Example**

```
Input:  nums = [3,1,5,8]
Output: 167   (burst 1, 5, 3, 8 → 3·1·5 + 3·5·8 + 1·3·8 + 1·8·1)

Input:  nums = [1,5]
Output: 10
```

**Constraints**

- $1 \le n \le 300$
- $0 \le \text{nums}[i] \le 100$

## Prerequisites

- Padding the array with sentinel 1s at both ends.
- Interval DP indexed by open endpoints `(left, right)`, iterating by increasing gap.
- The "which element is handled last" decomposition.

## Approach 1: Interval DP (bottom-up)

### Intuition

Pad to `arr = [1] + nums + [1]`. `dp[l][r]` = max coins from bursting every balloon strictly between `l` and `r`. Try each `k` in `(l, r)` as the last burst: it pays `arr[l] * arr[k] * arr[r]` plus `dp[l][k] + dp[k][r]`.

### Algorithm

1. `arr = [1] + nums + [1]`, `m = len(arr)`. `dp = [[0] * m for _ in range(m)]`.
2. For `length` from `2` to `m - 1` (gap between `l` and `r`): for each `l` with `r = l + length`: `dp[l][r] = max over k in (l+1 .. r-1) of arr[l]*arr[k]*arr[r] + dp[l][k] + dp[k][r]`.
3. Return `dp[0][m - 1]`.

```python
def maxCoins(nums: list[int]) -> int:
    arr = [1] + nums + [1]
    m = len(arr)
    dp = [[0] * m for _ in range(m)]

    for length in range(2, m):
        for l in range(0, m - length):
            r = l + length
            best = 0
            for k in range(l + 1, r):
                coins = arr[l] * arr[k] * arr[r] + dp[l][k] + dp[k][r]
                best = max(best, coins)
            dp[l][r] = best

    return dp[0][m - 1]
```

### Complexity

- **Time:** $O(n^3)$.
- **Space:** $O(n^2)$.

## Approach 2: Recursion with memoization

### Intuition

`f(l, r)` = max coins for the open interval `(l, r)` of the padded array. If `r - l < 2`, nothing to burst → 0. Otherwise try each `k` as the last burst.

### Algorithm

Top-down mirror of Approach 1, cached on `(l, r)`.

```python
from functools import lru_cache

def maxCoins(nums: list[int]) -> int:
    arr = [1] + nums + [1]

    @lru_cache(None)
    def f(l, r):
        if r - l < 2:
            return 0
        best = 0
        for k in range(l + 1, r):
            best = max(best, arr[l] * arr[k] * arr[r] + f(l, k) + f(k, r))
        return best

    return f(0, len(arr) - 1)
```

### Complexity

- **Time:** $O(n^3)$.
- **Space:** $O(n^2)$.

## Common Pitfalls

- **Thinking "first burst" instead of "last burst".** Only the last-burst framing keeps the sub-ranges independent, because the range borders are then fixed.
- **Forgetting the sentinel 1s.** Edge balloons need a neighbour value of 1; padding removes all boundary special-casing.
- **Iterating `l`/`r` in plain order.** `dp[l][r]` needs shorter intervals `dp[l][k]` and `dp[k][r]`, so iterate by increasing interval length.
- **Including `k = l` or `k = r`.** `k` must be strictly inside; `l` and `r` are borders, not balloons to burst here.

## The keystone

Interval DP — "pick which element of `[l, r]` is processed last (or first), splitting into two independent sub-intervals" — is the pattern for problems where an operation reshapes its neighbourhood. Matrix-chain multiplication and optimal BST construction share this exact $O(n^3)$ skeleton.
