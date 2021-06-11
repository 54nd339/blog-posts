---
title: Best Time to Buy and Sell Stock - Carry the Minimum So Far
description: Maximising profit from one buy and one later sell — checking every pair is quadratic, but a single pass that remembers the cheapest price so far finds the best sell in linear time.
date: 2021-06-11
draft: false
slug: /dsa/best-time-to-buy-and-sell-stock
tags:
  - Competitive Programming
  - Sliding Window
  - Dynamic Programming
---

The linear idea: for each day taken as the *sell* day, the best profit is that price minus the cheapest price on any earlier day — and the cheapest earlier price is a running minimum.

## Description

Given `prices`, where `prices[i]` is a stock's price on day `i`, you may buy on one day and sell on a later day. Return the maximum profit, or `0` if no profitable trade exists.

**Example**

```
Input:  prices = [7, 1, 5, 3, 6, 4]
Output: 5          # buy at 1, sell at 6

Input:  prices = [7, 6, 4, 3, 1]
Output: 0
```

**Constraints**

- $1 \le \text{prices.length} \le 10^5$
- $0 \le \text{prices}[i] \le 10^4$

## Prerequisites

- The idea of a prefix extreme (minimum so far) queried in $O(1)$.
- [Sliding window](/citadel/dsa/sliding-window) with a left edge pinned to the best-so-far position.

## Approach 1: Brute Force

### Intuition

Try every buy day against every later sell day.

### Algorithm

1. `best = 0`.
2. For each `i`, for each `j > i`, update `best` with `prices[j] - prices[i]`.
3. Return `best`.

```python
def maxProfit(prices: list[int]) -> int:
    best = 0
    n = len(prices)
    for i in range(n):
        for j in range(i + 1, n):
            best = max(best, prices[j] - prices[i])
    return best
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Two Pointers

### Intuition

`left` is the buy day, `right` the sell day. If `prices[right] > prices[left]`, book the profit. If not, `right` is a cheaper buy than `left`, so jump `left` to `right`.

### Algorithm

1. `left = 0`, `right = 1`, `best = 0`.
2. While `right < n`:
   - If `prices[right] > prices[left]`, update `best` with `prices[right] - prices[left]`.
   - Else set `left = right`.
   - `right += 1`.
3. Return `best`.

```python
def maxProfit(prices: list[int]) -> int:
    left, right = 0, 1
    best = 0
    while right < len(prices):
        if prices[right] > prices[left]:
            best = max(best, prices[right] - prices[left])
        else:
            left = right
        right += 1
    return best
```

### Complexity

- **Time:** $O(n)$ — `right` sweeps once.
- **Space:** $O(1)$.

## Approach 3: Dynamic Programming

### Intuition

Track only two things: the lowest price seen so far, and the best profit achievable if you sell today.

### Algorithm

1. `min_price = prices[0]`, `best = 0`.
2. For each `price`: update `best` with `price - min_price`, then update `min_price` with `price`.
3. Return `best`.

```python
def maxProfit(prices: list[int]) -> int:
    min_price = prices[0]
    best = 0
    for price in prices:
        best = max(best, price - min_price)
        min_price = min(min_price, price)
    return best
```

### Complexity

- **Time:** $O(n)$ — one pass.
- **Space:** $O(1)$.

## Common Pitfalls

- **Returning a negative profit.** The floor is `0` — you may decline to trade. Initialising `best` to `prices[1] - prices[0]` breaks on a strictly decreasing array.
- **Buying and selling on the same day.** In Approach 3, `price - min_price` can be `0` on the day you set a new low; that is fine (contributes nothing), but selling *before* buying must be impossible — updating `min_price` after the profit check guarantees it.
- **Approach 2: not jumping `left`.** When `prices[right] <= prices[left]`, `left` must move to `right`; leaving it behind keeps computing against a price you would never have bought at.

## The keystone

When "the best partner for position `j` among all earlier `i`" is a prefix extreme, carry the running extreme and query it in $O(1)$ — a sliding window whose left edge is pinned to the best-so-far position. The same idea underlies [Maximum Subarray](/citadel/dsa/maximum-subarray).
