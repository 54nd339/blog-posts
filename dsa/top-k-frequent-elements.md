---
title: Top K Frequent Elements - When Bucket Sort Beats a Heap
description: Returning the k most common values — a full sort is easy, a size-k min-heap trims it to O(n log k), and bucket sort by frequency reaches linear time because a frequency cannot exceed n.
date: 2021-04-26
draft: false
slug: /dsa/top-k-frequent-elements
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Heap
---

Every approach starts the same way — a frequency map in one pass. The differences are all in how you extract the top `k` from that map.

## Description

Given an integer array `nums` and an integer `k`, return the `k` values that appear most often. The answer is guaranteed unique and may be returned in any order.

**Example**

```
Input:  nums = [1, 1, 1, 2, 2, 3], k = 2
Output: [1, 2]

Input:  nums = [7], k = 1
Output: [7]
```

**Constraints**

- $1 \le \text{nums.length} \le 10^5$
- `k` is in the range `[1, number of distinct elements in nums]`.

## Prerequisites

- Hash maps / `Counter` for frequencies.
- Binary heaps (`heapq`): $O(\log n)$ push and pop, and the size-capped-heap trick for top-k.
- Counting / bucket sort: linear time when the key is a bounded integer.

## Approach 1: Sorting

### Intuition

Count occurrences, order the distinct values by count, take the largest `k`.

### Algorithm

1. Build a frequency map.
2. Make a list of `[count, value]` pairs.
3. Sort it ascending by count.
4. Pop from the end (highest count), collecting values until you have `k`.

```python
from collections import Counter

def topKFrequent(nums: list[int], k: int) -> list[int]:
    freq = Counter(nums)
    pairs = sorted([count, val] for val, count in freq.items())
    res = []
    while len(res) < k:
        res.append(pairs.pop()[1])
    return res
```

### Complexity

- **Time:** $O(n \log n)$ — dominated by sorting up to `n` distinct values.
- **Space:** $O(n)$ for the map and the pair list.

## Approach 2: Min-Heap

### Intuition

You only need the top `k`, not a full ordering. Keep a min-heap of at most `k` `(count, value)` pairs; whenever it overflows, evict the smallest count. What survives is the `k` most frequent.

### Algorithm

1. Build a frequency map.
2. For each `(value, count)`: push `(count, value)` onto the heap; if the heap size exceeds `k`, pop the minimum.
3. Drain the heap into the result list.

```python
import heapq
from collections import Counter

def topKFrequent(nums: list[int], k: int) -> list[int]:
    freq = Counter(nums)
    heap: list[tuple[int, int]] = []
    for value, count in freq.items():
        heapq.heappush(heap, (count, value))
        if len(heap) > k:
            heapq.heappop(heap)
    return [value for _, value in heap]
```

### Complexity

- **Time:** $O(n \log k)$ — each of up to `n` distinct values costs a $\log k$ heap operation.
- **Space:** $O(n + k)$ — the map plus the heap.

## Approach 3: Bucket Sort

### Intuition

A frequency is an integer between `1` and `n`. Index buckets by frequency, drop each value into its bucket, then read buckets from high frequency down.

### Algorithm

1. Build a frequency map.
2. Create `buckets`, a list of `n + 1` empty lists; `buckets[f]` holds values with frequency `f`.
3. Place each value into `buckets[freq[value]]`.
4. Iterate `f` from `n` down to `1`, appending each value in `buckets[f]` to the result, stopping once it has `k` entries.

```python
from collections import Counter

def topKFrequent(nums: list[int], k: int) -> list[int]:
    freq = Counter(nums)
    buckets: list[list[int]] = [[] for _ in range(len(nums) + 1)]
    for value, count in freq.items():
        buckets[count].append(value)

    res: list[int] = []
    for count in range(len(buckets) - 1, 0, -1):
        for value in buckets[count]:
            res.append(value)
            if len(res) == k:
                return res
    return res
```

### Complexity

- **Time:** $O(n)$ — counting is $O(n)$, filling buckets $O(\text{distinct})$, the final sweep visits at most `n + 1` buckets.
- **Space:** $O(n)$ for the bucket array.

## Common Pitfalls

- **Heap direction (Approach 2).** `heapq` is a min-heap, so the pair must be `(count, value)` for the *least frequent* to be evicted. Pushing all pairs then popping `k` times is $O(n \log n)$, no better than sorting — the win is capping the heap at `k`.
- **Walking buckets low-to-high (Approach 3).** Iterate from `len(buckets) - 1` down to `1`; index `0` is unused because no value has frequency zero.
- **`Counter(nums).most_common(k)`** does the job in one call but returns `(value, count)` pairs — you still need to project out the values.

## The keystone

Bucket sort wins whenever the sort key is an integer from a small known range — here a frequency bounded by `n` — turning an $O(n \log n)$ ordering into an $O(n)$ counting pass. The size-capped heap is the one to keep for *streaming* data, where `n` is unbounded but `k` is small; it returns in [Kth Largest Element in a Stream](/citadel/dsa/kth-largest-element-in-a-stream).
