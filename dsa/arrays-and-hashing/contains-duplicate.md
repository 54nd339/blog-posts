---
title: Contains Duplicate - Trading Space for a Single Pass
description: Deciding whether an array has any repeated value, four ways — all-pairs comparison, sort-then-scan, a one-pass hash set, and the one-liner set-length check.
date: 2021-04-13
draft: false
slug: /dsa/contains-duplicate
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Hash Table
---

The first rung of the NeetCode 150 ladder, and a small decision that echoes through the rest of the section: when a brute-force scan keeps re-checking "have I seen this?", replace it with an $O(1)$ hash lookup.

## Description

Given an integer array `nums`, return `true` if any value appears at least twice, and `false` if every element is distinct.

**Example**

```
Input:  nums = [1, 2, 3, 1]
Output: true

Input:  nums = [1, 2, 3, 4]
Output: false
```

**Constraints**

- $1 \le \text{nums.length} \le 10^5$
- $-10^9 \le \text{nums}[i] \le 10^9$

## Prerequisites

- [Arrays and hashing](/citadel/dsa/arrays-and-hashing) — the pattern this problem introduces.
- Hash sets: $O(1)$ average membership test and insertion.
- Sorting: $O(n \log n)$, and the fact that it makes equal values adjacent.

## Approach 1: Brute Force

### Intuition

If two equal elements exist, they sit at two different indices. With no other idea yet, check every unordered pair.

### Algorithm

1. Loop `i` over every index.
2. For each `i`, loop `j` over every later index `i + 1 .. n - 1`.
3. If `nums[i] == nums[j]`, return `true`.
4. If all pairs are exhausted, return `false`.

```python
def containsDuplicate(nums: list[int]) -> bool:
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] == nums[j]:
                return True
    return False
```

### Complexity

- **Time:** $O(n^2)$ — the inner loop runs $n-1, n-2, \dots, 1$ times, summing to $n(n-1)/2$.
- **Space:** $O(1)$ — no auxiliary structure.

## Approach 2: Sorting

### Intuition

Sorting drags equal values next to each other. A duplicate then exists exactly when some element equals its immediate neighbour, which one linear scan can check.

### Algorithm

1. Sort `nums` in non-decreasing order.
2. Walk from index `1` to the end.
3. If `nums[i] == nums[i - 1]`, return `true`.
4. If the scan completes, return `false`.

```python
def containsDuplicate(nums: list[int]) -> bool:
    nums.sort()
    for i in range(1, len(nums)):
        if nums[i] == nums[i - 1]:
            return True
    return False
```

### Complexity

- **Time:** $O(n \log n)$ — the sort dominates; the scan is $O(n)$.
- **Space:** $O(1)$ if the sort is in place and mutating the input is allowed; $O(n)$ if you copy first with `sorted(nums)`.

## Approach 3: Hash Set

### Intuition

You do not need the duplicate pair, only the fact that one exists. Keep a set of everything seen so far; the first repeat ends the search, often well before the end of the array.

### Algorithm

1. Create an empty set `seen`.
2. For each `x` in `nums`: if `x` is in `seen`, return `true`; otherwise add `x` to `seen`.
3. If the loop finishes, return `false`.

```python
def containsDuplicate(nums: list[int]) -> bool:
    seen = set()
    for x in nums:
        if x in seen:
            return True
        seen.add(x)
    return False
```

### Complexity

- **Time:** $O(n)$ — one pass; membership and insertion are $O(1)$ on average.
- **Space:** $O(n)$ — in the worst case (all distinct) the set holds every element.

## Approach 4: Hash Set Length

### Intuition

A set discards duplicates as it is built. If `set(nums)` ends up smaller than `nums`, something was dropped.

### Algorithm

1. Build `set(nums)`.
2. Return whether its length is less than `len(nums)`.

```python
def containsDuplicate(nums: list[int]) -> bool:
    return len(set(nums)) != len(nums)
```

### Complexity

- **Time:** $O(n)$ — one pass to build the set.
- **Space:** $O(n)$ — the set.

## Common Pitfalls

- **Self-comparison in the brute force.** Starting the inner loop at `j = i` (or `j = 0` with no guard) compares an element with itself and returns `true` for every input. Start at `i + 1`.
- **Adding before checking.** In Approach 3, calling `seen.add(x)` before `x in seen` makes the test always true. Check first, then add.
- **Mutating the caller's array.** Approach 2 sorts in place. If the input must be preserved, sort a copy and accept the $O(n)$ space.
- **No early exit in Approach 4.** It always builds the whole set, even when `nums[0] == nums[1]`. Same asymptotics, but slower on inputs with an early duplicate.

## The keystone

Approaches 3 and 4 spend $O(n)$ memory to turn an $O(n^2)$ scan into an $O(n)$ one — a hash set that remembers what you have already processed. Once "have I seen this value?" costs $O(1)$, [Two Sum](/citadel/dsa/two-sum), [Valid Anagram](/citadel/dsa/valid-anagram), and [Longest Consecutive Sequence](/citadel/dsa/longest-consecutive-sequence) all follow the same move.
