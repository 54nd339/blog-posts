---
title: Merge Intervals - Sort by Start, Then Sweep
description: Merging all overlapping intervals in a list — sort by start so overlaps are adjacent, then extend the last kept interval when the next one starts within it, otherwise open a new one.
date: 2022-06-04
draft: false
slug: /dsa/merge-intervals
tags:
  - Competitive Programming
  - Intervals
  - Sorting
---

Once intervals are sorted by start, any interval that overlaps an earlier one must overlap the *most recent* kept interval. So keep a result list; for each interval, either stretch the last entry's end or append a new entry.

## Description

Given an array of `intervals` where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals and return the non-overlapping intervals that cover all the input ranges.

**Example**

```
Input:  intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input:  intervals = [[1,4],[4,5]]
Output: [[1,5]]
```

**Constraints**

- $1 \le \text{intervals.length} \le 10^4$
- $0 \le \text{start}_i \le \text{end}_i \le 10^4$

## Prerequisites

- Sorting by start.
- The invariant that, post-sort, only the last kept interval can be extended.

## Approach 1: Sort then sweep

### Intuition

Sort by start. Initialise the result with the first interval. For each subsequent `[s, e]`: if `s <= res[-1][1]`, the two overlap (or touch) — set `res[-1][1] = max(res[-1][1], e)`. Otherwise append `[s, e]`.

### Algorithm

1. `intervals.sort(key=lambda x: x[0])`.
2. `res = [intervals[0][:]]`.
3. For each `s, e` in `intervals[1:]`: if `s <= res[-1][1]`, `res[-1][1] = max(res[-1][1], e)`; else `res.append([s, e])`.
4. Return `res`.

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    intervals.sort(key=lambda x: x[0])
    res = [intervals[0][:]]
    for s, e in intervals[1:]:
        if s <= res[-1][1]:
            res[-1][1] = max(res[-1][1], e)
        else:
            res.append([s, e])
    return res
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$ (or $O(\log n)$ extra if sorting in place and writing to the input).

## Approach 2: Sweep line over endpoints

### Intuition

Turn each interval into a `+1` event at its start and a `-1` event at its end. Sort events; a running counter rising from 0 opens a merged block, falling back to 0 closes it.

### Algorithm

1. Build events `(start, +1)` and `(end, -1)`; sort, breaking ties so `+1` at a value comes before `-1` (to merge touching intervals).
2. Sweep: `count += delta`; when `count` goes 0→positive, record a block start; when it returns to 0, record the block end.

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    events = []
    for s, e in intervals:
        events.append((s, 0))   # 0 sorts before 1, so starts precede ends at a tie
        events.append((e, 1))
    events.sort()

    res = []
    count = 0
    start = None
    for value, typ in events:
        if typ == 0:
            if count == 0:
                start = value
            count += 1
        else:
            count -= 1
            if count == 0:
                res.append([start, value])
    return res
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Sorting by end instead of start.** The "only the last interval can be extended" invariant depends on a start-sort.
- **`s < res[-1][1]` instead of `<=`.** `[1,4]` and `[4,5]` touch and should merge; use `<=`.
- **`res[-1][1] = e` unconditionally.** The new interval might be fully contained (`[1,10]` then `[2,3]`); take the `max`.
- **Mutating `intervals[0]` into the result.** Copy it, or a later `res[-1][1] = ...` edits the caller's data.

## The keystone

"Sort by start, then a single sweep where each element either extends the current block or starts a new one" is the interval-merging template. [Insert Interval](/citadel/dsa/insert-interval), [Partition Labels](/citadel/dsa/partition-labels), and skyline-style problems are all this sweep.
