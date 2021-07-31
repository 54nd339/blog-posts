---
title: Search in Rotated Sorted Array - One Half Is Always Sorted
description: Finding a target in a rotated sorted array in logarithmic time — at every midpoint one side is a clean sorted run, so you can test whether the target lies in it and discard the other side.
date: 2021-07-31
draft: false
slug: /dsa/search-in-rotated-sorted-array
tags:
  - Competitive Programming
  - Binary Search
  - Arrays & Hashing
---

Split a rotated sorted array at any midpoint and at least one side is fully sorted. Check whether the target falls inside that sorted side; if it does, keep it, otherwise search the other side.

## Description

Given a rotated ascending array `nums` of distinct integers and a `target`, return its index or `-1`, in $O(\log n)$.

**Example**

```
Input:  nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input:  nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```

**Constraints**

- $1 \le \text{nums.length} \le 5000$
- $-10^4 \le \text{nums}[i], \text{target} \le 10^4$, all distinct.
- `nums` is an ascending array rotated by some amount.

## Prerequisites

- [Binary search](/citadel/dsa/binary-search).
- [Find Minimum in Rotated Sorted Array](/citadel/dsa/find-minimum-in-rotated-sorted-array) — locating the pivot.

## Approach 1: Brute Force

### Intuition

Linear scan.

### Algorithm

1. Return the first index where `nums[i] == target`, else `-1`.

```python
def search(nums: list[int], target: int) -> int:
    for i, x in enumerate(nums):
        if x == target:
            return i
    return -1
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Binary Search (find pivot, then search a segment)

### Intuition

Find the index of the minimum (the pivot) with one binary search. That splits the array into two sorted runs; binary-search whichever run could contain `target`.

### Algorithm

1. Binary-search for `pivot`, the index of the smallest element (compare `nums[mid]` with `nums[hi]`).
2. If `target` is between `nums[pivot]` and `nums[n - 1]` inclusive, binary-search `[pivot, n - 1]`; otherwise binary-search `[0, pivot - 1]`.

```python
def search(nums: list[int], target: int) -> int:
    n = len(nums)
    lo, hi = 0, n - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < nums[hi]:
            hi = mid
        else:
            lo = mid + 1
    pivot = lo

    def bsearch(l: int, r: int) -> int:
        while l <= r:
            m = (l + r) // 2
            if nums[m] == target:
                return m
            if nums[m] < target:
                l = m + 1
            else:
                r = m - 1
        return -1

    if pivot <= n - 1 and nums[pivot] <= target <= nums[n - 1]:
        return bsearch(pivot, n - 1)
    return bsearch(0, pivot - 1)
```

### Complexity

- **Time:** $O(\log n)$ — two logarithmic searches.
- **Space:** $O(1)$.

## Approach 3: Binary Search (Two Pass)

### Intuition

Same as Approach 2, expressed as: find the pivot, then decide which half to search by comparing `target` to `nums[-1]`, then run one plain binary search.

### Algorithm

1. Find `pivot` as above.
2. If `target > nums[-1]`, search `[0, pivot - 1]`; else search `[pivot, n - 1]`.
3. Standard binary search on that range.

```python
def search(nums: list[int], target: int) -> int:
    n = len(nums)
    lo, hi = 0, n - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < nums[hi]:
            hi = mid
        else:
            lo = mid + 1
    pivot = lo

    if target > nums[-1]:
        lo, hi = 0, pivot - 1
    else:
        lo, hi = pivot, n - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 4: Binary Search (One Pass)

### Intuition

No separate pivot hunt. At each `mid`, determine which side is sorted, then check whether `target` lies within that sorted side's value range; move accordingly.

### Algorithm

1. `lo = 0`, `hi = n - 1`.
2. While `lo <= hi`: `mid = (lo + hi) // 2`. If `nums[mid] == target`, return `mid`.
   - If `nums[lo] <= nums[mid]` (left half sorted): if `nums[lo] <= target < nums[mid]`, `hi = mid - 1`; else `lo = mid + 1`.
   - Else (right half sorted): if `nums[mid] < target <= nums[hi]`, `lo = mid + 1`; else `hi = mid - 1`.
3. Return `-1`.

```python
def search(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **The "sorted half" test needs `<=`, not `<`.** With a two-element window `nums[lo] <= nums[mid]` can be an equality (`lo == mid`); `<` misclassifies it.
- **Range-inclusive comparisons.** The left half owns `[nums[lo], nums[mid])`; the right half owns `(nums[mid], nums[hi]]`. Getting an endpoint's inclusivity wrong sends the search the wrong way at the boundary.
- **Assuming duplicates.** This problem guarantees distinct values; with duplicates, `nums[lo] == nums[mid] == nums[hi]` forces an $O(n)$ fallback.

## The keystone

A rotation preserves "one half is a clean sorted run". Identify that half, test membership against its known value range, discard the rest — logarithmic without ever materialising the pivot. The same "which sorted segment am I in?" question drives [Find Minimum in Rotated Sorted Array](/citadel/dsa/find-minimum-in-rotated-sorted-array).
