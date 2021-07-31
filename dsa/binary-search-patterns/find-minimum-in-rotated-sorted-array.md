---
title: Find Minimum in Rotated Sorted Array - Compare Against the Right End
description: The pivot of a rotated sorted array in logarithmic time — comparing the midpoint to the right endpoint tells you which half holds the wrap-around, and the minimum sits at that wrap.
date: 2021-07-31
draft: false
slug: /dsa/find-minimum-in-rotated-sorted-array
tags:
  - Competitive Programming
  - Binary Search
  - Arrays & Hashing
---

The minimum is the single point where the array "drops". Binary search finds it by asking, at each midpoint, "is the drop to my left or my right?" — and the reliable reference to compare against is the right endpoint.

## Description

A sorted array of distinct numbers has been rotated by an unknown amount. Return its minimum, in $O(\log n)$.

**Example**

```
Input:  nums = [3,4,5,1,2]
Output: 1

Input:  nums = [4,5,6,7,0,1,2]
Output: 0

Input:  nums = [11,13,15,17]
Output: 11
```

**Constraints**

- $1 \le \text{nums.length} \le 5000$
- $-5000 \le \text{nums}[i] \le 5000$, all distinct.
- The array is an ascending sort rotated between `1` and `n` times (a rotation by `n` leaves it sorted).

## Prerequisites

- [Binary search](/citadel/dsa/binary-search).
- The idea that a rotated sorted array is two sorted runs, and one comparison identifies which run a midpoint is in.

## Approach 1: Brute Force

### Intuition

Scan for the smallest element.

### Algorithm

1. Return `min(nums)`.

```python
def findMin(nums: list[int]) -> int:
    return min(nums)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Binary Search (sorted-half check)

### Intuition

Track a running answer. At each step, if the current window `[lo, hi]` is already sorted (`nums[lo] <= nums[hi]`), its first element is the smallest in it — fold that in and stop. Otherwise take `nums[mid]` as a candidate and move toward the unsorted half, where the true minimum lives.

### Algorithm

1. `lo = 0`, `hi = n - 1`, `ans = nums[0]`.
2. While `lo <= hi`:
   - If `nums[lo] <= nums[hi]`: `ans = min(ans, nums[lo])`; break.
   - `mid = (lo + hi) // 2`; `ans = min(ans, nums[mid])`.
   - If `nums[lo] <= nums[mid]` (left half sorted), `lo = mid + 1`; else `hi = mid - 1`.
3. Return `ans`.

```python
def findMin(nums: list[int]) -> int:
    lo, hi = 0, len(nums) - 1
    ans = nums[0]
    while lo <= hi:
        if nums[lo] <= nums[hi]:
            ans = min(ans, nums[lo])
            break
        mid = (lo + hi) // 2
        ans = min(ans, nums[mid])
        if nums[lo] <= nums[mid]:
            lo = mid + 1
        else:
            hi = mid - 1
    return ans
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 3: Binary Search (Lower Bound)

### Intuition

Compare `nums[mid]` with `nums[hi]`. If `nums[mid] < nums[hi]`, the segment `[mid, hi]` is non-decreasing, so the minimum is at `mid` or to its left — set `hi = mid`. Otherwise the wrap is in `(mid, hi]` — set `lo = mid + 1`. Converge to a single index.

### Algorithm

1. `lo = 0`, `hi = n - 1`.
2. While `lo < hi`: `mid = (lo + hi) // 2`. If `nums[mid] < nums[hi]`, `hi = mid`; else `lo = mid + 1`.
3. Return `nums[lo]`.

```python
def findMin(nums: list[int]) -> int:
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < nums[hi]:
            hi = mid
        else:
            lo = mid + 1
    return nums[lo]
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Comparing `nums[mid]` with `nums[lo]` instead of `nums[hi]` (Approach 3).** The left-endpoint comparison is ambiguous — in a non-rotated array `nums[mid] >= nums[lo]` too — so you cannot distinguish "sorted" from "pivot to the right" without an extra case.
- **`hi = mid - 1` when `nums[mid] < nums[hi]`.** `mid` itself may be the minimum; use `hi = mid`.
- **Loop condition.** With `hi = mid`, `while lo <= hi` never terminates; use `while lo < hi`.

## The keystone

In a rotated sorted array, every midpoint sits in one of two sorted runs, and one comparison with a fixed reference point (here `nums[hi]`) identifies which run and which half to keep. Choosing the *unambiguous* reference is the craft — the same reasoning drives [Search in Rotated Sorted Array](/citadel/dsa/search-in-rotated-sorted-array).
