---
title: K Closest Points to Origin - Partial Order Is Enough
description: The k points nearest the origin — you never need the points fully sorted, so a size-k max-heap or a quickselect partition beats an O(n log n) sort.
date: 2021-11-13
draft: false
slug: /dsa/k-closest-points-to-origin
tags:
  - Competitive Programming
  - Heap
  - Quickselect
---

The output is a *set* of `k` points, unordered. That means you only need to separate the `k` closest from the rest — a partial order — which a heap or quickselect delivers faster than a full sort. Compare *squared* distances to skip the square root.

## Description

Given an array `points` where `points[i] = [x, y]`, and an integer `k`, return the `k` points closest to the origin `(0, 0)` by Euclidean distance, in any order.

**Example**

```
Input:  points = [[1,3],[-2,2]], k = 1
Output: [[-2,2]]

Input:  points = [[3,3],[5,-1],[-2,4]], k = 2
Output: [[3,3],[-2,4]]
```

**Constraints**

- $1 \le k \le \text{points.length} \le 10^4$
- $-10^4 \le x, y \le 10^4$

## Prerequisites

- Binary heaps; the size-`k` heap idiom.
- Quickselect (partition-based selection) — see [Kth Largest Element in an Array](/citadel/dsa/kth-largest-element-in-an-array).

## Approach 1: Sorting

### Intuition

Sort all points by squared distance; take the first `k`.

### Algorithm

1. Key each point by `x*x + y*y`.
2. Sort ascending; return `points[:k]`.

```python
def kClosest(points: list[list[int]], k: int) -> list[list[int]]:
    points.sort(key=lambda p: p[0] * p[0] + p[1] * p[1])
    return points[:k]
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$ (or $O(1)$ with an in-place sort).

## Approach 2: Min-Heap

### Intuition

Heapify all `(dist, point)` pairs, then pop the `k` smallest.

### Algorithm

1. Build a list of `(x*x + y*y, x, y)`; `heapify` it.
2. Pop `k` times, collecting the points.

```python
import heapq

def kClosest(points: list[list[int]], k: int) -> list[list[int]]:
    heap = [(x * x + y * y, x, y) for x, y in points]
    heapq.heapify(heap)
    return [[x, y] for _, x, y in heapq.nsmallest(k, heap)]
```

### Complexity

- **Time:** $O(n + k \log n)$.
- **Space:** $O(n)$.

## Approach 3: Max-Heap of size k

### Intuition

Keep a max-heap of the `k` smallest-distance points seen so far. Push each point (with negated distance); if the heap exceeds `k`, pop the farthest.

### Algorithm

1. For each point: `heappush(heap, (-(dist), x, y))`; if `len(heap) > k`, `heappop`.
2. Return the points left in the heap.

```python
import heapq

def kClosest(points: list[list[int]], k: int) -> list[list[int]]:
    heap: list[tuple[int, int, int]] = []
    for x, y in points:
        heapq.heappush(heap, (-(x * x + y * y), x, y))
        if len(heap) > k:
            heapq.heappop(heap)
    return [[x, y] for _, x, y in heap]
```

### Complexity

- **Time:** $O(n \log k)$.
- **Space:** $O(k)$.

## Approach 4: Quickselect

### Intuition

Partition the array around a pivot distance so that the `k` smallest end up in the first `k` slots — without sorting either side. Average $O(n)$.

### Algorithm

1. `dist(p) = p[0]**2 + p[1]**2`.
2. `partition(l, r)`: choose `points[r]` as pivot; move all points with distance `<= pivot` to the front; return the pivot's final index `p`.
3. Loop: partition `[l, r]`; if `p == k`, stop; if `p < k`, `l = p + 1`; else `r = p - 1`.
4. Return `points[:k]`.

```python
import random

def kClosest(points: list[list[int]], k: int) -> list[list[int]]:
    def dist(p):
        return p[0] * p[0] + p[1] * p[1]

    def partition(l, r):
        pivot_i = random.randint(l, r)
        points[pivot_i], points[r] = points[r], points[pivot_i]
        pivot = dist(points[r])
        store = l
        for i in range(l, r):
            if dist(points[i]) <= pivot:
                points[i], points[store] = points[store], points[i]
                store += 1
        points[store], points[r] = points[r], points[store]
        return store

    l, r = 0, len(points) - 1
    while l <= r:
        p = partition(l, r)
        if p == k:
            break
        if p < k:
            l = p + 1
        else:
            r = p - 1
    return points[:k]
```

### Complexity

- **Time:** $O(n)$ average, $O(n^2)$ worst case.
- **Space:** $O(1)$ (in place).

## Common Pitfalls

- **Computing the actual distance with `sqrt`.** Squared distance preserves order and avoids floats.
- **Quickselect target index.** You want the partition point to land at index `k` (0-based), so the first `k` slots hold the `k` closest; a common slip is aiming for `k - 1` or `k + 1`.
- **Min-heap of all n vs max-heap of k.** Both work; the size-`k` max-heap wins when `k` is much smaller than `n` and is the streaming-friendly choice.

## The keystone

"Return the k smallest/largest, order doesn't matter" is a *selection* problem, not a sorting one. Quickselect gets it in average $O(n)$; a size-`k` heap gets it in $O(n \log k)$ and also handles a stream. Reserve the full sort for when you actually need everything ordered.
