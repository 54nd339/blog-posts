---
title: Subsets II - Skip Duplicate Siblings
description: The power set of a multiset without duplicate subsets — sort so equal values are adjacent, then at each recursion level use a repeated value only once.
date: 2021-11-22
draft: false
slug: /dsa/subsets-ii
tags:
  - Competitive Programming
  - Backtracking
  - Recursion
---

[Subsets](/citadel/dsa/subsets) plus repeated elements. `[1,2,2]` would otherwise yield `[1,2]` twice, from the two different `2`s. Sort first so equal values sit together, then within one recursion level, once you have skipped a value you must not start a branch with an equal value again.

## Description

Given an integer array `nums` that may contain duplicates, return all possible subsets. The solution must not contain duplicate subsets.

**Example**

```
Input:  nums = [1,2,2]
Output: [[],[1],[1,2],[1,2,2],[2],[2,2]]

Input:  nums = [0]
Output: [[],[0]]
```

**Constraints**

- $1 \le \text{nums.length} \le 10$
- $-10 \le \text{nums}[i] \le 10$

## Prerequisites

- [Subsets](/citadel/dsa/subsets).
- The "sort, then skip `nums[j] == nums[j-1]` when `j > start`" duplicate-avoidance idiom.

## Approach 1: Brute Force (dedupe with a set)

### Intuition

Generate all $2^n$ subsets as in Subsets, but sort each and store it as a tuple in a set to drop duplicates.

### Algorithm

1. Sort `nums`.
2. Recurse include/exclude; at the leaf, add `tuple(cur)` to a set.
3. Convert the set to a list of lists.

```python
def subsetsWithDup(nums: list[int]) -> list[list[int]]:
    nums.sort()
    seen = set()

    def dfs(i, cur):
        if i == len(nums):
            seen.add(tuple(cur))
            return
        dfs(i + 1, cur + [nums[i]])
        dfs(i + 1, cur)

    dfs(0, [])
    return [list(t) for t in seen]
```

### Complexity

- **Time:** $O(n \cdot 2^n)$.
- **Space:** $O(n \cdot 2^n)$.

## Approach 2: Backtracking (skip duplicates at each level)

### Intuition

Sort. In a `for` loop over `j` from `start`, add `nums[j]` and recurse from `j + 1`. Skip `j` when `j > start` and `nums[j] == nums[j - 1]` — that means an equal value already started a branch at this level.

### Algorithm

1. Sort `nums`.
2. `dfs(start, cur)`: append `cur[:]` to `res`.
3. For `j` from `start` to end: if `j > start` and `nums[j] == nums[j - 1]`, `continue`. Append `nums[j]`, `dfs(j + 1, cur)`, pop.

```python
def subsetsWithDup(nums: list[int]) -> list[list[int]]:
    nums.sort()
    res = []
    cur = []

    def dfs(start):
        res.append(cur[:])
        for j in range(start, len(nums)):
            if j > start and nums[j] == nums[j - 1]:
                continue
            cur.append(nums[j])
            dfs(j + 1)
            cur.pop()

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ worst case (all distinct); fewer when duplicates prune branches.
- **Space:** $O(n)$ recursion; $O(n \cdot 2^n)$ output.

## Approach 3: Backtracking (include/exclude, skip a run)

### Intuition

Binary version: at index `i`, include `nums[i]` and recurse to `i + 1`; then to *exclude* it, jump past every value equal to `nums[i]` before recursing, so the "exclude" branch never re-adds an equal value.

### Algorithm

1. Sort `nums`.
2. `dfs(i, cur)`: if `i == n`, record `cur[:]`.
3. Include: `dfs(i + 1, cur + [nums[i]])`.
4. Exclude: advance `k = i + 1` while `nums[k] == nums[i]`, then `dfs(k, cur)`.

```python
def subsetsWithDup(nums: list[int]) -> list[list[int]]:
    nums.sort()
    n = len(nums)
    res = []

    def dfs(i, cur):
        if i == n:
            res.append(cur[:])
            return
        cur.append(nums[i])
        dfs(i + 1, cur)
        cur.pop()
        k = i + 1
        while k < n and nums[k] == nums[i]:
            k += 1
        dfs(k, cur)

    dfs(0, [])
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ worst case.
- **Space:** $O(n)$ recursion.

## Common Pitfalls

- **`j > start` vs `j > 0`.** The skip must compare against the *level's* start index, not absolute `0` — using `j > 0` would wrongly skip a duplicate that is the legitimate first pick at a deeper level.
- **Not sorting.** The `nums[j] == nums[j-1]` test only works when equal values are adjacent.
- **Deduping only at the leaves.** That is Approach 1 and works, but wastes time building duplicate subsets you then discard; the level-skip prunes them before they form.

## The keystone

"Sort, then at each choice point use each distinct value once" is the universal fix for duplicate-generating backtracking. It reappears verbatim in [Combination Sum II](/citadel/dsa/combination-sum-ii) and the duplicate-tolerant version of [Permutations](/citadel/dsa/permutations).
