---
title: Maximum Subarray - Kadane Drops the Negative Prefix
description: The largest sum of a contiguous subarray — extend the running sum, but reset it to the current element the moment it goes negative, since a negative prefix can only hurt what follows.
date: 2022-05-24
draft: false
slug: /dsa/maximum-subarray
tags:
  - Competitive Programming
  - Greedy
  - Divide and Conquer
---

A running sum that has gone negative is dead weight — any later subarray is better off starting fresh. So carry `cur`, the best sum of a subarray *ending here*: `cur = max(x, cur + x)`. The answer is the largest `cur` ever seen.

## Description

Given an integer array `nums`, find the contiguous non-empty subarray with the largest sum and return that sum.

**Example**

```
Input:  nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6   ([4,-1,2,1])

Input:  nums = [5,4,-1,7,8]
Output: 23
```

**Constraints**

- $1 \le \text{nums.length} \le 10^5$
- $-10^4 \le \text{nums}[i] \le 10^4$

## Prerequisites

- The greedy insight: discard a negative running prefix.
- (For the third approach) merging left / right / cross-boundary maxima.

## Approach 1: Kadane's algorithm

### Intuition

`cur` = max sum of a subarray ending at the current index. Either extend the previous one (`cur + x`) or start over at `x`. Track the global max.

### Algorithm

1. `cur = best = nums[0]`.
2. For each `x` in `nums[1:]`: `cur = max(x, cur + x)`; `best = max(best, cur)`.
3. Return `best`.

```python
def maxSubArray(nums: list[int]) -> int:
    cur = best = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Prefix-sum minimum

### Intuition

The max subarray ending at `i` is `prefix[i] - min(prefix[0..i-1])`. Sweep the running prefix sum while tracking its smallest value so far.

### Algorithm

1. `prefix = 0`, `min_prefix = 0`, `best = -inf`.
2. For each `x`: `prefix += x`; `best = max(best, prefix - min_prefix)`; `min_prefix = min(min_prefix, prefix)`.
3. Return `best`.

```python
def maxSubArray(nums: list[int]) -> int:
    prefix = 0
    min_prefix = 0
    best = float("-inf")
    for x in nums:
        prefix += x
        best = max(best, prefix - min_prefix)
        min_prefix = min(min_prefix, prefix)
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Divide and conquer

### Intuition

Split in half. The best subarray is entirely in the left half, entirely in the right half, or crosses the midpoint. The crossing one is the best suffix of the left plus the best prefix of the right.

### Algorithm

1. `solve(lo, hi)`: if `lo == hi`, return `nums[lo]`.
2. `mid = (lo + hi) // 2`. Recurse left and right.
3. Compute the max suffix of `[lo, mid]` and max prefix of `[mid+1, hi]`; their sum is the cross value.
4. Return `max(left, right, cross)`.

```python
def maxSubArray(nums: list[int]) -> int:
    def solve(lo, hi):
        if lo == hi:
            return nums[lo]
        mid = (lo + hi) // 2
        left = solve(lo, mid)
        right = solve(mid + 1, hi)

        s = 0
        best_suffix = float("-inf")
        for i in range(mid, lo - 1, -1):
            s += nums[i]
            best_suffix = max(best_suffix, s)

        s = 0
        best_prefix = float("-inf")
        for i in range(mid + 1, hi + 1):
            s += nums[i]
            best_prefix = max(best_prefix, s)

        return max(left, right, best_suffix + best_prefix)

    return solve(0, len(nums) - 1)
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(\log n)$ recursion.

## Common Pitfalls

- **Initialising `best` to 0.** An all-negative array like `[-3, -1, -2]` must return `-1`; seed with `nums[0]` or `-inf`.
- **`cur = max(0, cur + x)`.** That allows an empty subarray; the problem requires non-empty, so it is `max(x, cur + x)`.
- **Prefix-sum version starting `min_prefix` at the first element.** It starts at 0 (the empty prefix) so a subarray from index 0 is covered.
- **Divide and conquer forgetting the cross case.** The optimal subarray often straddles the midpoint.

## The keystone

Kadane is the definitional greedy scan: a running quantity, and a rule for when to abandon it. "Drop the prefix that can only hurt you" recurs in [Gas Station](/citadel/dsa/gas-station), where a failed start means every station up to the failure point is a dead start too.
