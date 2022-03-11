---
title: House Robber - Take This One and Skip a House, or Skip This One
description: The maximum sum of non-adjacent array elements — at each house choose the better of robbing it plus the best up to two houses back, or skipping it and keeping the best so far.
date: 2022-03-11
draft: false
slug: /dsa/house-robber
tags:
  - Competitive Programming
  - Dynamic Programming
  - Arrays
---

At house `i` you either rob it — adding `nums[i]` to the best total that ends at or before `i - 2` — or you skip it, keeping the best total through `i - 1`. Two rolling variables hold exactly those two quantities.

## Description

Given an integer array `nums` of house values, return the maximum amount you can rob without robbing two adjacent houses.

**Example**

```
Input:  nums = [1,2,3,1]
Output: 4   (rob houses 0 and 2)

Input:  nums = [2,7,9,3,1]
Output: 12  (rob houses 0, 2, 4)
```

**Constraints**

- $1 \le \text{nums.length} \le 100$
- $0 \le \text{nums}[i] \le 400$

## Prerequisites

- The "include vs exclude" recurrence with a one-house gap.
- Reducing a DP array to two scalars.

## Approach 1: Bottom-up array

### Intuition

`dp[i]` = max loot considering houses `0..i`. `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — skip house `i`, or rob it and add the best from two back.

### Algorithm

1. Handle `n == 1` directly.
2. `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.
3. For `i` from `2`: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.
4. Return `dp[n-1]`.

```python
def rob(nums: list[int]) -> int:
    n = len(nums)
    if n == 1:
        return nums[0]
    dp = [0] * n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i in range(2, n):
        dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
    return dp[n - 1]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 2: Two rolling variables

### Intuition

`prev2` = best through `i - 2`, `prev1` = best through `i - 1`. For each house, `cur = max(prev1, prev2 + nums[i])`, then shift.

### Algorithm

1. `prev2 = prev1 = 0`.
2. For each `x` in `nums`: `cur = max(prev1, prev2 + x)`; `prev2, prev1 = prev1, cur`.
3. Return `prev1`.

```python
def rob(nums: list[int]) -> int:
    prev2 = prev1 = 0
    for x in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + x)
    return prev1
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Recursion with memoization

### Intuition

`f(i)` = max loot from `nums[i:]`. `f(i) = max(f(i + 1), nums[i] + f(i + 2))`. Cache on `i`.

### Algorithm

Top-down mirror of Approach 1, indexing forward.

```python
from functools import lru_cache

def rob(nums: list[int]) -> int:
    n = len(nums)

    @lru_cache(None)
    def f(i):
        if i >= n:
            return 0
        return max(f(i + 1), nums[i] + f(i + 2))

    return f(0)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`dp[1]` base case.** It is `max(nums[0], nums[1])`, not `nums[1]` — you would never take the smaller of the first two alone.
- **Starting `prev1`/`prev2` at anything but 0.** The rolling form handles `n == 1` for free only if both start at 0.
- **Assuming the answer alternates houses.** `[2,1,1,2]` robs houses 0 and 3, which are two apart *and* three apart; the recurrence, not a fixed pattern, decides.
- **Negative values.** Constraints keep `nums[i] >= 0`; with negatives you would also compare against robbing nothing.

## The keystone

"Best subset with a spacing constraint" collapses to a two-term recurrence and $O(1)$ space. [House Robber II](/citadel/dsa/house-robber-ii) reuses this function twice to handle a circular street.
