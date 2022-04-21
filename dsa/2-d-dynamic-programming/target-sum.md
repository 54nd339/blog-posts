---
title: Target Sum - Assign Signs, or Count a Subset
description: Counting ways to put plus or minus before each number to reach a target — a memoized index-and-running-sum recursion, or an algebraic reduction to counting subsets with a fixed sum.
date: 2022-04-21
draft: false
slug: /dsa/target-sum
tags:
  - Competitive Programming
  - Dynamic Programming
  - Backtracking
---

Split the numbers into a positive group `P` and a negative group `N`. Then `sum(P) - sum(N) = target` and `sum(P) + sum(N) = total`, so `sum(P) = (total + target) / 2`. Counting sign assignments becomes counting subsets that sum to that value — a standard knapsack. Or just memoize the sign-choice recursion directly.

## Description

Given an array `nums` of non-negative integers and an integer `target`, count the ways to prefix each number with `+` or `-` so the resulting expression equals `target`.

**Example**

```
Input:  nums = [1,1,1,1,1], target = 3
Output: 5

Input:  nums = [1], target = 1
Output: 1
```

**Constraints**

- $1 \le \text{nums.length} \le 20$
- $0 \le \text{nums}[i] \le 1000$; $0 \le \sum \text{nums} \le 1000$; $-1000 \le \text{target} \le 1000$

## Prerequisites

- The `(total + target) / 2` subset-sum reduction and its parity/range guards.
- Counting knapsack, or memoized recursion on `(index, runningSum)`.

## Approach 1: Recursion with memoization on (index, sum)

### Intuition

`f(i, cur)` = ways to assign signs to `nums[i:]` so the total becomes `target`, given the running sum `cur`. Branch on `+nums[i]` and `-nums[i]`. Cache on `(i, cur)`.

### Algorithm

1. `f(i, cur)`: if `i == n`, return `1` if `cur == target` else `0`.
2. Return `f(i + 1, cur + nums[i]) + f(i + 1, cur - nums[i])`.
3. Memoize on `(i, cur)`.

```python
from functools import lru_cache

def findTargetSumWays(nums: list[int], target: int) -> int:
    n = len(nums)

    @lru_cache(None)
    def f(i, cur):
        if i == n:
            return 1 if cur == target else 0
        return f(i + 1, cur + nums[i]) + f(i + 1, cur - nums[i])

    return f(0, 0)
```

### Complexity

- **Time:** $O(n \cdot S)$ where `S` is the range of possible sums ($2 \cdot \text{total} + 1$).
- **Space:** $O(n \cdot S)$.

## Approach 2: Subset-sum reduction (1-D counting knapsack)

### Intuition

Need `sum(P) = (total + target) / 2 = S`. If `total + target` is odd or `S < 0` or `S > total`, the answer is 0. Otherwise count subsets of `nums` summing to `S`: `dp[s] += dp[s - x]`, `s` descending.

### Algorithm

1. `total = sum(nums)`; if `(total + target) % 2` or `abs(target) > total`, return `0`. `S = (total + target) // 2`.
2. `dp = [0] * (S + 1)`, `dp[0] = 1`.
3. For each `x`: for `s` from `S` down to `x`: `dp[s] += dp[s - x]`.
4. Return `dp[S]`.

```python
def findTargetSumWays(nums: list[int], target: int) -> int:
    total = sum(nums)
    if (total + target) % 2 or abs(target) > total:
        return 0
    S = (total + target) // 2
    dp = [0] * (S + 1)
    dp[0] = 1
    for x in nums:
        for s in range(S, x - 1, -1):
            dp[s] += dp[s - x]
    return dp[S]
```

### Complexity

- **Time:** $O(n \cdot S)$.
- **Space:** $O(S)$.

## Approach 3: 2-D DP on (index, sum)

### Intuition

`dp[i][s]` = ways for the first `i` numbers to reach running sum `s` (offset by `total` so indices stay non-negative). Transition adds and subtracts `nums[i-1]`.

### Algorithm

1. Offset all sums by `total`; width `2*total + 1`.
2. `dp[0][total] = 1`. For each `i`, for each reachable `s`: `dp[i][s+nums[i-1]] += dp[i-1][s]` and `dp[i][s-nums[i-1]] += dp[i-1][s]`.
3. Return `dp[n][target + total]` (0 if out of range).

```python
def findTargetSumWays(nums: list[int], target: int) -> int:
    total = sum(nums)
    if abs(target) > total:
        return 0
    width = 2 * total + 1
    dp = [0] * width
    dp[total] = 1
    for x in nums:
        nxt = [0] * width
        for s in range(width):
            if dp[s]:
                nxt[s + x] += dp[s]
                nxt[s - x] += dp[s]
        dp = nxt
    return dp[target + total]
```

### Complexity

- **Time:** $O(n \cdot \text{total})$.
- **Space:** $O(\text{total})$.

## Common Pitfalls

- **Missing the parity check.** `(total + target)` odd ⇒ no integer subset sum ⇒ answer 0.
- **`abs(target) > total`.** Unreachable; return 0 before building any table.
- **Zeros in `nums`.** A `0` can take `+` or `-` with the same effect, so it doubles the count — the recursion and the knapsack both handle this automatically (`dp[s] += dp[s - 0]` counts it), do not special-case it away.
- **Negative indices in the 2-D form.** Offset sums by `total`.

## The keystone

A "+/- on each element" problem is a subset-sum in disguise: fix what the positive group must total and count subsets. The algebraic reduction — two equations in `sum(P)` and `sum(N)` — is the reusable trick; it is the same move as [Partition Equal Subset Sum](/citadel/dsa/partition-equal-subset-sum) with `target = 0`.
