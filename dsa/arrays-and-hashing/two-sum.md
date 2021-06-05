---
title: Two Sum - The One-Pass Hash Map
description: Finding the two array indices whose values add to a target — brute force, a sorted two-pointer sweep, and the hash map that answers it in a single linear pass.
date: 2021-06-05
draft: false
slug: /dsa/two-sum
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Hash Table
---

The problem everyone starts with. The interesting version asks a different question: instead of "which other number pairs with `nums[i]`?", ask "have I already seen the number `nums[i]` needs?" — and make that lookup $O(1)$.

## Description

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one such pair exists, and you may not use the same element twice. The order of the two indices does not matter.

**Example**

```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]          # 2 + 7 == 9

Input:  nums = [3, 2, 4], target = 6
Output: [1, 2]

Input:  nums = [3, 3], target = 6
Output: [0, 1]
```

**Constraints**

- $2 \le \text{nums.length} \le 10^4$
- $-10^9 \le \text{nums}[i], \text{target} \le 10^9$
- Exactly one valid answer exists.

## Prerequisites

- [Arrays and hashing](/citadel/dsa/arrays-and-hashing).
- Hash maps: $O(1)$ average lookup and insertion, keyed here by value.
- [Two pointers](/citadel/dsa/two-pointers) on a sorted array (for Approach 2).

## Approach 1: Brute Force

### Intuition

Try every pair and check whether the two values hit the target.

### Algorithm

1. Loop `i` over every index.
2. For each `i`, loop `j` from `i + 1` to the end.
3. If `nums[i] + nums[j] == target`, return `[i, j]`.

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []
```

### Complexity

- **Time:** $O(n^2)$ — roughly $n^2/2$ pairs.
- **Space:** $O(1)$.

## Approach 2: Sorting

### Intuition

On a sorted array, a pair of pointers at the two ends can decide, from the current sum, which end to move: too small means raise the low pointer, too large means lower the high pointer. Sorting loses the original indices, so pair each value with its index first.

### Algorithm

1. Build a list of `(value, original_index)` pairs and sort it by value.
2. Set `lo = 0`, `hi = n - 1`.
3. While `lo < hi`: let `s` be the sum of the two values. If `s == target`, return the two stored indices. If `s < target`, `lo += 1`; otherwise `hi -= 1`.

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    arr = sorted((v, i) for i, v in enumerate(nums))
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        s = arr[lo][0] + arr[hi][0]
        if s == target:
            return [arr[lo][1], arr[hi][1]]
        if s < target:
            lo += 1
        else:
            hi -= 1
    return []
```

### Complexity

- **Time:** $O(n \log n)$ for the sort; the sweep is $O(n)$.
- **Space:** $O(n)$ for the paired array.

## Approach 3: Hash Map (Two Pass)

### Intuition

If you know every value's index up front, then for each element you can look up whether its complement `target - nums[i]` exists — and confirm it is a *different* index.

### Algorithm

1. First pass: store `value -> index` for every element in a map (later duplicates overwrite earlier ones).
2. Second pass: for each `i`, compute `need = target - nums[i]`. If `need` is in the map at an index `j != i`, return `[i, j]`.

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    index = {x: i for i, x in enumerate(nums)}
    for i, x in enumerate(nums):
        j = index.get(target - x, -1)
        if j != -1 and j != i:
            return [i, j]
    return []
```

### Complexity

- **Time:** $O(n)$ — two passes, $O(1)$ per element.
- **Space:** $O(n)$ for the map.

## Approach 4: Hash Map (One Pass)

### Intuition

You do not need the whole map before you start querying it. Check for the complement among the elements seen *so far*, then add the current element.

### Algorithm

1. Create an empty map `seen` of `value -> index`.
2. For each `i` with value `x`: if `target - x` is in `seen`, return `[seen[target - x], i]`.
3. Otherwise record `seen[x] = i` and continue.

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
    return []
```

### Complexity

- **Time:** $O(n)$ — a single pass.
- **Space:** $O(n)$ for the map.

## Common Pitfalls

- **Inserting before checking (Approach 4).** When `target == 2 * nums[i]` and this is the first sighting of that value, checking after the insert finds the element you just added and returns `[i, i]`. Check first, insert second.
- **Losing indices in Approach 2.** After sorting you must return the *stored* original indices, not `lo` and `hi`.
- **The `j != i` guard in Approach 2-pass.** Without it, a value whose complement is itself matches its own index.
- **Returning values instead of indices.** The problem asks for positions.

## The keystone

"Remember what you have seen, query for what you need" in a single pass is the backbone of [Contains Duplicate](/citadel/dsa/contains-duplicate), [Group Anagrams](/citadel/dsa/group-anagrams), and the sliding-window problems later in the set. When the array is already sorted, the [two-pointer](/citadel/dsa/two-sum-ii) version trades the map for $O(1)$ space.
