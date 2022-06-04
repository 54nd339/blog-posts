---
title: Non-Overlapping Intervals - Keep the One That Ends Soonest
description: The fewest intervals to remove so none overlap — sort by end, greedily keep each interval that starts at or after the last kept end, and count the rest as removals.
date: 2022-06-04
draft: false
slug: /dsa/non-overlapping-intervals
tags:
  - Competitive Programming
  - Intervals
  - Greedy
---

This is the classic activity-selection problem. Sort by end time. Keep an interval whenever its start is at or after the last kept interval's end; otherwise it conflicts and is counted as a removal. Ending as early as possible leaves the most room for what follows.

## Description

Given an array of `intervals`, return the minimum number of intervals you must remove so that the rest are non-overlapping. Intervals that only touch at an endpoint (`[1,2]`, `[2,3]`) do not count as overlapping.

**Example**

```
Input:  intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1   (remove [1,3])

Input:  intervals = [[1,2],[1,2],[1,2]]
Output: 2
```

**Constraints**

- $1 \le \text{intervals.length} \le 10^5$
- $-5 \cdot 10^4 \le \text{start}_i < \text{end}_i \le 5 \cdot 10^4$

## Prerequisites

- Activity selection: sort by finish time, take the earliest-ending compatible interval.
- Counting removals = total minus the max kept.

## Approach 1: Sort by end, greedy keep

### Intuition

After sorting by end, iterate. Track `prev_end`, the end of the last kept interval (start at `-inf`). If `start >= prev_end`, keep this interval and update `prev_end`. Otherwise it overlaps — increment the removal count.

### Algorithm

1. `intervals.sort(key=lambda x: x[1])`.
2. `prev_end = -inf`, `removed = 0`.
3. For each `s, e`: if `s >= prev_end`, `prev_end = e`; else `removed += 1`.
4. Return `removed`.

```python
def eraseOverlapIntervals(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[1])
    prev_end = float("-inf")
    removed = 0
    for s, e in intervals:
        if s >= prev_end:
            prev_end = e
        else:
            removed += 1
    return removed
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(1)$ extra.

## Approach 2: Sort by start, keep the shorter on conflict

### Intuition

Sort by start. On an overlap between the current interval and the last kept one, discard whichever ends later (keep the earlier end to minimise future conflicts).

### Algorithm

1. `intervals.sort()`.
2. `prev_end = intervals[0][1]`, `removed = 0`.
3. For each `s, e` in `intervals[1:]`: if `s < prev_end` (overlap): `removed += 1`; `prev_end = min(prev_end, e)`. Else `prev_end = e`.
4. Return `removed`.

```python
def eraseOverlapIntervals(intervals: list[list[int]]) -> int:
    intervals.sort()
    prev_end = intervals[0][1]
    removed = 0
    for s, e in intervals[1:]:
        if s < prev_end:
            removed += 1
            prev_end = min(prev_end, e)
        else:
            prev_end = e
    return removed
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(1)$ extra.

## Common Pitfalls

- **Sorting by start and keeping the current interval on every conflict.** You must keep the one that ends earlier; blindly advancing `prev_end` to the new `e` can force more removals later.
- **Treating touching intervals as overlapping.** `[1,2]` and `[2,3]` are fine — the keep test is `s >= prev_end`, not `s > prev_end`.
- **Initialising `prev_end` to 0.** Starts can be negative; use `-inf` (or seed from the first interval).
- **Returning the kept count.** The question asks for *removed* = `n - kept`.

## The keystone

"Earliest finish time first" is the provably optimal rule for packing the most non-overlapping intervals — the foundational greedy exchange argument. [Meeting Rooms](/citadel/dsa/meeting-rooms) is the yes/no version: can *zero* be removed.
