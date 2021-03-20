---
title: 0/1 Knapsack - Maximizing Value Under a Weight Limit with DP
description: Pick a subset of weighted, valued items to maximize value without exceeding a capacity, taking each item whole or not at all. The obvious approach tries every subset; dynamic programming solves it in O(nW) with a table of subproblems.
date: 2021-03-20
draft: false
slug: /algorithms/01Kanpsack
tags:
  - Algorithms
  - Dynamic Programming
---

You have a knapsack that holds `W` kilograms and `n` items, each with a weight and a value. Take each item whole or leave it — no fractions. Which subset maximizes total value without going over `W`? The same shape shows up in budgeting, scheduling, and cutting-stock problems.

The obvious method is to try every subset and keep the best legal one. There are $2^n$ subsets, so that dies around $n = 40$. Dynamic programming brings it down to $O(nW)$ by noticing the subproblems overlap.

## The problem

Given $n$ items where item $i$ has weight $w_i$ and value $v_i$, and a capacity $W$, choose a subset $S$ maximizing $\sum_{i \in S} v_i$ subject to $\sum_{i \in S} w_i \le W$. Each item is in or out — that "0/1" is what makes it hard; allowing fractions makes it [a greedy problem instead](/citadel/algorithms/FractionalKnapsack).

## The idea

Consider the items one at a time. For item $i$ at a remaining capacity $w$, there are exactly two choices: skip it, or take it (if it fits) and recurse on the rest with capacity $w - w_i$. Both branches reduce to "best value from the first $i-1$ items at some capacity" — and across the whole recursion those (item index, capacity) pairs repeat constantly. Cache them.

Let `dp[i][w]` be the best value using the first `i` items with capacity `w`. Then:

$$dp[i][w] = \begin{cases} 0 & i = 0 \text{ or } w = 0 \\ dp[i-1][w] & w_i > w \\ \max\big(dp[i-1][w],\; v_i + dp[i-1][w - w_i]\big) & \text{otherwise} \end{cases}$$

The last line is the whole algorithm: take the better of skipping item $i$ or taking it.

## The code

Bottom-up, filling the table in order:

```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        wi, vi = weights[i - 1], values[i - 1]
        for w in range(capacity + 1):
            dp[i][w] = dp[i - 1][w]                      # skip item i
            if wi <= w:
                dp[i][w] = max(dp[i][w], vi + dp[i - 1][w - wi])  # or take it
    return dp[n][capacity]


assert knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7) == 9      # items 2 and 3
```

Top-down with memoization mirrors the recurrence directly:

```python
from functools import lru_cache

def knapsack_memo(weights, values, capacity):
    @lru_cache(maxsize=None)
    def best(i, w):
        if i == 0 or w == 0:
            return 0
        if weights[i - 1] > w:
            return best(i - 1, w)
        return max(best(i - 1, w),
                   values[i - 1] + best(i - 1, w - weights[i - 1]))
    return best(len(weights), capacity)
```

Each row of `dp` only depends on the row above it, so you can drop to a single array of length `W + 1` and iterate `w` **downward** (so `dp[w - wi]` still holds the previous row's value). That gives $O(W)$ space when you only need the number, not the chosen set.

## Recovering which items

The table holds the best *value*, but often you want the actual subset. Keep the full `dp[i][w]` table and walk it backwards from `dp[n][W]`: at each item, if the value changed when item `i` was added, it must have been taken.

```python
def knapsack_items(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        wi, vi = weights[i - 1], values[i - 1]
        for w in range(capacity + 1):
            dp[i][w] = dp[i - 1][w]
            if wi <= w and vi + dp[i - 1][w - wi] > dp[i][w]:
                dp[i][w] = vi + dp[i - 1][w - wi]

    chosen, w = [], capacity
    for i in range(n, 0, -1):
        if dp[i][w] != dp[i - 1][w]:          # item i-1 was taken
            chosen.append(i - 1)
            w -= weights[i - 1]
    chosen.reverse()
    return dp[n][capacity], chosen

assert knapsack_items([1, 3, 4, 5], [1, 4, 5, 7], 7) == (9, [1, 2])
```

The rolling-array version loses this — reconstruction needs the per-item rows, so it costs $O(nW)$ space again.

## Unbounded knapsack

If each item can be taken **any number of times**, one line changes: when you take item `i` you may take it again, so the recurrence looks at the *current* row instead of the one above, and `w` is iterated **upward**.

```python
def unbounded_knapsack(weights, values, capacity):
    dp = [0] * (capacity + 1)
    for w in range(1, capacity + 1):
        for i in range(len(weights)):
            if weights[i] <= w:
                dp[w] = max(dp[w], values[i] + dp[w - weights[i]])
    return dp[capacity]
```

This is the "coin change / rod cutting" shape: same $O(nW)$, but a single array suffices because repetition is allowed. The 0/1 version's downward sweep is precisely what *forbids* reusing an item within one row.

## Cost

Both versions solve each `(i, w)` subproblem once, and there are $n(W+1)$ of them, each $O(1)$ work: **$O(nW)$ time**. Space is $O(nW)$ for the full table, $O(W)$ with the rolling array, plus $O(n)$ recursion stack for the memoized version.

## The catch: pseudo-polynomial

$O(nW)$ looks polynomial, but $W$ is a *value* in the input, not a count of things. Writing $W$ takes only $\log W$ bits, so the running time is exponential in the input *size*. That's why 0/1 knapsack is [NP-complete](/citadel/algorithms/ComplexityClasses) despite this clean table — the algorithm is fast only when $W$ is small. Change the rules to allow fractional items and the difficulty vanishes: [greedy on value-to-weight ratio](/citadel/algorithms/FractionalKnapsack) is optimal and runs in $O(n \log n)$. The same DP table technique drives [longest common subsequence](/citadel/algorithms/LongestCommonSubsequence) and [matrix chain multiplication](/citadel/algorithms/MatrixChainMultiplication).
