---
title: Maximum Product Subarray - Carry the Min Along With the Max
description: The largest product of a contiguous subarray — because a negative flips large and small, track both the max and min product ending at each index and swap them on a negative element.
date: 2022-03-22
draft: false
slug: /dsa/maximum-product-subarray
tags:
  - Competitive Programming
  - Dynamic Programming
  - Arrays
---

Unlike a sum, a product can leap from very negative to very positive when multiplied by a negative number. So at each index keep *both* the largest and smallest product of a subarray ending there; the new max is the best of `x`, `x·prev_max`, `x·prev_min`.

## Description

Given an integer array `nums`, return the maximum product of a contiguous non-empty subarray. The answer fits in a 32-bit integer.

**Example**

```
Input:  nums = [2,3,-2,4]
Output: 6   ([2,3])

Input:  nums = [-2,0,-1]
Output: 0
```

**Constraints**

- $1 \le \text{nums.length} \le 2 \cdot 10^4$
- $-10 \le \text{nums}[i] \le 10$; products fit in 32 bits.

## Prerequisites

- The insight that a negative multiplier swaps the roles of running max and min.
- Handling zeros, which reset both running products.

## Approach 1: Track running max and min

### Intuition

`cur_max` / `cur_min` = largest / smallest product of a subarray ending at the current index. For each `x`, the candidates are `x`, `cur_max * x`, `cur_min * x`. Take max and min of those. Update a global answer.

### Algorithm

1. `best = cur_max = cur_min = nums[0]`.
2. For each `x` in `nums[1:]`: compute `a = cur_max * x`, `b = cur_min * x`; `cur_max = max(x, a, b)`, `cur_min = min(x, a, b)`; `best = max(best, cur_max)`.
3. Return `best`.

```python
def maxProduct(nums: list[int]) -> int:
    best = cur_max = cur_min = nums[0]
    for x in nums[1:]:
        a, b = cur_max * x, cur_min * x
        cur_max = max(x, a, b)
        cur_min = min(x, a, b)
        best = max(best, cur_max)
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Prefix and suffix products

### Intuition

The maximum-product subarray always touches one end of a "block" delimited by zeros. Sweep a running prefix product left to right and a running suffix product right to left, resetting to 1 after a zero; the answer is the max value either sweep produces.

### Algorithm

1. `prefix = suffix = 0`, `best = nums[0]`.
2. For `i` in `range(n)`: `prefix = (prefix or 1) * nums[i]`; `suffix = (suffix or 1) * nums[n - 1 - i]`; `best = max(best, prefix, suffix)`.
3. Return `best`.

```python
def maxProduct(nums: list[int]) -> int:
    n = len(nums)
    best = nums[0]
    prefix = suffix = 0
    for i in range(n):
        prefix = (prefix or 1) * nums[i]
        suffix = (suffix or 1) * nums[n - 1 - i]
        best = max(best, prefix, suffix)
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Brute force

### Intuition

Try every subarray, multiplying as you extend the right end.

### Algorithm

1. For each start `i`: `p = 1`; for each `j >= i`: `p *= nums[j]`; `best = max(best, p)`.

```python
def maxProduct(nums: list[int]) -> int:
    best = nums[0]
    for i in range(len(nums)):
        p = 1
        for j in range(i, len(nums)):
            p *= nums[j]
            best = max(best, p)
    return best
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Keeping only the running max.** A single negative wipes it out; without `cur_min` you miss `[-2,-3]` producing 6.
- **Computing `cur_max` then using the updated value for `cur_min`.** Snapshot `cur_max * x` and `cur_min * x` *before* reassigning either.
- **Initialising `best` to 0 or 1.** Use `nums[0]`; an all-negative single-element array like `[-3]` must return `-3`.
- **Zeros.** They force both running products to `x` (i.e. 0); the `max(x, ...)` form handles the reset without a special case.

## The keystone

When the accumulation is not monotonic — a product, or anything a single element can invert — carry the extremes in both directions. The max/min pair here is the same defensive move as tracking two heaps for a running median.
