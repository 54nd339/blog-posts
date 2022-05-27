---
title: Jump Game II - Count the Frontier Extensions
description: The fewest jumps to reach the last index — a BFS-by-levels over the array where each level is the range reachable with one more jump, incrementing the count when you cross the current level's end.
date: 2022-05-27
draft: false
slug: /dsa/jump-game-ii
tags:
  - Competitive Programming
  - Greedy
  - Arrays
---

Think of it as BFS without a queue. Level `k` is the set of indices reachable in exactly `k` jumps, a contiguous range `[start, end]`. While scanning it, track the farthest you could land; when you reach `end`, that farthest becomes the next level's end and the jump count goes up.

## Description

Given an array `nums` where `nums[i]` is the maximum jump length from `i`, return the minimum number of jumps to reach the last index. It is guaranteed you can.

**Example**

```
Input:  nums = [2,3,1,1,4]
Output: 2   (index 0 → 1 → 4)

Input:  nums = [2,3,0,1,4]
Output: 2
```

**Constraints**

- $1 \le \text{nums.length} \le 10^4$
- $0 \le \text{nums}[i] \le 1000$; the last index is always reachable.

## Prerequisites

- The reachable frontier from [Jump Game](/citadel/dsa/jump-game).
- Level-by-level BFS expressed as two pointers over the array.

## Approach 1: Greedy level expansion (BFS by ranges)

### Intuition

`cur_end` marks the end of the current jump's reach; `farthest` is the best landing seen while scanning up to `cur_end`. Each time `i` hits `cur_end`, commit a jump: `jumps += 1`, `cur_end = farthest`.

### Algorithm

1. `jumps = 0`, `cur_end = 0`, `farthest = 0`.
2. For `i` from `0` to `n - 2`: `farthest = max(farthest, i + nums[i])`. If `i == cur_end`: `jumps += 1`; `cur_end = farthest`.
3. Return `jumps`.

```python
def jump(nums: list[int]) -> int:
    n = len(nums)
    jumps = cur_end = farthest = 0
    for i in range(n - 1):
        farthest = max(farthest, i + nums[i])
        if i == cur_end:
            jumps += 1
            cur_end = farthest
    return jumps
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Dynamic programming

### Intuition

`dp[i]` = min jumps to reach `i`. `dp[0] = 0`; for each `i`, relax `dp[j] = min(dp[j], dp[i] + 1)` for `j` in `i+1 .. i+nums[i]`.

### Algorithm

1. `dp = [inf] * n`, `dp[0] = 0`.
2. For `i` from `0` to `n - 1`: if `dp[i]` is finite, for `j` from `i + 1` to `min(n - 1, i + nums[i])`: `dp[j] = min(dp[j], dp[i] + 1)`.
3. Return `dp[n - 1]`.

```python
def jump(nums: list[int]) -> int:
    n = len(nums)
    dp = [float("inf")] * n
    dp[0] = 0
    for i in range(n):
        if dp[i] == float("inf"):
            continue
        for j in range(i + 1, min(n - 1, i + nums[i]) + 1):
            if dp[i] + 1 < dp[j]:
                dp[j] = dp[i] + 1
    return dp[n - 1]
```

### Complexity

- **Time:** $O(n^2)$ worst case.
- **Space:** $O(n)$.

## Common Pitfalls

- **Looping `i` to `n - 1` in the greedy version.** Stop at `n - 2`; standing on the last index must not trigger another jump.
- **Updating `cur_end` before scanning the whole level.** Commit the jump only when `i` actually reaches `cur_end`, using the `farthest` accumulated across the entire level.
- **Counting a jump per index.** You count once per *level*, not per step within a level.
- **Ignoring `nums[i] == 0`.** A zero contributes `i + 0` to `farthest`, i.e. nothing; the guarantee that the end is reachable means some earlier index jumps over it.

## The keystone

Minimum jumps is unweighted shortest path, and the greedy range-expansion *is* BFS with the queue replaced by two indices — $O(n)$ instead of $O(n^2)$. The "extend a frontier, count crossings" shape reappears any time levels are contiguous intervals.
