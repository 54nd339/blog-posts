---
title: Kth Largest Element in an Array - Quickselect Gets It in Linear Time
description: The k-th largest value in an unsorted array — sorting is O(n log n), a size-k heap is O(n log k), and quickselect partitions to the answer in average O(n).
date: 2021-11-13
draft: false
slug: /dsa/kth-largest-element-in-an-array
tags:
  - Competitive Programming
  - Heap
  - Quickselect
---

You want one order statistic, not a sorted array. Quickselect uses the partition step of quicksort but only recurses into the side that contains the answer, giving average linear time.

## Description

Given an integer array `nums` and an integer `k`, return the k-th largest element. It is the k-th largest in sorted order, not necessarily distinct.

**Example**

```
Input:  nums = [3,2,1,5,6,4], k = 2
Output: 5

Input:  nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

**Constraints**

- $1 \le k \le \text{nums.length} \le 10^5$
- $-10^4 \le \text{nums}[i] \le 10^4$

## Prerequisites

- The quicksort partition routine.
- Size-`k` heaps.

## Approach 1: Sorting

### Intuition

Sort ascending; the k-th largest sits at index `n - k`.

### Algorithm

1. Sort `nums`.
2. Return `nums[len(nums) - k]`.

```python
def findKthLargest(nums: list[int], k: int) -> int:
    nums.sort()
    return nums[len(nums) - k]
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(1)$ or $O(n)$.

## Approach 2: Min-Heap

### Intuition

Keep a min-heap of the `k` largest values seen. Its root is the answer.

### Algorithm

1. For each `x`: push `x`; if `len(heap) > k`, pop the minimum.
2. Return `heap[0]`.

```python
import heapq

def findKthLargest(nums: list[int], k: int) -> int:
    heap: list[int] = []
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]
```

### Complexity

- **Time:** $O(n \log k)$.
- **Space:** $O(k)$.

`heapq.nlargest(k, nums)[-1]` is the one-line form.

## Approach 3: Quickselect

### Intuition

Convert to the 0-based ascending index `target = n - k`. Partition around a random pivot; if the pivot lands at `target`, it is the answer; otherwise recurse into the side containing `target`.

### Algorithm

1. `target = len(nums) - k`.
2. `partition(l, r)`: swap a random pivot to `r`; move elements `<= pivot` to the front; place the pivot; return its index `p`.
3. Recurse/loop: if `p == target`, return `nums[p]`; if `p < target`, search `[p + 1, r]`; else `[l, p - 1]`.

```python
import random

def findKthLargest(nums: list[int], k: int) -> int:
    target = len(nums) - k

    def partition(l, r):
        i = random.randint(l, r)
        nums[i], nums[r] = nums[r], nums[i]
        pivot = nums[r]
        store = l
        for j in range(l, r):
            if nums[j] <= pivot:
                nums[j], nums[store] = nums[store], nums[j]
                store += 1
        nums[store], nums[r] = nums[r], nums[store]
        return store

    l, r = 0, len(nums) - 1
    while True:
        p = partition(l, r)
        if p == target:
            return nums[p]
        if p < target:
            l = p + 1
        else:
            r = p - 1
```

### Complexity

- **Time:** $O(n)$ average — work halves each step; $O(n^2)$ worst case, made unlikely by a random pivot.
- **Space:** $O(1)$ (iterative, in place).

## Approach 4: Quickselect with Median-of-Three

### Intuition

Same algorithm, choosing the pivot as the median of `nums[l]`, `nums[mid]`, `nums[r]` to make pathological inputs rarer without randomness.

### Algorithm

1. Compute `target = k - 1` over a *descending* partition (elements greater than the pivot move left), or keep ascending with `target = n - k`.
2. Pivot = median of the three sampled values, swapped to `r`.
3. Partition and narrow `[l, r]` toward `target` as in Approach 3.

```python
def findKthLargest(nums: list[int], k: int) -> int:
    target = len(nums) - k

    def med3(l, mid, r):
        a, b, c = nums[l], nums[mid], nums[r]
        if a <= b <= c or c <= b <= a:
            return mid
        if b <= a <= c or c <= a <= b:
            return l
        return r

    def partition(l, r):
        mid = (l + r) // 2
        pi = med3(l, mid, r)
        nums[pi], nums[r] = nums[r], nums[pi]
        pivot = nums[r]
        store = l
        for j in range(l, r):
            if nums[j] <= pivot:
                nums[j], nums[store] = nums[store], nums[j]
                store += 1
        nums[store], nums[r] = nums[r], nums[store]
        return store

    l, r = 0, len(nums) - 1
    while True:
        p = partition(l, r)
        if p == target:
            return nums[p]
        if p < target:
            l = p + 1
        else:
            r = p - 1
```

### Complexity

- **Time:** $O(n)$ average, $O(n^2)$ worst case (much rarer).
- **Space:** $O(1)$.

## Common Pitfalls

- **Index conversion.** The k-th *largest* is ascending index `n - k`. Off-by-one here is the top bug.
- **Deterministic pivot on sorted input.** Always taking `nums[r]` degrades to $O(n^2)$ on already-sorted or reverse-sorted arrays; randomise or use median-of-three.
- **Recursing into both halves.** Quickselect recurses into *one* side only — that is the whole speedup over quicksort.
- **Duplicates.** The array may repeat values; quickselect and the heap both handle this since they compare, not deduplicate.

## The keystone

Selection is cheaper than sorting: quickselect's "partition, then recurse into one side" gives expected $O(n)$ for any order statistic. It is the same partition used by [K Closest Points to Origin](/citadel/dsa/k-closest-points-to-origin) and the standard answer to "find the median without sorting".
