---
title: Insert Interval - Three Phases in One Pass
description: Adding one interval to a sorted non-overlapping list — copy intervals ending before the new one, absorb every interval that overlaps it into a widening block, then copy the rest.
date: 2022-05-30
draft: false
slug: /dsa/insert-interval
tags:
  - Competitive Programming
  - Intervals
  - Arrays
---

The list is already sorted and disjoint, so a single left-to-right pass has three phases: intervals strictly before the new one (copy as-is), intervals that touch it (merge into one growing interval), and intervals strictly after (copy as-is).

## Description

Given a list of non-overlapping `intervals` sorted by start, and a `newInterval`, insert it so the result stays sorted and non-overlapping (merging as needed). Return the new list.

**Example**

```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]

Input:  intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]
```

**Constraints**

- $0 \le \text{intervals.length} \le 10^4$
- `intervals` is sorted by start; $0 \le \text{start} \le \text{end} \le 10^5$.

## Prerequisites

- Overlap test: `a.start <= b.end and b.start <= a.end`.
- Merging by taking `min` of starts and `max` of ends.

## Approach 1: Linear three-phase scan

### Intuition

Walk the intervals. While an interval ends before `newInterval` starts, append it. While an interval starts at or before `newInterval` ends, expand `newInterval` to cover it. Append `newInterval`. Append the remaining intervals.

### Algorithm

1. `res = []`, `i = 0`, `n = len(intervals)`.
2. While `i < n` and `intervals[i][1] < newInterval[0]`: append `intervals[i]`; `i += 1`.
3. While `i < n` and `intervals[i][0] <= newInterval[1]`: `newInterval = [min(newInterval[0], intervals[i][0]), max(newInterval[1], intervals[i][1])]`; `i += 1`.
4. Append `newInterval`. Append the rest `intervals[i:]`.
5. Return `res`.

```python
def insert(intervals: list[list[int]], newInterval: list[int]) -> list[list[int]]:
    res = []
    i, n = 0, len(intervals)

    while i < n and intervals[i][1] < newInterval[0]:
        res.append(intervals[i])
        i += 1

    while i < n and intervals[i][0] <= newInterval[1]:
        newInterval = [min(newInterval[0], intervals[i][0]),
                       max(newInterval[1], intervals[i][1])]
        i += 1
    res.append(newInterval)

    while i < n:
        res.append(intervals[i])
        i += 1

    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the output.

## Approach 2: Append everything, then merge

### Intuition

Add `newInterval` to the list, sort by start, and run the standard [Merge Intervals](/citadel/dsa/merge-intervals) sweep. Simpler to remember, one `log` factor slower.

### Algorithm

1. `arr = sorted(intervals + [newInterval])`.
2. `res = [arr[0]]`; for each `s, e` in `arr[1:]`: if `s <= res[-1][1]`, `res[-1][1] = max(res[-1][1], e)`; else append `[s, e]`.
3. Return `res`.

```python
def insert(intervals: list[list[int]], newInterval: list[int]) -> list[list[int]]:
    arr = sorted(intervals + [newInterval])
    res = [arr[0][:]]
    for s, e in arr[1:]:
        if s <= res[-1][1]:
            res[-1][1] = max(res[-1][1], e)
        else:
            res.append([s, e])
    return res
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`<` vs `<=` in the overlap phase.** Touching intervals like `[1,3]` and `[3,5]` merge here, so the second `while` uses `intervals[i][0] <= newInterval[1]`.
- **Reassigning `newInterval` vs mutating it.** Build a fresh `[min, max]`; mutating a shared reference can corrupt the input list.
- **Forgetting the tail.** After the merge phase, the remaining intervals must still be copied.
- **Empty input.** With `intervals == []`, phases 1 and 3 do nothing and you return `[newInterval]`.

## The keystone

A pre-sorted disjoint list turns insertion into a three-phase linear scan: skip, merge, skip. Once the list might not be sorted or disjoint, you fall back to the sort-then-sweep of [Merge Intervals](/citadel/dsa/merge-intervals).
