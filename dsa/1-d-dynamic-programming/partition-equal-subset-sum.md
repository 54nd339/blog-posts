---
title: Partition Equal Subset Sum - A Boolean Knapsack for Half the Total
description: Whether an array splits into two equal-sum halves — reachable-sum DP asking if any subset totals half the array sum, as a boolean set, a 1-D table, or a bitset.
date: 2022-03-30
draft: false
slug: /dsa/partition-equal-subset-sum
tags:
  - Competitive Programming
  - Dynamic Programming
  - Bit Manipulation
---

If the total is odd, no split exists. Otherwise the question is: can some subset sum to `total // 2`? That is subset-sum — a 0/1 knapsack with a boolean "reachable" instead of a value. Track the set of achievable sums as you consider each number.

## Description

Given an array `nums` of positive integers, return `true` if it can be partitioned into two subsets with equal sums.

**Example**

```
Input:  nums = [1,5,11,5]
Output: true   ([1,5,5] and [11])

Input:  nums = [1,2,3,5]
Output: false
```

**Constraints**

- $1 \le \text{nums.length} \le 200$
- $1 \le \text{nums}[i] \le 100$

## Prerequisites

- Reduce "equal partition" to "subset summing to total/2".
- 0/1 knapsack reachability; iterate sums downward for the 1-D form.

## Approach 1: Reachable-sum set

### Intuition

Maintain a set of sums achievable so far, starting `{0}`. For each number `x`, add `x` to every existing sum. If `target` ever appears, return `True`.

### Algorithm

1. `total = sum(nums)`; if odd, return `False`. `target = total // 2`.
2. `sums = {0}`. For each `x`: `sums |= {s + x for s in sums if s + x <= target}`. If `target in sums`, return `True`.
3. Return `target in sums`.

```python
def canPartition(nums: list[int]) -> bool:
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    sums = {0}
    for x in nums:
        nxt = set(sums)
        for s in sums:
            if s + x == target:
                return True
            if s + x < target:
                nxt.add(s + x)
        sums = nxt
    return target in sums
```

### Complexity

- **Time:** $O(n \cdot \text{target})$.
- **Space:** $O(\text{target})$.

## Approach 2: 1-D boolean DP

### Intuition

`dp[s]` = "sum `s` is reachable". For each `x`, update `s` from `target` down to `x`: `dp[s] |= dp[s - x]`. Descending order stops one number being used twice.

### Algorithm

1. `total` even check; `target = total // 2`.
2. `dp = [False] * (target + 1)`, `dp[0] = True`.
3. For each `x`: for `s` from `target` down to `x`: `dp[s] = dp[s] or dp[s - x]`.
4. Return `dp[target]`.

```python
def canPartition(nums: list[int]) -> bool:
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    for x in nums:
        for s in range(target, x - 1, -1):
            if dp[s - x]:
                dp[s] = True
    return dp[target]
```

### Complexity

- **Time:** $O(n \cdot \text{target})$.
- **Space:** $O(\text{target})$.

## Approach 3: Bitset

### Intuition

Represent reachable sums as the bits of one big integer: bit `s` set means sum `s` is reachable. Adding `x` is `bits |= bits << x`. Check bit `target` at the end.

### Algorithm

1. Even check; `target = total // 2`.
2. `bits = 1` (only sum 0). For each `x`: `bits |= bits << x`.
3. Return whether bit `target` of `bits` is set (mask to `target` bits to bound the size).

```python
def canPartition(nums: list[int]) -> bool:
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    bits = 1
    mask = (1 << (target + 1)) - 1
    for x in nums:
        bits = (bits | (bits << x)) & mask
    return (bits >> target) & 1 == 1
```

### Complexity

- **Time:** $O(n \cdot \text{target} / 64)$ with machine-word shifts.
- **Space:** $O(\text{target} / 64)$.

## Common Pitfalls

- **Skipping the odd-total short circuit.** An odd sum can never split evenly.
- **Iterating `s` upward in the 1-D DP.** Ascending order lets `x` be reused (that would be *unbounded* knapsack); go from `target` down to `x`.
- **`dp[0]` not `True`.** The empty subset sums to 0 and seeds everything.
- **Not capping sums at `target`.** Without the bound the reachable set (or bitset) grows to the full total, doubling the work.

## The keystone

"Can a subset hit exactly `T`" is boolean 0/1 knapsack; the 1-D descending update and the bitset shift are the two standard ways to run it fast. [Target Sum](/citadel/dsa/target-sum) is the same subset-sum after rearranging `+`/`-` assignments into "pick a subset that sums to `(total + target) / 2`".
