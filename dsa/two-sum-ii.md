---
title: Two Sum II - What a Sorted Array Buys You
description: Finding two values in a sorted array that add to a target with O(1) extra space — brute force, per-element binary search, a hash map, and the converging two pointers the sortedness makes possible.
date: 2021-06-04
draft: false
slug: /dsa/two-sum-ii
tags:
  - Competitive Programming
  - Two Pointers
  - Binary Search
---

The original [Two Sum](/citadel/dsa/two-sum) leans on a hash map for its linear pass, at $O(n)$ space. Here the array is sorted, and that single fact replaces the map with two pointers and a rule for which one to move.

## Description

Given a **1-indexed** array `numbers` sorted in non-decreasing order, return the two indices whose values sum to `target`. Exactly one solution exists, you may not use an element twice, and you must use only $O(1)$ extra space.

**Example**

```
Input:  numbers = [2, 7, 11, 15], target = 9
Output: [1, 2]

Input:  numbers = [2, 3, 4], target = 6
Output: [1, 3]
```

**Constraints**

- $2 \le \text{numbers.length} \le 3 \times 10^4$
- $-1000 \le \text{numbers}[i], \text{target} \le 1000$
- `numbers` is sorted non-decreasing; exactly one valid answer; $O(1)$ extra space.

## Prerequisites

- [Binary search](/citadel/dsa/binary-search) on a sorted range (Approach 2).
- [Two pointers](/citadel/dsa/two-pointers) and the monotonic move rule.

## Approach 1: Brute Force

### Intuition

Ignore the sortedness and try all pairs.

### Algorithm

1. For each `i` from `0` to `n - 1`, for each `j` from `i + 1` to `n - 1`.
2. If `numbers[i] + numbers[j] == target`, return `[i + 1, j + 1]`.

```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    n = len(numbers)
    for i in range(n):
        for j in range(i + 1, n):
            if numbers[i] + numbers[j] == target:
                return [i + 1, j + 1]
    return []
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Binary Search

### Intuition

For each `i`, the partner value is `target - numbers[i]`, and the array is sorted, so binary-search for it in the slice after `i`.

### Algorithm

1. For each `i`, compute `need = target - numbers[i]`.
2. Binary-search `numbers[i+1 .. n-1]` for `need`.
3. If found at index `j`, return `[i + 1, j + 1]`.

```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    n = len(numbers)
    for i in range(n):
        need = target - numbers[i]
        lo, hi = i + 1, n - 1
        while lo <= hi:
            mid = (lo + hi) // 2
            if numbers[mid] == need:
                return [i + 1, mid + 1]
            if numbers[mid] < need:
                lo = mid + 1
            else:
                hi = mid - 1
    return []
```

### Complexity

- **Time:** $O(n \log n)$ — `n` searches of $O(\log n)$ each.
- **Space:** $O(1)$.

## Approach 3: Hash Map

### Intuition

The classic Two Sum trick still works: as you scan, look up whether each element's complement has already been seen.

### Algorithm

1. Keep a map of `value -> 1-indexed position`.
2. For each `i`, compute `need = target - numbers[i]`. If `need` is in the map, return `[map[need], i + 1]`.
3. Otherwise store `numbers[i] -> i + 1`.

```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    seen = {}
    for i, x in enumerate(numbers):
        if target - x in seen:
            return [seen[target - x], i + 1]
        seen[x] = i + 1
    return []
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — which violates the problem's $O(1)$ requirement, so this is a stepping stone, not the accepted answer.

## Approach 4: Two Pointers

### Intuition

Put `lo` at the first element and `hi` at the last. Their sum is the largest-plus-smallest available. Too small: raise `lo` (every other element is at least `numbers[lo]`). Too large: lower `hi`. Each move discards an element that cannot be in any solution.

### Algorithm

1. `lo = 0`, `hi = n - 1`.
2. While `lo < hi`: let `s = numbers[lo] + numbers[hi]`.
   - `s == target` → return `[lo + 1, hi + 1]`.
   - `s < target` → `lo += 1`.
   - `s > target` → `hi -= 1`.

```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    lo, hi = 0, len(numbers) - 1
    while lo < hi:
        s = numbers[lo] + numbers[hi]
        if s == target:
            return [lo + 1, hi + 1]
        if s < target:
            lo += 1
        else:
            hi -= 1
    return []
```

### Complexity

- **Time:** $O(n)$ — `lo` and `hi` together move `n` steps.
- **Space:** $O(1)$.

## Common Pitfalls

- **1-indexing.** The answer is `[lo + 1, hi + 1]`, not `[lo, hi]`. Off-by-one here is the most common wrong answer.
- **Binary search over the whole array (Approach 2).** Searching from index `0` can match `numbers[i]` with itself when `target == 2 * numbers[i]`. Restrict the search to indices after `i`.
- **Moving the wrong pointer (Approach 4).** `hi -= 1` when the sum is too small discards the largest value — the one you needed — and you can walk past the answer.

## The keystone

A sorted array turns "search for a value" into "decide a direction": opposite-end pointers with a monotonic move rule replace an auxiliary hash structure and drop space to $O(1)$. That is exactly the upgrade [3Sum](/citadel/dsa/3sum) uses for its inner loop.
