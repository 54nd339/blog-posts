---
title: Binary Search - Halving the Search Space Correctly
description: Locating a value in a sorted array in logarithmic time — recursive and iterative forms, the lower- and upper-bound variants that generalise to "first index satisfying a predicate", and the library call.
date: 2021-07-19
draft: false
slug: /dsa/binary-search
tags:
  - Competitive Programming
  - Binary Search
  - Arrays & Hashing
---

The concept is "higher or lower?" Writing it without an off-by-one or an infinite loop is a skill worth drilling, because every later problem in this section is a variation on the boundary logic here.

## Description

Given a sorted array `nums` of distinct integers and an integer `target`, return the index of `target`, or `-1` if it is absent. Run in $O(\log n)$.

**Example**

```
Input:  nums = [-1, 0, 3, 5, 9, 12], target = 9
Output: 4

Input:  nums = [-1, 0, 3, 5, 9, 12], target = 2
Output: -1
```

**Constraints**

- $1 \le \text{nums.length} \le 10^4$
- `nums` is sorted ascending with distinct values.
- $-10^4 \le \text{nums}[i], \text{target} \le 10^4$

## Prerequisites

- A sorted array (or any search space monotone with respect to the predicate).
- The half-open vs closed interval distinction — it drives the loop condition.

## Approach 1: Recursive Binary Search

### Intuition

Look at the middle of the current range. Recurse into the half that could still contain `target`.

### Algorithm

1. `rec(l, r)`: if `l > r`, return `-1`.
2. `m = l + (r - l) // 2`.
3. `nums[m] == target` → return `m`; `nums[m] < target` → `rec(m + 1, r)`; else `rec(l, m - 1)`.
4. Call `rec(0, n - 1)`.

```python
def search(nums: list[int], target: int) -> int:
    def rec(l: int, r: int) -> int:
        if l > r:
            return -1
        m = l + (r - l) // 2
        if nums[m] == target:
            return m
        if nums[m] < target:
            return rec(m + 1, r)
        return rec(l, m - 1)
    return rec(0, len(nums) - 1)
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(\log n)$ recursion stack.

## Approach 2: Iterative Binary Search

### Intuition

The same halving without the call stack, over an inclusive range `[lo, hi]`.

### Algorithm

1. `lo = 0`, `hi = n - 1`.
2. While `lo <= hi`: `m = lo + (hi - lo) // 2`. Equal → return `m`. Less → `lo = m + 1`. Greater → `hi = m - 1`.
3. Return `-1`.

```python
def search(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        m = lo + (hi - lo) // 2
        if nums[m] == target:
            return m
        if nums[m] < target:
            lo = m + 1
        else:
            hi = m - 1
    return -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 3: Upper Bound

### Intuition

Find the first index whose value is *strictly greater* than `target`, using a half-open range `[lo, hi)`. The element just before it is `target` if present.

### Algorithm

1. `lo = 0`, `hi = n`.
2. While `lo < hi`: `m = lo + (hi - lo) // 2`. If `nums[m] > target`, `hi = m`; else `lo = m + 1`.
3. If `lo > 0` and `nums[lo - 1] == target`, return `lo - 1`; else `-1`.

```python
def search(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        m = lo + (hi - lo) // 2
        if nums[m] > target:
            hi = m
        else:
            lo = m + 1
    return lo - 1 if lo > 0 and nums[lo - 1] == target else -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 4: Lower Bound

### Intuition

Find the first index whose value is `>= target`. If that element equals `target`, it is the answer.

### Algorithm

1. `lo = 0`, `hi = n`.
2. While `lo < hi`: `m = lo + (hi - lo) // 2`. If `nums[m] >= target`, `hi = m`; else `lo = m + 1`.
3. If `lo < n` and `nums[lo] == target`, return `lo`; else `-1`.

```python
def search(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        m = lo + (hi - lo) // 2
        if nums[m] >= target:
            hi = m
        else:
            lo = m + 1
    return lo if lo < len(nums) and nums[lo] == target else -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 5: Built-In Function

### Intuition

`bisect_left` is a lower-bound search; check the returned index.

### Algorithm

1. `i = bisect_left(nums, target)`.
2. Return `i` if `i < n` and `nums[i] == target`, else `-1`.

```python
from bisect import bisect_left

def search(nums: list[int], target: int) -> int:
    i = bisect_left(nums, target)
    return i if i < len(nums) and nums[i] == target else -1
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Loop condition and range must match.** Inclusive `[lo, hi]` uses `while lo <= hi` with `hi = m - 1`; half-open `[lo, hi)` uses `while lo < hi` with `hi = m`. Mixing them either skips the last element or loops forever.
- **`lo = m` on the "less than" branch.** When `lo == hi - 1` and `nums[m] < target`, `lo = m` leaves the range unchanged — infinite loop. Use `m + 1`.
- **`m = (lo + hi) // 2` in fixed-width-integer languages** can overflow; `lo + (hi - lo) // 2` cannot. Harmless in Python, but keep the habit.

## The keystone

Binary search applies whenever the search space is ordered *with respect to the predicate you test* — if "is the answer at least `x`?" flips from false to true exactly once as `x` grows, bisect for that boundary. The lower/upper-bound templates are that boundary search, and they power [Koko Eating Bananas](/citadel/dsa/koko-eating-bananas) and [Search in Rotated Sorted Array](/citadel/dsa/search-in-rotated-sorted-array).
