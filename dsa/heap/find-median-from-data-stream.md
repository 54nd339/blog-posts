---
title: Find Median from Data Stream - Two Heaps That Meet in the Middle
description: Reporting the running median as numbers arrive — keep the smaller half in a max-heap and the larger half in a min-heap, balanced in size, so the median is at the tops.
date: 2021-11-16
draft: false
slug: /dsa/find-median-from-data-stream
tags:
  - Competitive Programming
  - Heap
  - Design
---

Split the stream into a lower half and an upper half. If the lower half is a max-heap and the upper half a min-heap, and their sizes differ by at most one, the median is the top of the larger heap (odd count) or the average of the two tops (even count).

## Description

Implement `MedianFinder` with `addNum(num)` and `findMedian()`, where `findMedian` returns the median of all numbers added so far.

**Example**

```
addNum(1); addNum(2); findMedian() -> 1.5
addNum(3); findMedian() -> 2.0
```

**Constraints**

- $-10^5 \le \text{num} \le 10^5$
- `findMedian` is only called after at least one `addNum`.
- Up to $5 \times 10^4$ calls; follow-ups about bounded ranges exist.

## Prerequisites

- Min- and max-heaps (`heapq`, negating for max).
- The invariant of two size-balanced halves.

## Approach 1: Sorting

### Intuition

Keep all numbers in a list; sort on each `findMedian` and read the middle.

### Algorithm

1. `addNum`: append to a list.
2. `findMedian`: sort; return the middle element (odd) or the average of the two middle (even).

```python
class MedianFinder:
    def __init__(self):
        self.nums = []

    def addNum(self, num: int) -> None:
        self.nums.append(num)

    def findMedian(self) -> float:
        self.nums.sort()
        n = len(self.nums)
        if n % 2:
            return float(self.nums[n // 2])
        return (self.nums[n // 2 - 1] + self.nums[n // 2]) / 2
```

### Complexity

- **Time:** $O(1)$ for `addNum`; $O(n \log n)$ for `findMedian`.
- **Space:** $O(n)$.

## Approach 2: Two Heaps

### Intuition

`small` is a max-heap of the lower half (store negatives), `large` a min-heap of the upper half. Keep `len(small)` equal to `len(large)` or exactly one more. Then:

- odd total → median is `-small[0]`.
- even total → median is `(-small[0] + large[0]) / 2`.

### Algorithm

1. `addNum(num)`: push `num` onto `small` (as `-num`). Move `small`'s max over to `large` (so `large` only holds values `>=` everything in `small`). If `large` is now larger than `small`, move `large`'s min back to `small`.
2. `findMedian`: if `len(small) > len(large)`, return `-small[0]`; else return `(-small[0] + large[0]) / 2`.

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.small = []   # max-heap (negated) — lower half
        self.large = []   # min-heap — upper half

    def addNum(self, num: int) -> None:
        heapq.heappush(self.small, -num)
        heapq.heappush(self.large, -heapq.heappop(self.small))
        if len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))

    def findMedian(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2
```

### Complexity

- **Time:** $O(\log n)$ per `addNum`; $O(1)$ per `findMedian`.
- **Space:** $O(n)$.

## Common Pitfalls

- **Balancing without ordering.** You must first push onto `small`, then hand its max to `large` — that step is what guarantees every element of `small` is `<=` every element of `large`. Skipping it lets the two heaps interleave and the tops are no longer the middle elements.
- **Deciding which heap is bigger.** Fix a convention (here `small` may have one extra) and read the median accordingly.
- **Sign errors.** `small` stores negated values; the current max of the lower half is `-small[0]`.
- **Follow-up: values in a small fixed range.** A counting array (bucket) plus a running total gives $O(1)$ `addNum` and $O(\text{range})$ `findMedian`, beating the heaps when the range is tiny.

## The keystone

Two heaps facing each other maintain a *partition* of a stream into a lower and an upper half in $O(\log n)$ per insert, exposing the boundary elements in $O(1)$. Any "running k-th order statistic" or "median of a window" problem is a candidate for this facing-heaps structure.
