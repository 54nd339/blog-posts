---
title: Min Cost Climbing Stairs - Cheapest Way to the Top
description: The minimum cost to reach the top of a staircase where each step charges a toll — the cost to stand on step i is its toll plus the cheaper of the two steps you could have come from.
date: 2022-03-02
draft: false
slug: /dsa/min-cost-climbing-stairs
tags:
  - Competitive Programming
  - Dynamic Programming
  - Arrays
---

The recurrence is [Climbing Stairs](/citadel/dsa/climbing-stairs) with `min` instead of `+`: the cheapest way to *pay your way onto* step `i` is `cost[i]` plus the cheaper of reaching `i - 1` or `i - 2`. The top is one step past the last index, and it charges nothing.

## Description

Given an array `cost` where `cost[i]` is the fee to step off stair `i`, you may start at index 0 or 1 and each move climbs 1 or 2 stairs. Return the minimum total cost to reach the top (just past the last stair).

**Example**

```
Input:  cost = [10,15,20]
Output: 15   (start at index 1, pay 15, jump 2 to the top)

Input:  cost = [1,100,1,1,1,100,1,1,100,1]
Output: 6
```

**Constraints**

- $2 \le \text{cost.length} \le 1000$
- $0 \le \text{cost}[i] \le 999$

## Prerequisites

- The two-predecessor recurrence with `min`.
- The "top" is index `n`, with zero cost.

## Approach 1: Bottom-up array

### Intuition

`dp[i]` = minimum cost to *reach* step `i` (before paying its toll). `dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`. Reaching index 0 or 1 is free, so `dp[0] = dp[1] = 0`. The answer is `dp[n]`.

### Algorithm

1. `dp = [0] * (n + 1)`.
2. For `i` from `2` to `n`: `dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])`.
3. Return `dp[n]`.

```python
def minCostClimbingStairs(cost: list[int]) -> int:
    n = len(cost)
    dp = [0] * (n + 1)
    for i in range(2, n + 1):
        dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])
    return dp[n]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 2: Two rolling variables

### Intuition

`dp[i]` reads only `dp[i-1]` and `dp[i-2]`; keep those two.

### Algorithm

1. `one, two = 0, 0` — cost to reach the last two positions.
2. For `i` from `2` to `n`: `cur = min(one + cost[i - 1], two + cost[i - 2])`; shift `two, one = one, cur`.
3. Return `one`.

```python
def minCostClimbingStairs(cost: list[int]) -> int:
    n = len(cost)
    two, one = 0, 0
    for i in range(2, n + 1):
        cur = min(one + cost[i - 1], two + cost[i - 2])
        two, one = one, cur
    return one
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i)` = min cost to reach index `i`. `f(0) = f(1) = 0`; `f(i) = min(f(i-1) + cost[i-1], f(i-2) + cost[i-2])`. Cache on `i`.

### Algorithm

Top-down version of Approach 1.

```python
from functools import lru_cache

def minCostClimbingStairs(cost: list[int]) -> int:
    n = len(cost)

    @lru_cache(None)
    def f(i):
        if i <= 1:
            return 0
        return min(f(i - 1) + cost[i - 1], f(i - 2) + cost[i - 2])

    return f(n)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Stopping at the last index instead of the top.** The goal is index `n`, one past the array; `dp` has `n + 1` entries.
- **Charging the toll on arrival rather than departure.** `cost[i]` is paid to *leave* step `i`; model it as added when you move *from* `i`.
- **Wrong base case.** Both `dp[0]` and `dp[1]` are 0 — you may start on either for free.
- **Confusing "cost to reach `i`" with "cost including `i`'s toll".** Pick one definition and keep the recurrence consistent.

## The keystone

Swap the accumulation operator and a counting DP becomes an optimisation DP — same window, same $O(1)$ space trick. [House Robber](/citadel/dsa/house-robber) is this recurrence again, with a "can't take adjacent" twist.
