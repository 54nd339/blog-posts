---
title: Find the Duplicate Number - The Array Is a Linked List
description: One repeated value in an array of n+1 numbers from 1 to n, without modifying it or using extra space — reading index to value turns the array into a linked list whose cycle entry is the duplicate.
date: 2021-09-14
draft: false
slug: /dsa/find-the-duplicate-number
tags:
  - Competitive Programming
  - Linked List
  - Binary Search
---

The constraints — don't modify the array, use $O(1)$ extra space — rule out sorting and hashing. The trick: treat each value as a "next index" pointer. Because two array slots hold the same value, two pointers lead into the same node, forming a cycle whose entrance *is* the duplicate.

## Description

Given an array `nums` of `n + 1` integers where each is in `[1, n]`, exactly one value is repeated (possibly many times). Return that value without modifying the array and using only constant extra space.

**Example**

```
Input:  nums = [1, 3, 4, 2, 2]
Output: 2

Input:  nums = [3, 1, 3, 4, 2]
Output: 3
```

**Constraints**

- $1 \le n \le 10^5$; `nums.length == n + 1`
- $1 \le \text{nums}[i] \le n$
- Only one value repeats; it may appear more than twice.

## Prerequisites

- [Linked List Cycle](/citadel/dsa/linked-list-cycle) — Floyd's algorithm and finding the cycle entry.
- [Binary search](/citadel/dsa/binary-search) on the answer (counting argument).

## Approach 1: Sorting

### Intuition

After sorting, the duplicate is adjacent to itself.

### Algorithm

1. Sort a copy of `nums`.
2. Return the first element equal to its predecessor.

```python
def findDuplicate(nums: list[int]) -> int:
    s = sorted(nums)
    for i in range(1, len(s)):
        if s[i] == s[i - 1]:
            return s[i]
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$ for the copy (sorting in place would modify the input, which is disallowed).

## Approach 2: Hash Set

### Intuition

Track seen values; the first repeat is the answer.

### Algorithm

1. Empty set. For each `x`: if `x` in set, return `x`; else add it.

```python
def findDuplicate(nums: list[int]) -> int:
    seen = set()
    for x in nums:
        if x in seen:
            return x
        seen.add(x)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Marking a Frequency Array

### Intuition

Keep a separate array of counts (or booleans) indexed by value.

### Algorithm

1. `seen = [False] * (n + 1)`.
2. For each `x`: if `seen[x]`, return `x`; else set `seen[x] = True`.

```python
def findDuplicate(nums: list[int]) -> int:
    seen = [False] * len(nums)
    for x in nums:
        if seen[x]:
            return x
        seen[x] = True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 4: Negative Marking

### Intuition

Use the sign of `nums[abs(x) - 1]` as a "seen" flag. This *does* mutate the array (restore it afterwards if needed), so it bends the rules, but uses $O(1)$ extra space.

### Algorithm

1. For each `x` in `nums`: let `i = abs(x) - 1`. If `nums[i] < 0`, return `abs(x)`. Else negate `nums[i]`.

```python
def findDuplicate(nums: list[int]) -> int:
    for x in nums:
        i = abs(x) - 1
        if nums[i] < 0:
            return abs(x)
        nums[i] = -nums[i]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ extra (mutates input).

## Approach 5: Binary Search (counting)

### Intuition

For a candidate `m`, count how many array values are `<= m`. If that count exceeds `m`, the duplicate is in `[1, m]`; otherwise it is in `[m + 1, n]`. The count-vs-`m` relation is monotone.

### Algorithm

1. `lo = 1`, `hi = n`.
2. While `lo < hi`: `m = (lo + hi) // 2`; `count = sum(1 for x in nums if x <= m)`. If `count > m`, `hi = m`; else `lo = m + 1`.
3. Return `lo`.

```python
def findDuplicate(nums: list[int]) -> int:
    lo, hi = 1, len(nums) - 1
    while lo < hi:
        m = (lo + hi) // 2
        count = sum(1 for x in nums if x <= m)
        if count > m:
            hi = m
        else:
            lo = m + 1
    return lo
```

### Complexity

- **Time:** $O(n \log n)$ — a linear count per binary-search step.
- **Space:** $O(1)$.

## Approach 6: Bit Manipulation

### Intuition

For each bit position, count set bits across `nums` and across `1..n`. Wherever `nums` has more, that bit belongs to the duplicate.

### Algorithm

1. For each bit `b` in `0..31`:
   - `nums_count` = numbers in `nums` with bit `b` set; `base_count` = numbers in `1..n` with bit `b` set.
   - If `nums_count > base_count`, set bit `b` in the answer.

```python
def findDuplicate(nums: list[int]) -> int:
    n = len(nums) - 1
    res = 0
    for b in range(32):
        mask = 1 << b
        nums_count = sum(1 for x in nums if x & mask)
        base_count = sum(1 for x in range(1, n + 1) if x & mask)
        if nums_count > base_count:
            res |= mask
    return res
```

### Complexity

- **Time:** $O(32n)$.
- **Space:** $O(1)$.

## Approach 7: Fast and Slow Pointers

### Intuition

Follow `i → nums[i]` as pointers. Since values lie in `[1, n]` and there are `n + 1` of them, this "linked list" (starting from index `0`) must contain a cycle, and the cycle's entry node is the repeated value. Floyd's algorithm finds it.

### Algorithm

1. `slow = fast = nums[0]`. Advance `slow = nums[slow]`, `fast = nums[nums[fast]]` until they meet.
2. Reset `slow = nums[0]`. Advance both by one (`slow = nums[slow]`, `fast = nums[fast]`) until they meet again.
3. That meeting value is the duplicate.

```python
def findDuplicate(nums: list[int]) -> int:
    slow = fast = nums[0]
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]
        if slow == fast:
            break
    slow = nums[0]
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]
    return slow
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$, and the array is not modified.

## Common Pitfalls

- **Approach 7 needs index `0` as a safe start.** Values are in `[1, n]`, so no value points back to index `0` — it is outside every cycle, guaranteeing the tortoise/hare phase-two math holds.
- **Phase two moves *both* pointers one step.** After the first meeting, resetting `slow` to `nums[0]` and advancing both by one lands them on the cycle entrance.
- **Binary search counts values `<= m`, not `== m`.** The pigeonhole argument is about the cumulative count exceeding `m`.

## The keystone

An array whose entries are valid indices *is* a functional graph — one out-edge per node — and a repeated value is a merge point, hence a cycle. Recasting an array problem as a linked-list problem to inherit Floyd's $O(1)$-space cycle machinery is the move worth remembering.
