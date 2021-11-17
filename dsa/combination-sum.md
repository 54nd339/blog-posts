---
title: Combination Sum - Reuse Allowed, Order Doesn't Matter
description: All multisets of candidates that sum to a target, with unlimited reuse — a recursion that either takes the current candidate again or moves past it, with sorting for early pruning.
date: 2021-11-17
draft: false
slug: /dsa/combination-sum
tags:
  - Competitive Programming
  - Backtracking
  - Recursion
---

Each candidate can be used any number of times, and `[2,2,3]` is the same combination as `[3,2,2]`. To avoid generating both, only ever move *forward* through the candidate list — but allow staying on the current index to reuse it.

## Description

Given an array of distinct integers `candidates` and a target, return all unique combinations where the chosen numbers sum to `target`. A number may be used unlimited times; two combinations are the same if they use the same multiset of numbers.

**Example**

```
Input:  candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]

Input:  candidates = [2,3,5], target = 8
Output: [[2,2,2,2],[2,3,3],[3,5]]
```

**Constraints**

- $1 \le \text{candidates.length} \le 30$; $2 \le \text{candidates}[i] \le 40$; distinct.
- $1 \le \text{target} \le 40$

## Prerequisites

- [Subsets](/citadel/dsa/subsets) — the include/exclude tree.
- Forward-only indexing to dedupe unordered combinations.

## Approach 1: Backtracking

### Intuition

At index `i`, either take `candidates[i]` (and stay at `i`, since reuse is allowed) or skip to `i + 1`. Stop when the running total hits the target (record) or exceeds it or runs out of candidates.

### Algorithm

1. `dfs(i, cur, total)`.
2. If `total == target`, append `cur[:]`; return.
3. If `i >= len(candidates)` or `total > target`, return.
4. Take: `cur.append(candidates[i])`, `dfs(i, cur, total + candidates[i])`, `cur.pop()`.
5. Skip: `dfs(i + 1, cur, total)`.

```python
def combinationSum(candidates: list[int], target: int) -> list[list[int]]:
    res = []

    def dfs(i, cur, total):
        if total == target:
            res.append(cur[:])
            return
        if i >= len(candidates) or total > target:
            return
        cur.append(candidates[i])
        dfs(i, cur, total + candidates[i])
        cur.pop()
        dfs(i + 1, cur, total)

    dfs(0, [], 0)
    return res
```

### Complexity

- **Time:** $O(2^{t/m})$ where `t` is the target and `m` the smallest candidate.
- **Space:** $O(t/m)$ recursion depth.

## Approach 2: Backtracking (sorted, with pruning)

### Intuition

Sort the candidates. Then in a `for` loop over `j >= i`, the moment `total + candidates[j] > target` you can `break` — every later candidate is at least as large and will also overshoot.

### Algorithm

1. Sort `candidates`.
2. `dfs(start, cur, total)`: if `total == target`, record and return.
3. For `j` from `start` to end: if `total + candidates[j] > target`, `break`. Append, recurse `dfs(j, ...)` (reuse allowed), pop.

```python
def combinationSum(candidates: list[int], target: int) -> list[list[int]]:
    candidates.sort()
    res = []

    def dfs(start, cur, total):
        if total == target:
            res.append(cur[:])
            return
        for j in range(start, len(candidates)):
            if total + candidates[j] > target:
                break
            cur.append(candidates[j])
            dfs(j, cur, total + candidates[j])
            cur.pop()

    dfs(0, [], 0)
    return res
```

### Complexity

- **Time:** $O(2^{t/m})$ worst case, far less in practice thanks to the `break`.
- **Space:** $O(t/m)$.

## Common Pitfalls

- **Recursing to `i + 1` on the "take" branch.** That forbids reuse; you must pass the *same* index (`i` or `j`) to allow taking a candidate again.
- **Not restricting the loop to `j >= start`.** Without it you generate `[2,3]` and `[3,2]` as distinct combinations.
- **Pruning with `continue` instead of `break`.** On a sorted array, once one candidate overshoots, all later ones do — `break` is correct and faster.
- **Overshoot check missing.** Without `total > target` (or the `break`), the recursion explores forever on large targets.

## The keystone

Unordered combinations with reuse = a forward-only recursion that may stay put. Sorting turns "this branch is hopeless" into a cheap `break`. [Combination Sum II](/citadel/dsa/combination-sum-ii) is the same tree with reuse disabled and a duplicate-skip added.
