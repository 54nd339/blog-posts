---
title: Kth Largest Element in a Stream - A Heap of Size K
description: Reporting the k-th largest value after each insertion — re-sorting every time is wasteful, but a min-heap capped at k elements keeps its root equal to the answer.
date: 2021-11-06
draft: false
slug: /dsa/kth-largest-element-in-a-stream
tags:
  - Competitive Programming
  - Heap
  - Design
---

If you keep exactly the `k` largest values seen so far in a min-heap, the smallest of them — the heap's root — *is* the k-th largest overall. Each `add` is one push and possibly one pop.

## Description

Implement `KthLargest(k, nums)` and `add(val)`, where `add` returns the k-th largest element among all values added so far (including the initial `nums`).

**Example**

```
KthLargest(3, [4,5,8,2])
add(3)  -> 4
add(5)  -> 5
add(10) -> 5
add(9)  -> 8
add(4)  -> 8
```

**Constraints**

- $1 \le k \le 10^4$; $0 \le \text{nums.length} \le 10^4$
- $-10^4 \le \text{nums}[i], \text{val} \le 10^4$
- Up to $10^4$ `add` calls; there are always at least `k` elements when `add` is queried.

## Prerequisites

- Binary heaps (`heapq`) and the size-capped-heap idiom.

## Approach 1: Sorting

### Intuition

Keep all values in a list; on each `add`, sort and index `len - k`.

### Algorithm

1. Store `k` and the running list.
2. `add(val)`: append `val`, sort ascending, return `arr[len(arr) - k]`.

```python
class KthLargest:
    def __init__(self, k: int, nums: list[int]):
        self.k = k
        self.arr = nums

    def add(self, val: int) -> int:
        self.arr.append(val)
        self.arr.sort()
        return self.arr[len(self.arr) - self.k]
```

### Complexity

- **Time:** $O(n \log n)$ per `add`.
- **Space:** $O(n)$.

## Approach 2: Min-Heap

### Intuition

Maintain a min-heap holding only the `k` largest values. Its root is the k-th largest. On `add`, push the new value; if the heap grew past `k`, pop the smallest.

### Algorithm

1. Heapify `nums`; while `len(heap) > k`, pop.
2. `add(val)`: `heappush(heap, val)`; if `len(heap) > k`, `heappop(heap)`; return `heap[0]`.

```python
import heapq

class KthLargest:
    def __init__(self, k: int, nums: list[int]):
        self.k = k
        self.heap = nums
        heapq.heapify(self.heap)
        while len(self.heap) > k:
            heapq.heappop(self.heap)

    def add(self, val: int) -> int:
        heapq.heappush(self.heap, val)
        if len(self.heap) > self.k:
            heapq.heappop(self.heap)
        return self.heap[0]
```

### Complexity

- **Time:** $O(\log k)$ per `add`; $O(n \log n)$ for the constructor.
- **Space:** $O(k)$.

## Common Pitfalls

- **Using a max-heap and popping `k - 1` times per query.** That is $O(k \log n)$ per `add`; the point of the min-heap is that it *stays* size `k`.
- **Forgetting to shrink the heap in the constructor.** After heapifying `nums` you must pop down to `k`, or the first `add` reads the wrong root.
- **Returning after the push but before the pop.** Push, then rebalance to size `k`, then read `heap[0]`.

## The keystone

"The k-th largest" is exactly "the minimum of the k largest", so a size-`k` min-heap answers it in $O(1)$ with $O(\log k)$ maintenance. The capped-heap pattern is the go-to for any streaming top-k question ([Top K Frequent Elements](/citadel/dsa/top-k-frequent-elements), [K Closest Points to Origin](/citadel/dsa/k-closest-points-to-origin)).
