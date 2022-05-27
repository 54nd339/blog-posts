---
title: Gas Station - If the Total Works, One Start Works
description: Finding a circular route start from which the gas tank never goes negative — if total gas covers total cost, the answer is the index right after the point where the running tank hits its lowest.
date: 2022-05-27
draft: false
slug: /dsa/gas-station
tags:
  - Competitive Programming
  - Greedy
  - Arrays
---

Two facts. First: a full loop is possible iff `sum(gas) >= sum(cost)`. Second: if you run the tank from station 0 and it goes negative at station `i`, then no station in `0..i` can be the start — so jump the candidate to `i + 1`. One pass settles it.

## Description

There are `n` gas stations in a circle. `gas[i]` is the fuel at station `i`; `cost[i]` is the fuel needed to drive from `i` to `i + 1`. Starting with an empty tank, return the starting station index from which you can complete the circuit once, or `-1`. The answer is unique if it exists.

**Example**

```
Input:  gas = [1,2,3,4,5], cost = [3,4,5,1,2]
Output: 3

Input:  gas = [2,3,4], cost = [3,4,3]
Output: -1
```

**Constraints**

- $1 \le n \le 10^5$
- $0 \le \text{gas}[i], \text{cost}[i] \le 10^4$

## Prerequisites

- The feasibility condition `sum(gas) >= sum(cost)`.
- The greedy jump: a prefix that dips negative contains no valid start.

## Approach 1: One-pass greedy

### Intuition

Track `total` (overall balance, decides feasibility) and `tank` (running balance from the current candidate start). Whenever `tank` drops below 0 after station `i`, reset `tank = 0` and set the candidate start to `i + 1`.

### Algorithm

1. `total = 0`, `tank = 0`, `start = 0`.
2. For each `i`: `diff = gas[i] - cost[i]`; `total += diff`; `tank += diff`. If `tank < 0`: `start = i + 1`; `tank = 0`.
3. Return `start` if `total >= 0` else `-1`.

```python
def canCompleteCircuit(gas: list[int], cost: list[int]) -> int:
    total = tank = 0
    start = 0
    for i in range(len(gas)):
        diff = gas[i] - cost[i]
        total += diff
        tank += diff
        if tank < 0:
            start = i + 1
            tank = 0
    return start if total >= 0 else -1
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Brute force

### Intuition

Try each station as a start; simulate the full loop, bailing when the tank goes negative.

### Algorithm

1. For each `start` in `range(n)`: `tank = 0`; for `k` in `range(n)`, `i = (start + k) % n`, `tank += gas[i] - cost[i]`; break if `tank < 0`. If the loop finished, return `start`.
2. Return `-1`.

```python
def canCompleteCircuit(gas: list[int], cost: list[int]) -> int:
    n = len(gas)
    for start in range(n):
        tank = 0
        for k in range(n):
            i = (start + k) % n
            tank += gas[i] - cost[i]
            if tank < 0:
                break
        else:
            return start
    return -1
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Resetting `total` along with `tank`.** `total` must accumulate over the *whole* array to test global feasibility; only `tank` resets.
- **Setting `start = i` instead of `i + 1`.** Station `i` is exactly where the tank failed, so it cannot be the start either.
- **Returning `start` without the feasibility check.** The greedy pointer lands on a plausible index even when no solution exists; guard with `total >= 0`.
- **Off-by-one when `i + 1 == n`.** The final failing station gives `start = n`; that only occurs when `total < 0`, so the `-1` branch catches it.

## The keystone

"A negative prefix cannot contain the answer, so skip past it" is the same abandonment rule as Kadane in [Maximum Subarray](/citadel/dsa/maximum-subarray). Pair it with a global sum check and a two-value one-pass replaces an $O(n^2)$ simulation.
