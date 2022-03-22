---
title: Coin Change - Fewest Coins to Make an Amount
description: The minimum number of coins summing to a target from unlimited supplies of given denominations — an unbounded-knapsack DP where the best for amount a is one plus the best for a minus each coin.
date: 2022-03-22
draft: false
slug: /dsa/coin-change
tags:
  - Competitive Programming
  - Dynamic Programming
  - Breadth-First Search
---

The fewest coins for amount `a` is `1 + min over coins c of dp[a - c]`. Build `dp[0..amount]` upward; each entry tries every coin. Unreachable amounts stay at infinity, and `dp[amount] == inf` means "impossible".

## Description

Given an array `coins` and an integer `amount`, return the fewest coins needed to make up `amount`, or `-1` if it cannot be made. You have an unlimited number of each coin.

**Example**

```
Input:  coins = [1,2,5], amount = 11
Output: 3   (5 + 5 + 1)

Input:  coins = [2], amount = 3
Output: -1
```

**Constraints**

- $1 \le \text{coins.length} \le 12$; $1 \le \text{coins}[i] \le 2^{31} - 1$
- $0 \le \text{amount} \le 10^4$

## Prerequisites

- Unbounded knapsack: a coin may be reused, so iterate amounts outward.
- BFS view: amounts are nodes, each coin is an edge of weight 1.

## Approach 1: Bottom-up DP

### Intuition

`dp[a]` = fewest coins for amount `a`. `dp[0] = 0`. For each `a` from 1 to `amount`, try every coin `c <= a`: `dp[a] = min(dp[a], dp[a - c] + 1)`.

### Algorithm

1. `dp = [inf] * (amount + 1)`, `dp[0] = 0`.
2. For `a` from `1` to `amount`: for each `c` in `coins` with `c <= a`, `dp[a] = min(dp[a], dp[a - c] + 1)`.
3. Return `dp[amount]` if finite else `-1`.

```python
def coinChange(coins: list[int], amount: int) -> int:
    dp = [float("inf")] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != float("inf") else -1
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount})$.

## Approach 2: BFS over amounts

### Intuition

Level 0 is `{amount}` (or `{0}`). Each level subtracts one coin. The first level at which you hit 0 is the coin count — BFS gives the minimum for free.

### Algorithm

1. `q = deque([amount])`, `visited = {amount}`, `steps = 0`.
2. While `q`: for each of `len(q)` values `v`: if `v == 0`, return `steps`. For each coin `c` with `v - c >= 0` and `v - c` unvisited, enqueue and mark.
3. `steps += 1`. Return `-1` if the queue drains.

```python
from collections import deque

def coinChange(coins: list[int], amount: int) -> int:
    if amount == 0:
        return 0
    q = deque([amount])
    visited = {amount}
    steps = 0
    while q:
        steps += 1
        for _ in range(len(q)):
            v = q.popleft()
            for c in coins:
                nxt = v - c
                if nxt == 0:
                    return steps
                if nxt > 0 and nxt not in visited:
                    visited.add(nxt)
                    q.append(nxt)
    return -1
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount})$.

## Approach 3: Recursion with memoization

### Intuition

`f(a)` = fewest coins for `a`. `f(0) = 0`; `f(a) = 1 + min(f(a - c))` over coins with `a - c >= 0`; `inf` if none work.

### Algorithm

Top-down mirror of Approach 1, cached on `a`.

```python
from functools import lru_cache

def coinChange(coins: list[int], amount: int) -> int:
    @lru_cache(None)
    def f(a):
        if a == 0:
            return 0
        best = float("inf")
        for c in coins:
            if c <= a:
                best = min(best, f(a - c) + 1)
        return best

    res = f(amount)
    return res if res != float("inf") else -1
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount})$.

## Common Pitfalls

- **Greedy by largest coin.** Fails for `coins = [1,3,4], amount = 6` (greedy gives 4+1+1, optimal is 3+3).
- **`dp[0]` not set to 0.** It is the base of every chain.
- **Returning `dp[amount]` without the infinity check.** An unreachable amount must yield `-1`.
- **BFS without a visited set.** Amounts get re-enqueued from many predecessors, blowing up the queue.

## The keystone

"Fewest items to hit a target, unlimited reuse" is unbounded knapsack, and it doubles as an unweighted shortest-path problem on the number line — which is why BFS also solves it. [Coin Change II](/citadel/dsa/coin-change-ii) keeps the coins but counts *combinations* instead of minimising, which changes the loop order.
