---
title: Subsets - Include It or Don't
description: Generating the power set of distinct integers — a binary include/exclude recursion, an iterative doubling, or one bitmask per subset all produce the same 2 to the n results.
date: 2021-11-17
draft: false
slug: /dsa/subsets
tags:
  - Competitive Programming
  - Backtracking
  - Bit Manipulation
---

Every subset is a yes/no decision per element. That gives $2^n$ subsets and three natural ways to enumerate them: recurse on the decision, double an accumulating list, or let the bits of an integer be the decisions.

## Description

Given an array `nums` of unique integers, return all possible subsets (the power set). The solution must not contain duplicate subsets; order does not matter.

**Example**

```
Input:  nums = [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

Input:  nums = [0]
Output: [[],[0]]
```

**Constraints**

- $1 \le \text{nums.length} \le 10$
- All elements distinct.

## Prerequisites

- The include/exclude recursion tree.
- Bitmask enumeration `0 .. 2^n - 1`.

## Approach 1: Backtracking

### Intuition

At index `i`, branch: put `nums[i]` in the current subset and recurse, or leave it out and recurse. At the end of the array, record a copy.

### Algorithm

1. `res = []`, `cur = []`.
2. `dfs(i)`: if `i == len(nums)`, append `cur[:]` to `res`; return.
3. Include: `cur.append(nums[i])`, `dfs(i + 1)`, `cur.pop()`.
4. Exclude: `dfs(i + 1)`.

```python
def subsets(nums: list[int]) -> list[list[int]]:
    res = []
    cur = []

    def dfs(i: int) -> None:
        if i == len(nums):
            res.append(cur[:])
            return
        cur.append(nums[i])
        dfs(i + 1)
        cur.pop()
        dfs(i + 1)

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$ — $2^n$ subsets, each copied in $O(n)$.
- **Space:** $O(n)$ recursion; $O(n \cdot 2^n)$ output.

## Approach 2: Iteration

### Intuition

Start with `[[]]`. For each new number, every existing subset spawns a copy with that number appended.

### Algorithm

1. `res = [[]]`.
2. For each `x` in `nums`: `res += [subset + [x] for subset in res]`.

```python
def subsets(nums: list[int]) -> list[list[int]]:
    res = [[]]
    for x in nums:
        res += [subset + [x] for subset in res]
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$.
- **Space:** $O(n \cdot 2^n)$.

## Approach 3: Bit Manipulation

### Intuition

There are $2^n$ subsets, one per integer `mask` in `[0, 2^n)`. Bit `j` of `mask` decides whether `nums[j]` is in that subset.

### Algorithm

1. For `mask` from `0` to `2^n - 1`:
   - Build the subset from every `j` where `mask & (1 << j)` is set.

```python
def subsets(nums: list[int]) -> list[list[int]]:
    n = len(nums)
    res = []
    for mask in range(1 << n):
        res.append([nums[j] for j in range(n) if mask & (1 << j)])
    return res
```

### Complexity

- **Time:** $O(n \cdot 2^n)$.
- **Space:** $O(n \cdot 2^n)$.

## Common Pitfalls

- **Appending `cur` instead of `cur[:]`.** `cur` is mutated throughout; you must store a snapshot copy.
- **Iterating `res` while extending it.** In Approach 2, build the new subsets from a *snapshot* (`for subset in res` evaluated before `+=`, or slice `res[:]`) so you do not append to a list you are looping over.
- **Bitmask shift width.** `1 << n` is the count of subsets; `1 << j` for `j` in `range(n)` are the element bits.

## The keystone

The power set is the simplest backtracking tree: a binary choice at each of `n` levels. Everything else in this section adds constraints (a target sum, no duplicates, a grid) that prune branches of this same tree.
