---
title: Longest Consecutive Sequence - Only Count From a Run's Start
description: The longest run of consecutive integers in an unsorted array in linear time — a hash set makes membership O(1), and starting a count only from numbers with no left neighbour keeps the total work linear.
date: 2021-05-07
draft: false
slug: /dsa/longest-consecutive-sequence
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Hash Table
---

Sorting gets you there in $O(n \log n)$ with almost no thought. The linear solution needs one idea: put everything in a hash set, then only extend a run from numbers that have no predecessor in the set.

## Description

Given an unsorted array `nums`, return the length of the longest run of consecutive integers it contains. The numbers need not be adjacent in the array. Run in $O(n)$ time.

**Example**

```
Input:  nums = [100, 4, 200, 1, 3, 2]
Output: 4          # the run 1, 2, 3, 4

Input:  nums = [0, 3, 7, 2, 5, 8, 4, 6, 0, 1]
Output: 9
```

**Constraints**

- $0 \le \text{nums.length} \le 10^5$
- $-10^9 \le \text{nums}[i] \le 10^9$

## Prerequisites

- Hash sets: $O(1)$ membership.
- The charging argument that bounds a nested loop's total iterations to $O(n)$.

## Approach 1: Brute Force

### Intuition

Load values into a set for $O(1)$ lookups, then from each number walk upward as long as the next consecutive value is present.

### Algorithm

1. Build `s = set(nums)`.
2. For each number `x` in `nums`, set `length = 0` and, while `x + length` is in `s`, increment `length`.
3. Track the maximum `length`.

```python
def longestConsecutive(nums: list[int]) -> int:
    s = set(nums)
    best = 0
    for x in nums:
        length = 0
        while x + length in s:
            length += 1
        best = max(best, length)
    return best
```

### Complexity

- **Time:** $O(n^2)$ — a run of length `L` is walked fully from each of its `L` members (input like `range(n)`).
- **Space:** $O(n)$ for the set.

## Approach 2: Sorting

### Intuition

After sorting, consecutive integers are adjacent. Sweep once, extending the current run on a `+1` step, skipping duplicates, and resetting otherwise.

### Algorithm

1. Return `0` if `nums` is empty.
2. Sort `nums`.
3. Track `streak` and `best`. For each element: if equal to the previous, skip; if one more than the previous, `streak += 1`; else `streak = 1`. Update `best`.

```python
def longestConsecutive(nums: list[int]) -> int:
    if not nums:
        return 0
    nums = sorted(set(nums))
    best = streak = 1
    for i in range(1, len(nums)):
        if nums[i] == nums[i - 1] + 1:
            streak += 1
            best = max(best, streak)
        else:
            streak = 1
    return best
```

### Complexity

- **Time:** $O(n \log n)$ — the sort dominates.
- **Space:** $O(n)$ for the deduplicated sorted list.

## Approach 3: Hash Set

### Intuition

A number `x` is the *start* of a run exactly when `x - 1` is absent. Only from those starts do you walk upward. Each number is the interior of exactly one run, so the total walking is $O(n)$.

### Algorithm

1. Build `s = set(nums)`.
2. For each `x` in `s`: if `x - 1` is in `s`, skip (`x` is not a run start).
3. Otherwise set `length = 1` and, while `x + length` is in `s`, increment `length`.
4. Track the maximum `length`.

```python
def longestConsecutive(nums: list[int]) -> int:
    s = set(nums)
    best = 0
    for x in s:
        if x - 1 in s:
            continue
        length = 1
        while x + length in s:
            length += 1
        best = max(best, length)
    return best
```

### Complexity

- **Time:** $O(n)$ — the inner `while` runs its full length only from a run's unique start, touching each element once overall.
- **Space:** $O(n)$ for the set.

## Approach 4: Hash Map

### Intuition

Track, at each run's two *boundary* values, the current length of that run. A new number merges the runs ending just below it and starting just above it.

### Algorithm

1. Keep a map `length_at` from a number to the length of the run it bounds.
2. For each number `x` not already in the map: let `left = length_at.get(x - 1, 0)`, `right = length_at.get(x + 1, 0)`, `total = left + right + 1`.
3. Set `length_at[x] = total`, and update the two outer boundary positions `x - left` and `x + right` to `total`.
4. Track the maximum `total`.

```python
def longestConsecutive(nums: list[int]) -> int:
    length_at = {}
    best = 0
    for x in nums:
        if x in length_at:
            continue
        left = length_at.get(x - 1, 0)
        right = length_at.get(x + 1, 0)
        total = left + right + 1
        length_at[x] = total
        length_at[x - left] = total
        length_at[x + right] = total
        best = max(best, total)
    return best
```

### Complexity

- **Time:** $O(n)$ — each number is processed once; only boundary entries are updated.
- **Space:** $O(n)$ for the map.

## Common Pitfalls

- **Dropping the `x - 1 in s` guard (Approach 3).** Without it, a run of length `L` is walked from all `L` members and the solution degrades to $O(n^2)$.
- **Iterating `nums` instead of the set (Approach 3).** Iterating the set skips duplicate starts for free.
- **Not deduplicating before Approach 2.** A repeat makes `nums[i] == nums[i-1]`, which is neither `+1` nor a reset; handle it explicitly or `set()` first.
- **Updating only the inner boundary in Approach 4.** You must write `total` to both *outer* ends `x - left` and `x + right`, since a future number will read those.

## The keystone

The linear time is a charging argument: each element is the interior of exactly one run, and you only pay to walk a run from its unique start. When a nested loop looks quadratic, ask whether $O(1)$ membership plus a "start work only at a canonical point" rule bounds the total iterations — the same reasoning behind [union-find](/citadel/dsa/number-of-connected-components-in-an-undirected-graph) and monotonic-stack solutions.
