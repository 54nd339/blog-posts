---
title: Best Time to Buy and Sell Stock With Cooldown - A Three-State Machine
description: Maximising profit from unlimited trades with a one-day cooldown after selling — track three running states, holding, just-sold, and resting, and the best value transitioning between them each day.
date: 2022-04-12
draft: false
slug: /dsa/best-time-to-buy-and-sell-stock-with-cooldown
tags:
  - Competitive Programming
  - Dynamic Programming
  - State Machine
---

Each day you are in one of three states: **holding** a share, **just sold** today (so tomorrow is a forced cooldown), or **resting** with no share and free to buy. Write the best achievable profit in each state as a function of yesterday's states, and sweep the prices once.

## Description

Given an array `prices` where `prices[i]` is the stock price on day `i`, find the maximum profit with as many transactions as you like, subject to: you cannot hold more than one share, and after you sell you must skip the next day (cooldown).

**Example**

```
Input:  prices = [1,2,3,0,2]
Output: 3   (buy, sell, cooldown, buy, sell)

Input:  prices = [1]
Output: 0
```

**Constraints**

- $1 \le \text{prices.length} \le 5000$
- $0 \le \text{prices}[i] \le 1000$

## Prerequisites

- Modelling trading decisions as a finite state machine.
- Rolling three scalars, one per state.

## Approach 1: State-machine DP (three rolling variables)

### Intuition

Let `hold` = best profit while currently holding, `sold` = best profit having sold *today*, `rest` = best profit resting (not holding, did not sell today). Transitions per day at price `p`:

- `hold = max(hold, rest - p)` — keep holding, or buy today (only from `rest`, respecting cooldown).
- `sold = hold + p` — sell today.
- `rest = max(rest, sold)` — stay resting, or come off cooldown.

### Algorithm

1. `hold = -inf`, `sold = 0`, `rest = 0`.
2. For each `p` in `prices`: compute the three new values from the old ones (snapshot `sold` before overwriting).
3. Return `max(sold, rest)`.

```python
def maxProfit(prices: list[int]) -> int:
    hold, sold, rest = float("-inf"), 0, 0
    for p in prices:
        prev_sold = sold
        sold = hold + p
        hold = max(hold, rest - p)
        rest = max(rest, prev_sold)
    return max(sold, rest)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Recursion with memoization

### Intuition

`f(i, holding)` = best profit from day `i` onward given whether you hold a share. Options: do nothing → `f(i+1, holding)`; if holding, sell → `prices[i] + f(i+2, False)` (skip the cooldown day); if not holding, buy → `-prices[i] + f(i+1, True)`.

### Algorithm

1. `f(i, holding)`: if `i >= n`, return `0`.
2. `res = f(i + 1, holding)` (rest).
3. If `holding`: `res = max(res, prices[i] + f(i + 2, False))`.
4. Else: `res = max(res, -prices[i] + f(i + 1, True))`.
5. Memoize on `(i, holding)`.

```python
from functools import lru_cache

def maxProfit(prices: list[int]) -> int:
    n = len(prices)

    @lru_cache(None)
    def f(i, holding):
        if i >= n:
            return 0
        res = f(i + 1, holding)
        if holding:
            res = max(res, prices[i] + f(i + 2, False))
        else:
            res = max(res, -prices[i] + f(i + 1, True))
        return res

    return f(0, False)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Buying directly after selling.** The `hold` transition must draw from `rest`, never from `sold`, or you skip the cooldown.
- **Updating states in the wrong order.** `sold` depends on the *old* `hold`; `rest` depends on the *old* `sold`. Snapshot before you overwrite.
- **Cooldown as `i + 1` instead of `i + 2` in the recursion.** Selling on day `i` blocks day `i + 1`; the next free day is `i + 2`.
- **Returning `hold`.** The optimum never ends while holding a share; return `max(sold, rest)`.

## The keystone

When choices have a small set of modes and legal transitions between them, the DP state *is* the mode — three variables, a fixed transition table, one linear sweep. This state-machine view scales to the "at most `k` transactions" stock problems and any "cooldown / fee / limit" variant.
