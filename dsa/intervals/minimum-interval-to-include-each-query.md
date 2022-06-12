---
title: Minimum Interval to Include Each Query - Offline, Sorted, Heap-Fed
description: For each query point, the size of the smallest interval covering it — answer queries in increasing order, adding intervals as they become active and popping a min-heap by interval size.
date: 2022-06-12
draft: false
slug: /dsa/minimum-interval-to-include-each-query
tags:
  - Competitive Programming
  - Intervals
  - Heap
---

Answer the queries offline, sorted ascending. Sort intervals by start. As the query point advances, push every interval that has started into a heap keyed by size; lazily discard from the heap's top any interval that has already ended. The heap's minimum is the answer for that query.

## Description

Given `intervals` `[left, right]` and an array `queries`, for each query `q` return the size (`right - left + 1`) of the smallest interval with `left <= q <= right`, or `-1` if none contains `q`. Answers follow the original query order.

**Example**

```
Input:  intervals = [[1,4],[2,4],[3,6],[4,4]], queries = [2,3,4,5]
Output: [3,3,1,2]

Input:  intervals = [[2,3],[2,5],[1,8],[20,25]], queries = [2,19,5,22]
Output: [2,-1,4,3]
```

**Constraints**

- $1 \le \text{intervals.length}, \text{queries.length} \le 10^5$
- $1 \le \text{left} \le \text{right} \le 10^7$; $1 \le q \le 10^7$

## Prerequisites

- Offline processing: reorder queries, restore order at the end.
- Min-heap keyed by interval size, with lazy deletion of expired intervals.

## Approach 1: Sort queries, sweep intervals into a heap

### Intuition

Process queries from smallest to largest. Maintain a pointer into the start-sorted intervals; for the current query `q`, push all intervals with `left <= q` (heap entry `(size, right)`). Then pop while the top's `right < q` (it cannot cover `q` or any larger query). The remaining top's `size` is the answer.

### Algorithm

1. Sort `intervals` by `left`. Pair each query with its original index and sort by value.
2. `heap = []`, `i = 0`, `ans = [-1] * len(queries)`.
3. For each `(q, idx)` in sorted queries: while `i < len(intervals)` and `intervals[i][0] <= q`: push `(intervals[i][1] - intervals[i][0] + 1, intervals[i][1])`; `i += 1`. While `heap` and `heap[0][1] < q`: pop. If `heap`: `ans[idx] = heap[0][0]`.
4. Return `ans`.

```python
import heapq

def minInterval(intervals: list[list[int]], queries: list[int]) -> list[int]:
    intervals.sort(key=lambda x: x[0])
    sorted_queries = sorted((q, idx) for idx, q in enumerate(queries))

    ans = [-1] * len(queries)
    heap = []
    i = 0
    for q, idx in sorted_queries:
        while i < len(intervals) and intervals[i][0] <= q:
            l, r = intervals[i]
            heapq.heappush(heap, (r - l + 1, r))
            i += 1
        while heap and heap[0][1] < q:
            heapq.heappop(heap)
        if heap:
            ans[idx] = heap[0][0]
    return ans
```

### Complexity

- **Time:** $O((n + m) \log n)$ with `n` intervals and `m` queries.
- **Space:** $O(n + m)$.

## Common Pitfalls

- **Not restoring the original query order.** Carry each query's index through the sort and write answers into `ans[idx]`.
- **Popping intervals by `left` instead of `right`.** An interval leaves the heap when its `right` is below the current query point; sorting the heap by size while discarding by `right` is the whole trick.
- **Eager deletion.** You cannot remove an arbitrary interval from a heap cheaply — only the top. Lazy popping works because queries are non-decreasing, so an interval discarded now is useless for every later query too.
- **Size formula.** It is `right - left + 1` (inclusive integer length), not `right - left`.

## The keystone

Offline + sort + a heap that grows monotonically and is cleaned only at the top is the pattern for "answer many range queries against many intervals". The monotone query order is what makes lazy deletion sound — the same reason a sliding-window maximum can lazily drop stale indices.
