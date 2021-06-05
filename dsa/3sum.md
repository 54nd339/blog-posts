---
title: 3Sum - Fix One, Two-Pointer the Rest
description: Every distinct triplet that sums to zero — the cubic brute force drops to quadratic once you sort, fix the first element, and sweep the other two with converging pointers while skipping duplicates.
date: 2021-06-05
draft: false
slug: /dsa/3sum
tags:
  - Competitive Programming
  - Two Pointers
  - Sorting
---

Sort the array, then for each index treat `nums[i]` as fixed and find all pairs in the rest that sum to `-nums[i]` — which is [Two Sum II](/citadel/dsa/two-sum-ii) on a sorted array. The only work beyond that is not emitting the same triplet twice.

## Description

Given an integer array `nums`, return all distinct triplets `[a, b, c]` with `a + b + c == 0`. The solution set must not contain duplicate triplets.

**Example**

```
Input:  nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]

Input:  nums = [0, 1, 1]
Output: []

Input:  nums = [0, 0, 0]
Output: [[0, 0, 0]]
```

**Constraints**

- $3 \le \text{nums.length} \le 3000$
- $-10^5 \le \text{nums}[i] \le 10^5$

## Prerequisites

- [Two Sum II](/citadel/dsa/two-sum-ii) — the sorted two-pointer inner sweep.
- Sorting, and using adjacent equality to skip duplicates.

## Approach 1: Brute Force

### Intuition

Try every unordered triple of indices; dedupe by the sorted value tuple.

### Algorithm

1. Sort `nums`.
2. Create an empty set for triplets.
3. Three nested loops over `i < j < k`; if the three values sum to zero, add the sorted `(nums[i], nums[j], nums[k])` tuple to the set.
4. Return the set as a list.

```python
def threeSum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    n = len(nums)
    found = set()
    for i in range(n):
        for j in range(i + 1, n):
            for k in range(j + 1, n):
                if nums[i] + nums[j] + nums[k] == 0:
                    found.add((nums[i], nums[j], nums[k]))
    return [list(t) for t in found]
```

### Complexity

- **Time:** $O(n^3)$ — about $n^3/6$ triples.
- **Space:** $O(m)$ for the `m` unique triplets (up to $O(n^2)$).

## Approach 2: Hash Map

### Intuition

Fix the first two elements; the third is forced to `-(nums[i] + nums[j])`. A frequency map tells you whether that value is still available.

### Algorithm

1. Sort `nums` and build a count map.
2. For each first index `i` (skip duplicates of `nums[i]`), decrement `count[nums[i]]`.
3. For each second index `j > i` (skip duplicates of `nums[j]`), decrement `count[nums[j]]`.
4. Let `target = -(nums[i] + nums[j])`. If `target >= nums[j]` and `count[target] > 0`, record `[nums[i], nums[j], target]`.
5. Restore the counts decremented for the inner loop before advancing `i`.

```python
from collections import Counter

def threeSum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    count = Counter(nums)
    res = []
    n = len(nums)
    for i in range(n):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        count[nums[i]] -= 1
        for j in range(i + 1, n):
            if j > i + 1 and nums[j] == nums[j - 1]:
                continue
            count[nums[j]] -= 1
            target = -(nums[i] + nums[j])
            if target >= nums[j] and count[target] > 0:
                res.append([nums[i], nums[j], target])
        for j in range(i + 1, n):
            count[nums[j]] += 1
        count[nums[i]] += 1
    return res
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n)$ for the frequency map.

## Approach 3: Two Pointers

### Intuition

After sorting, for each `i` (the smallest of the triplet) put `lo = i + 1`, `hi = n - 1`. Too small: `lo += 1`. Too large: `hi -= 1`. Zero: record it and move *both* inward, skipping equal values so each distinct triplet is emitted once.

### Algorithm

1. Sort `nums`.
2. For each `i`: if `nums[i] > 0`, stop (no way to reach zero). If `i > 0` and `nums[i] == nums[i - 1]`, skip.
3. `lo = i + 1`, `hi = n - 1`. While `lo < hi`:
   - `total = nums[i] + nums[lo] + nums[hi]`.
   - `total < 0` → `lo += 1`; `total > 0` → `hi -= 1`.
   - `total == 0` → append the triplet, then `lo += 1`, `hi -= 1`, and skip duplicates: while `lo < hi and nums[lo] == nums[lo - 1]`, `lo += 1`; while `lo < hi and nums[hi] == nums[hi + 1]`, `hi -= 1`.

```python
def threeSum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    res = []
    n = len(nums)
    for i in range(n - 2):
        if nums[i] > 0:
            break
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        lo, hi = i + 1, n - 1
        while lo < hi:
            total = nums[i] + nums[lo] + nums[hi]
            if total < 0:
                lo += 1
            elif total > 0:
                hi -= 1
            else:
                res.append([nums[i], nums[lo], nums[hi]])
                lo += 1
                hi -= 1
                while lo < hi and nums[lo] == nums[lo - 1]:
                    lo += 1
                while lo < hi and nums[hi] == nums[hi + 1]:
                    hi -= 1
    return res
```

### Complexity

- **Time:** $O(n^2)$ — an $O(n)$ sweep for each of `n` values, plus $O(n \log n)$ to sort.
- **Space:** $O(1)$ beyond the output (or $O(\log n)$ for the sort's stack).

## Common Pitfalls

- **The first-element dedupe compares backward.** `if i > 0 and nums[i] == nums[i - 1]` skips *repeat* occurrences, keeping the first. Comparing with `nums[i + 1]` instead skips the first occurrence — the one you want to process.
- **After a zero hit, advance both pointers.** Moving only one and leaving the other guarantees a repeated triplet on the next iteration.
- **Deduping by index tuple instead of value tuple (Approach 1).** With repeats, one triplet of values arises from several index combinations.

## The keystone

"Find `k` numbers summing to a target" reduces by one dimension each time you fix an element: 3Sum is a loop around 2Sum, 4Sum a loop around 3Sum. Sorting first makes the innermost level an $O(n)$ two-pointer sweep and lets you dedupe by skipping equal neighbours.
