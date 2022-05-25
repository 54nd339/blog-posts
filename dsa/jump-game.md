---
title: Jump Game - Track the Farthest Reachable Index
description: Whether you can reach the last index when each value is a maximum jump length — sweep left to right keeping the farthest index reachable so far, and fail if you ever stand beyond it.
date: 2022-05-25
draft: false
slug: /dsa/jump-game
tags:
  - Competitive Programming
  - Greedy
  - Arrays
---

Keep one number: the farthest index reachable using jumps decided so far. At index `i`, if `i` is past that frontier you are stuck. Otherwise widen the frontier to `max(frontier, i + nums[i])`. Reachable iff the frontier ever covers the last index.

## Description

Given an array `nums` where `nums[i]` is the maximum forward jump length from index `i`, return `true` if you can reach the last index starting from index 0.

**Example**

```
Input:  nums = [2,3,1,1,4]
Output: true

Input:  nums = [3,2,1,0,4]
Output: false
```

**Constraints**

- $1 \le \text{nums.length} \le 10^4$
- $0 \le \text{nums}[i] \le 10^5$

## Prerequisites

- The greedy frontier: one running "farthest reachable" value.
- (Alternative) a backward reachability scan.

## Approach 1: Greedy forward frontier

### Intuition

`reach` = farthest index you can get to. Iterate `i`; if `i > reach`, return `False`. Else `reach = max(reach, i + nums[i])`. If `reach >= n - 1` at any point, return `True`.

### Algorithm

1. `reach = 0`.
2. For `i` from `0` to `n - 1`: if `i > reach`, return `False`. `reach = max(reach, i + nums[i])`. If `reach >= n - 1`, return `True`.
3. Return `True`.

```python
def canJump(nums: list[int]) -> bool:
    reach = 0
    n = len(nums)
    for i in range(n):
        if i > reach:
            return False
        reach = max(reach, i + nums[i])
        if reach >= n - 1:
            return True
    return True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Greedy backward (shrinking goal post)

### Intuition

Start with `goal = n - 1`. Scan from right to left; if index `i` can reach `goal` (`i + nums[i] >= goal`), move `goal` to `i`. Reachable iff `goal` ends at 0.

### Algorithm

1. `goal = n - 1`.
2. For `i` from `n - 2` down to `0`: if `i + nums[i] >= goal`, `goal = i`.
3. Return `goal == 0`.

```python
def canJump(nums: list[int]) -> bool:
    goal = len(nums) - 1
    for i in range(len(nums) - 2, -1, -1):
        if i + nums[i] >= goal:
            goal = i
    return goal == 0
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Dynamic programming

### Intuition

`dp[i]` = can index `i` reach the end. `dp[n-1] = True`; `dp[i]` is true if any `j` in `i+1 .. i+nums[i]` has `dp[j]` true.

### Algorithm

1. `dp = [False] * n`, `dp[n-1] = True`.
2. For `i` from `n - 2` down to `0`: scan `j` from `i + 1` to `min(n - 1, i + nums[i])`; if some `dp[j]`, set `dp[i] = True` and break.
3. Return `dp[0]`.

```python
def canJump(nums: list[int]) -> bool:
    n = len(nums)
    dp = [False] * n
    dp[n - 1] = True
    for i in range(n - 2, -1, -1):
        for j in range(i + 1, min(n - 1, i + nums[i]) + 1):
            if dp[j]:
                dp[i] = True
                break
    return dp[0]
```

### Complexity

- **Time:** $O(n^2)$ worst case.
- **Space:** $O(n)$.

## Common Pitfalls

- **Comparing `reach` to `n` instead of `n - 1`.** The target is the *last index*, `n - 1`.
- **Not checking `i > reach` before extending.** Once you fall behind the frontier you can never catch up; extend only from indices you can actually stand on.
- **Backward version with `>` instead of `>=`.** `i + nums[i] == goal` is a valid jump that lands exactly on the goal.
- **Assuming you must use the full jump.** `nums[i]` is a *maximum*; the greedy frontier already accounts for every shorter jump.

## The keystone

When the only thing that matters about the past is a single "best reachable" number, a greedy one-pass beats the DP. [Jump Game II](/citadel/dsa/jump-game-ii) keeps the frontier idea but counts how many times you must extend it.
