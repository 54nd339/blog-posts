---
title: Climbing Stairs - Fibonacci in Disguise
description: Counting ways to climb n stairs taking 1 or 2 steps at a time — the count for step n is the sum of the counts for n minus 1 and n minus 2, computed with two rolling variables.
date: 2022-03-02
draft: false
slug: /dsa/climbing-stairs
tags:
  - Competitive Programming
  - Dynamic Programming
  - Recursion
---

To reach step `n`, your last move was either a 1-step from `n - 1` or a 2-step from `n - 2`. So `ways(n) = ways(n - 1) + ways(n - 2)` — the Fibonacci recurrence. You only ever need the last two values.

## Description

You are climbing a staircase with `n` steps. Each time you can climb 1 or 2 steps. Return the number of distinct ways to reach the top.

**Example**

```
Input:  n = 2
Output: 2   (1+1, 2)

Input:  n = 3
Output: 3   (1+1+1, 1+2, 2+1)
```

**Constraints**

- $1 \le n \le 45$

## Prerequisites

- Identifying an optimal-substructure recurrence.
- Rolling two variables instead of a full array.

## Approach 1: Recursion with memoization

### Intuition

Define `f(i)` = ways to reach step `i`. `f(i) = f(i - 1) + f(i - 2)`, with `f(0) = f(1) = 1`. Cache to avoid the exponential blow-up of the naive tree.

### Algorithm

1. `f(i)`: if `i <= 1`, return `1`. Return cached value if present.
2. Otherwise `f(i) = f(i - 1) + f(i - 2)`; store and return.

```python
from functools import lru_cache

def climbStairs(n: int) -> int:
    @lru_cache(None)
    def f(i):
        if i <= 1:
            return 1
        return f(i - 1) + f(i - 2)

    return f(n)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the cache and recursion.

## Approach 2: Bottom-up array

### Intuition

Fill `dp[0..n]` left to right; each entry is the sum of the two before it.

### Algorithm

1. `dp = [1] * (n + 1)`.
2. For `i` from `2` to `n`: `dp[i] = dp[i - 1] + dp[i - 2]`.
3. Return `dp[n]`.

```python
def climbStairs(n: int) -> int:
    if n <= 1:
        return 1
    dp = [1] * (n + 1)
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Two rolling variables

### Intuition

The array only ever reads `dp[i-1]` and `dp[i-2]`, so keep just those two.

### Algorithm

1. `a, b = 1, 1` (ways to reach step 0 and step 1).
2. Repeat `n - 1` times: `a, b = b, a + b`.
3. Return `b`.

```python
def climbStairs(n: int) -> int:
    a, b = 1, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Base cases.** `f(0) = 1` (the empty climb) and `f(1) = 1`; getting either wrong shifts the whole sequence.
- **Off-by-one in the rolling loop.** Starting from `(1, 1)` as steps 0 and 1, you iterate `n - 1` times to land `b` on step `n`.
- **Recomputing without a cache.** The plain recursion is $O(\varphi^n)$ — always memoize or go bottom-up.
- **Integer overflow.** Not an issue in Python; in fixed-width languages `f(45)` still fits in 32 bits but larger `n` would not.

## The keystone

This is the archetypal DP: a one-line recurrence, two base cases, and a reduction from $O(n)$ space to $O(1)$ by keeping only the window the recurrence reads. [Min Cost Climbing Stairs](/citadel/dsa/min-cost-climbing-stairs) is the same window with a cost to minimise instead of paths to count.
