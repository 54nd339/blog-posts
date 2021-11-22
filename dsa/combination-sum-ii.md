---
title: Combination Sum II - Each Number Once, No Duplicate Combinations
description: Combinations summing to a target where each array element is used at most once and the array has repeats — sort, advance the index every pick, and skip equal siblings at each level.
date: 2021-11-22
draft: false
slug: /dsa/combination-sum-ii
tags:
  - Competitive Programming
  - Backtracking
  - Recursion
---

Two changes from [Combination Sum](/citadel/dsa/combination-sum): each element may be used only once (so every pick advances the index), and the candidates contain duplicates (so skip equal siblings at each recursion level, the [Subsets II](/citadel/dsa/subsets-ii) trick).

## Description

Given a collection of candidate numbers `candidates` (with duplicates) and a target, return all unique combinations that sum to `target`. Each number may be used at most once per combination.

**Example**

```
Input:  candidates = [10,1,2,7,6,1,5], target = 8
Output: [[1,1,6],[1,2,5],[1,7],[2,6]]

Input:  candidates = [2,5,2,1,2], target = 5
Output: [[1,2,2],[5]]
```

**Constraints**

- $1 \le \text{candidates.length} \le 100$; $1 \le \text{candidates}[i] \le 50$
- $1 \le \text{target} \le 30$

## Prerequisites

- [Combination Sum](/citadel/dsa/combination-sum) — the target-sum recursion.
- [Subsets II](/citadel/dsa/subsets-ii) — sort + skip `candidates[j] == candidates[j-1]` when `j > start`.

## Approach 1: Brute Force (dedupe with a set)

### Intuition

Sort, generate all subsets via include/exclude, and add those summing to `target` (as tuples) to a set.

### Algorithm

1. Sort `candidates`.
2. Recurse include/exclude with a running total; prune when `total > target`.
3. On `total == target`, add `tuple(cur)` to a set.

```python
def combinationSum2(candidates: list[int], target: int) -> list[list[int]]:
    candidates.sort()
    seen = set()

    def dfs(i, cur, total):
        if total == target:
            seen.add(tuple(cur))
            return
        if i == len(candidates) or total > target:
            return
        dfs(i + 1, cur + [candidates[i]], total + candidates[i])
        dfs(i + 1, cur, total)

    dfs(0, [], 0)
    return [list(t) for t in seen]
```

### Complexity

- **Time:** $O(n \cdot 2^n)$.
- **Space:** $O(n \cdot 2^n)$.

## Approach 2: Backtracking (level-skip + pruning)

### Intuition

Sort. In a `for` loop over `j` from `start`, skip `j` when `j > start` and `candidates[j] == candidates[j - 1]`. Because the array is sorted, `break` as soon as `candidates[j] > remaining`. Recurse from `j + 1` (no reuse).

### Algorithm

1. Sort `candidates`.
2. `dfs(start, cur, remaining)`: if `remaining == 0`, record `cur[:]`; return.
3. For `j` from `start`: if `j > start` and `candidates[j] == candidates[j - 1]`, `continue`. If `candidates[j] > remaining`, `break`. Append, `dfs(j + 1, cur, remaining - candidates[j])`, pop.

```python
def combinationSum2(candidates: list[int], target: int) -> list[list[int]]:
    candidates.sort()
    res = []
    cur = []

    def dfs(start, remaining):
        if remaining == 0:
            res.append(cur[:])
            return
        for j in range(start, len(candidates)):
            if j > start and candidates[j] == candidates[j - 1]:
                continue
            if candidates[j] > remaining:
                break
            cur.append(candidates[j])
            dfs(j + 1, remaining - candidates[j])
            cur.pop()

    dfs(0, target)
    return res
```

### Complexity

- **Time:** $O(2^n)$ worst case, much less with the prune and skip.
- **Space:** $O(n)$ recursion.

## Approach 3: Backtracking (frequency map)

### Intuition

Collapse duplicates up front: work with `(value, count)` pairs. At each unique value, try taking it `0, 1, ..., count` times, then move to the next value.

### Algorithm

1. Build `counts` (a sorted list of `(value, count)`).
2. `dfs(i, cur, remaining)`: if `remaining == 0`, record; if `i == len(counts)` or `value > remaining`, return.
3. For `take` from `0` to `count`: if `take * value > remaining`, break; extend `cur` with `take` copies; `dfs(i + 1, cur, remaining - take * value)`; trim back.

```python
from collections import Counter

def combinationSum2(candidates: list[int], target: int) -> list[list[int]]:
    counts = sorted(Counter(candidates).items())
    res = []

    def dfs(i, cur, remaining):
        if remaining == 0:
            res.append(cur[:])
            return
        if i == len(counts):
            return
        value, count = counts[i]
        take = 0
        while take <= count and take * value <= remaining:
            dfs(i + 1, cur + [value] * take, remaining - take * value)
            take += 1

    dfs(0, [], target)
    return res
```

### Complexity

- **Time:** $O(2^n)$ worst case; the frequency collapse helps when duplicates are common.
- **Space:** $O(n)$ recursion.

## Common Pitfalls

- **Recursing from `j` instead of `j + 1`.** That re-enables reuse — this problem forbids it.
- **`j > start` vs `j > 0` in the skip.** Must be the level start, or you drop valid combinations that legitimately begin with a repeated value at a deeper level.
- **No `break` on overshoot.** With sorted candidates, once `candidates[j] > remaining`, all later ones do too.
- **Set-dedupe as the *only* mechanism.** Works, but you still explore the duplicate branches; the level-skip prunes them.

## The keystone

Combination Sum II sits at the intersection of two backtracking idioms: forward-only indexing without reuse (subset selection) and sorted duplicate-skipping. Recognising a problem as "that tree, plus these two constraints" is how the whole section collapses into one technique.
