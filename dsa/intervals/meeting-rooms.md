---
title: Meeting Rooms - Any Overlap at All
description: Whether a person can attend every meeting — sort the intervals by start and check that each meeting begins no earlier than the previous one ends.
date: 2022-06-10
draft: false
slug: /dsa/meeting-rooms
tags:
  - Competitive Programming
  - Intervals
  - Sorting
---

If two meetings overlap, one person cannot attend both. Sort by start time; then the only possible conflict for meeting `i` is with meeting `i - 1`. One linear check after the sort settles it.

## Description

Given an array of meeting time `intervals` `[start, end]`, return `true` if a person could attend all meetings (no two overlap). Meetings that touch at an endpoint do not conflict.

**Example**

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: false

Input:  intervals = [[7,10],[2,4]]
Output: true
```

**Constraints**

- $0 \le \text{intervals.length} \le 10^4$
- $0 \le \text{start} \le \text{end} \le 10^6$

## Prerequisites

- Sorting intervals by start.
- After a start-sort, adjacent intervals are the only overlap candidates.

## Approach 1: Sort by start, check neighbours

### Intuition

Sort. Walk from the second interval; if any `start` is strictly less than the previous `end`, there is an overlap.

### Algorithm

1. `intervals.sort(key=lambda x: x[0])`.
2. For `i` from `1` to `n - 1`: if `intervals[i][0] < intervals[i - 1][1]`, return `False`.
3. Return `True`.

```python
def canAttendMeetings(intervals: list[list[int]]) -> bool:
    intervals.sort(key=lambda x: x[0])
    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i - 1][1]:
            return False
    return True
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(1)$ extra.

## Approach 2: Separate sorted start and end arrays

### Intuition

Sort all starts and all ends independently. If the `i`-th start (for `i >= 1`) comes before the `(i-1)`-th end, some meeting began before another finished.

### Algorithm

1. `starts = sorted(s for s, e in intervals)`, `ends = sorted(e for s, e in intervals)`.
2. For `i` from `1` to `n - 1`: if `starts[i] < ends[i - 1]`, return `False`.
3. Return `True`.

```python
def canAttendMeetings(intervals: list[list[int]]) -> bool:
    starts = sorted(s for s, _ in intervals)
    ends = sorted(e for _, e in intervals)
    for i in range(1, len(intervals)):
        if starts[i] < ends[i - 1]:
            return False
    return True
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`<=` instead of `<`.** Touching meetings `[1,2]` and `[2,3]` are allowed; only `start < prev_end` is a conflict.
- **Not sorting.** Without the start-sort, a conflict can be between non-adjacent intervals and the single-pass check misses it.
- **Comparing to `intervals[0][1]` for all `i`.** Compare each meeting to its immediate predecessor, not always the first.
- **Empty input.** Zero meetings trivially returns `True`.

## The keystone

Sorting by start reduces "does any pair overlap" to "does any adjacent pair overlap" — one $O(n)$ sweep. [Meeting Rooms II](/citadel/dsa/meeting-rooms-ii) asks the quantitative version: the maximum number of meetings running at once.
