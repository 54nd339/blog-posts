---
title: House Robber II - A Circular Street, Solved Twice
description: Maximum non-adjacent sum when the first and last houses are neighbours — run the linear House Robber on the array without the first house and again without the last, take the better.
date: 2022-03-17
draft: false
slug: /dsa/house-robber-ii
tags:
  - Competitive Programming
  - Dynamic Programming
  - Arrays
---

The houses form a circle, so robbing house 0 forbids house `n - 1` and vice versa. That single coupling splits into two independent linear problems: one where house 0 is off the table, one where house `n - 1` is. Solve each with [House Robber](/citadel/dsa/house-robber) and take the max.

## Description

Same as House Robber, but the houses are arranged in a circle — `nums[0]` and `nums[n-1]` are adjacent. Return the maximum you can rob without taking two adjacent houses.

**Example**

```
Input:  nums = [2,3,2]
Output: 3   (can't take houses 0 and 2 together)

Input:  nums = [1,2,3,1]
Output: 4   (houses 0 and 2)
```

**Constraints**

- $1 \le \text{nums.length} \le 100$
- $0 \le \text{nums}[i] \le 1000$

## Prerequisites

- [House Robber](/citadel/dsa/house-robber) as a reusable subroutine.
- Case-splitting on the one constraint that breaks linearity.

## Approach 1: Two linear runs

### Intuition

Any valid selection either excludes house 0 or excludes house `n - 1` (it cannot include both). Run linear House Robber on `nums[1:]` and on `nums[:-1]`; the answer is the larger. Handle `n == 1` as its own case.

### Algorithm

1. If `n == 1`, return `nums[0]`.
2. `rob_linear(arr)` = the $O(1)$-space House Robber.
3. Return `max(rob_linear(nums[1:]), rob_linear(nums[:-1]))`.

```python
def rob(nums: list[int]) -> int:
    if len(nums) == 1:
        return nums[0]

    def rob_linear(arr):
        prev2 = prev1 = 0
        for x in arr:
            prev2, prev1 = prev1, max(prev1, prev2 + x)
        return prev1

    return max(rob_linear(nums[1:]), rob_linear(nums[:-1]))
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ if you index sub-ranges instead of slicing (slicing adds $O(n)$).

## Approach 2: Two runs with index bounds (no slicing)

### Intuition

Same idea, but pass `lo`/`hi` bounds to avoid allocating sub-arrays.

### Algorithm

1. `helper(lo, hi)`: run the rolling recurrence over `nums[lo:hi]`.
2. Return `max(helper(1, n), helper(0, n - 1))`, with the `n == 1` guard.

```python
def rob(nums: list[int]) -> int:
    n = len(nums)
    if n == 1:
        return nums[0]

    def helper(lo, hi):
        prev2 = prev1 = 0
        for i in range(lo, hi):
            prev2, prev1 = prev1, max(prev1, prev2 + nums[i])
        return prev1

    return max(helper(1, n), helper(0, n - 1))
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Running the circular case as one pass with a special rule.** It is far cleaner to reduce to two linear passes than to track "did I take house 0" through the whole scan.
- **Forgetting `n == 1`.** Both sub-ranges are empty, so the two-run version would return 0 instead of `nums[0]`.
- **Using `n == 2` as a special case unnecessarily.** `max(rob_linear([nums[1]]), rob_linear([nums[0]]))` already gives `max(nums[0], nums[1])`.
- **Slicing in a tight-memory setting.** `nums[1:]` copies; pass indices if $O(1)$ space matters.

## The keystone

A lone constraint that couples the two ends of an array often dissolves into "solve it without end A" and "solve it without end B". Recognising when a hard variant is two easy variants in a trenchcoat saves you from inventing new state.
