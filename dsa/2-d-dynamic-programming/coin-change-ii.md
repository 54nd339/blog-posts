---
title: Coin Change II - Count Combinations, Not Permutations
description: Counting the ways to make an amount from unlimited coins where order does not matter — loop coins on the outside and amounts on the inside so each combination is counted once.
date: 2022-04-21
draft: false
slug: /dsa/coin-change-ii
tags:
  - Competitive Programming
  - Dynamic Programming
  - Combinatorics
---

The recurrence is `dp[a] += dp[a - coin]`, but the loop order decides everything. Coins on the *outside* means each coin's contribution is folded in once, so `1+2` and `2+1` are the same combination. Amounts outside would count them separately.

## Description

Given an integer `amount` and an array of distinct `coins`, return the number of combinations that make up `amount`. You have an unlimited supply of each coin. Two combinations differing only in order are the same.

**Example**

```
Input:  amount = 5, coins = [1,2,5]
Output: 4   (5; 2+2+1; 2+1+1+1; 1+1+1+1+1)

Input:  amount = 3, coins = [2]
Output: 0
```

**Constraints**

- $1 \le \text{coins.length} \le 300$; $1 \le \text{coins}[i] \le 5000$; distinct.
- $0 \le \text{amount} \le 5000$

## Prerequisites

- Unbounded knapsack counting.
- Why "coins outer, amount inner" counts combinations and the reverse counts permutations.

## Approach 1: 1-D DP (coins outer loop)

### Intuition

`dp[a]` = ways to make `a` using the coins processed so far. Start `dp[0] = 1`. For each coin, sweep `a` upward adding `dp[a - coin]`; because the coin loop is outside, you never revisit an earlier coin after moving on, so order is fixed.

### Algorithm

1. `dp = [0] * (amount + 1)`, `dp[0] = 1`.
2. For each `coin`: for `a` from `coin` to `amount`: `dp[a] += dp[a - coin]`.
3. Return `dp[amount]`.

```python
def change(amount: int, coins: list[int]) -> int:
    dp = [0] * (amount + 1)
    dp[0] = 1
    for coin in coins:
        for a in range(coin, amount + 1):
            dp[a] += dp[a - coin]
    return dp[amount]
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount})$.

## Approach 2: 2-D DP

### Intuition

`dp[i][a]` = ways to make `a` using only the first `i` coins. Either skip coin `i` (`dp[i-1][a]`) or use at least one copy (`dp[i][a - coins[i-1]]`).

### Algorithm

1. `dp = [[0] * (amount + 1) for _ in range(len(coins) + 1)]`; `dp[i][0] = 1` for all `i`.
2. For `i` from `1` to `len(coins)`, `a` from `1` to `amount`: `dp[i][a] = dp[i-1][a] + (dp[i][a - coins[i-1]] if a >= coins[i-1] else 0)`.
3. Return `dp[len(coins)][amount]`.

```python
def change(amount: int, coins: list[int]) -> int:
    n = len(coins)
    dp = [[0] * (amount + 1) for _ in range(n + 1)]
    for i in range(n + 1):
        dp[i][0] = 1
    for i in range(1, n + 1):
        for a in range(1, amount + 1):
            dp[i][a] = dp[i - 1][a]
            if a >= coins[i - 1]:
                dp[i][a] += dp[i][a - coins[i - 1]]
    return dp[n][amount]
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount} \cdot |\text{coins}|)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i, a)` = ways to make `a` using coins `i..`. Skip coin `i` (`f(i+1, a)`), or take one and stay on `i` (`f(i, a - coins[i])`).

### Algorithm

Top-down mirror of Approach 2, cached on `(i, a)`.

```python
from functools import lru_cache

def change(amount: int, coins: list[int]) -> int:
    n = len(coins)

    @lru_cache(None)
    def f(i, a):
        if a == 0:
            return 1
        if i == n or a < 0:
            return 0
        return f(i + 1, a) + f(i, a - coins[i])

    return f(0, amount)
```

### Complexity

- **Time:** $O(\text{amount} \cdot |\text{coins}|)$.
- **Space:** $O(\text{amount} \cdot |\text{coins}|)$.

## Common Pitfalls

- **Swapping the loops in the 1-D form.** `for a: for coin: dp[a] += dp[a - coin]` counts ordered sequences (permutations), giving a much larger, wrong number.
- **`dp[0]` not 1.** The empty selection is the one way to make 0.
- **Sweeping `a` downward.** Here you *want* reuse, so `a` goes upward (contrast [Partition Equal Subset Sum](/citadel/dsa/partition-equal-subset-sum), which sweeps down to forbid it).
- **Recursion that decrements `i` on "take".** Take must keep `i` (reuse) and skip must advance it; getting this backwards changes what you are counting.

## The keystone

In counting knapsacks, the loop nesting encodes whether order matters: item-outer for combinations, capacity-outer for permutations. Same three lines, opposite meaning. [Target Sum](/citadel/dsa/target-sum) reduces to a counting knapsack after an algebraic rearrangement.
