---
title: Permutations - Every Order of Every Element
description: All orderings of a distinct-element array — insert each element into every gap of smaller permutations, or backtrack with a used marker, or swap elements into place in situ.
date: 2021-11-18
draft: false
slug: /dsa/permutations
tags:
  - Competitive Programming
  - Backtracking
  - Recursion
---

A permutation uses every element exactly once, in some order. There are `n!` of them. The backtracking version picks an unused element for each position; the swap version rearranges the array in place.

## Description

Given an array `nums` of distinct integers, return all possible permutations, in any order.

**Example**

```
Input:  nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

Input:  nums = [1]
Output: [[1]]
```

**Constraints**

- $1 \le \text{nums.length} \le 6$; all distinct.

## Prerequisites

- The `n!` recursion tree.
- A `used` marker (boolean array or bitmask) or in-place swapping.

## Approach 1: Recursion (insert everywhere)

### Intuition

Permutations of `nums` = permutations of `nums[1:]` with `nums[0]` inserted into every position.

### Algorithm

1. If `nums` is empty, return `[[]]`.
2. Recursively permute `nums[1:]`.
3. For each smaller permutation, produce `n` new ones by inserting `nums[0]` at each index `0 .. len`.

```python
def permute(nums: list[int]) -> list[list[int]]:
    if not nums:
        return [[]]
    first = nums[0]
    rest_perms = permute(nums[1:])
    res = []
    for p in rest_perms:
        for i in range(len(p) + 1):
            res.append(p[:i] + [first] + p[i:])
    return res
```

### Complexity

- **Time:** $O(n! \cdot n^2)$ — `n!` results, each built with $O(n)$ slicing at $O(n)$ positions.
- **Space:** $O(n! \cdot n)$.

## Approach 2: Backtracking (used array)

### Intuition

Fill positions left to right. For each position, try every element not yet used.

### Algorithm

1. `used = [False] * n`, `cur = []`, `res = []`.
2. `dfs()`: if `len(cur) == n`, append `cur[:]`; return.
3. For each `i`: if not `used[i]`, mark it, `cur.append(nums[i])`, recurse, `cur.pop()`, unmark.

```python
def permute(nums: list[int]) -> list[list[int]]:
    res = []
    cur = []
    used = [False] * len(nums)

    def dfs():
        if len(cur) == len(nums):
            res.append(cur[:])
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            used[i] = True
            cur.append(nums[i])
            dfs()
            cur.pop()
            used[i] = False

    dfs()
    return res
```

### Complexity

- **Time:** $O(n! \cdot n)$.
- **Space:** $O(n)$ recursion (an integer bitmask can replace `used` with the same complexity).

## Approach 3: Backtracking (in-place swap)

### Intuition

Treat `nums[:k]` as the fixed prefix. For position `k`, swap each of `nums[k:]` into slot `k`, recurse on `k + 1`, then swap back.

### Algorithm

1. `dfs(k)`: if `k == n`, append `nums[:]`; return.
2. For `i` from `k` to `n - 1`: swap `nums[k]` and `nums[i]`; `dfs(k + 1)`; swap back.

```python
def permute(nums: list[int]) -> list[list[int]]:
    res = []
    n = len(nums)

    def dfs(k):
        if k == n:
            res.append(nums[:])
            return
        for i in range(k, n):
            nums[k], nums[i] = nums[i], nums[k]
            dfs(k + 1)
            nums[k], nums[i] = nums[i], nums[k]

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n! \cdot n)$ — `n!` leaves, $O(n)$ to copy each.
- **Space:** $O(n)$ recursion; no `used` array.

## Common Pitfalls

- **Forgetting to unmark / swap back.** Every state change before recursing must be undone after, or sibling branches see corrupted state.
- **Appending `nums` (Approach 3) instead of `nums[:]`.** The array keeps mutating; store a copy.
- **Approach 3 with duplicate elements.** The plain swap version emits duplicate permutations if `nums` has repeats; for that case, use the `used` array plus a "skip equal, already-used sibling" check (as in [Subsets II](/citadel/dsa/subsets-ii)).

## The keystone

Permutations is the `n!` sibling of the `2^n` subsets tree: at each position you choose one of the *remaining* elements. Track "remaining" with a boolean array, a bitmask, or by physically partitioning the array with swaps.
