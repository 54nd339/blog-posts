---
title: Meeting Rooms II - The Peak Number of Concurrent Meetings
description: The fewest rooms to hold every meeting — a min-heap of end times reused as meetings start, or a chronological sweep of separate start and end arrays tracking the running overlap.
date: 2022-06-10
draft: false
slug: /dsa/meeting-rooms-ii
tags:
  - Competitive Programming
  - Intervals
  - Heap
---

The answer is the largest number of meetings active at the same instant. Process meetings by start time; keep a min-heap of the end times of meetings currently in rooms. If the earliest end is at or before the next start, that room frees up; otherwise allocate a new one. The heap's peak size is the answer.

## Description

Given meeting time `intervals` `[start, end]`, return the minimum number of conference rooms required.

**Example**

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: 2

Input:  intervals = [[7,10],[2,4]]
Output: 1
```

**Constraints**

- $1 \le \text{intervals.length} \le 10^4$
- $0 \le \text{start} < \text{end} \le 10^6$

## Prerequisites

- Min-heap keyed on end time.
- The equivalent "sorted starts vs sorted ends" two-pointer sweep.

## Approach 1: Min-heap of end times

### Intuition

Sort meetings by start. The heap holds end times of ongoing meetings. For each meeting, pop every end time `<= its start` (those rooms are free), then push its own end. Track the max heap size.

### Algorithm

1. `intervals.sort(key=lambda x: x[0])`. `heap = []`, `rooms = 0`.
2. For each `s, e`: while `heap` and `heap[0] <= s`, pop. Push `e`. `rooms = max(rooms, len(heap))`.
3. Return `rooms`.

```python
import heapq

def minMeetingRooms(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[0])
    heap = []
    rooms = 0
    for s, e in intervals:
        while heap and heap[0] <= s:
            heapq.heappop(heap)
        heapq.heappush(heap, e)
        rooms = max(rooms, len(heap))
    return rooms
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Approach 2: Chronological start/end sweep

### Intuition

Sort starts and ends separately. Advance a pointer through starts; each start needs a room, each end (that has already passed) frees one. The running room count's maximum is the answer.

### Algorithm

1. `starts = sorted(s for s, e in intervals)`, `ends = sorted(e for s, e in intervals)`.
2. `i = j = 0`, `cur = best = 0`.
3. While `i < n`: if `starts[i] < ends[j]`, `cur += 1`, `best = max(best, cur)`, `i += 1`; else `cur -= 1`, `j += 1`.
4. Return `best`.

```python
def minMeetingRooms(intervals: list[list[int]]) -> int:
    n = len(intervals)
    starts = sorted(s for s, _ in intervals)
    ends = sorted(e for _, e in intervals)
    i = j = cur = best = 0
    while i < n:
        if starts[i] < ends[j]:
            cur += 1
            best = max(best, cur)
            i += 1
        else:
            cur -= 1
            j += 1
    return best
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Approach 3: Difference array on time

### Intuition

Add `+1` at each start and `-1` at each end, then sweep time in order; the running sum's maximum is the concurrency peak.

### Algorithm

1. Build `events = [(s, 1) for ...] + [(e, -1) for ...]`; sort with `-1` before `+1` at equal times (a meeting ending frees the room for one starting then).
2. Sweep, tracking the running sum and its max.

```python
def minMeetingRooms(intervals: list[list[int]]) -> int:
    events = []
    for s, e in intervals:
        events.append((s, 1))
        events.append((e, -1))
    events.sort(key=lambda x: (x[0], x[1]))

    cur = best = 0
    for _, delta in events:
        cur += delta
        best = max(best, cur)
    return best
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Popping with `<` instead of `<=`.** A meeting starting exactly when another ends can reuse the room; pop end times `<= start`.
- **Tie order in the sweep.** When a start and an end coincide, process the end first (`-1` before `+1`), or you over-count rooms by one.
- **Sorting `(start, end)` pairs but only reading starts.** For the heap approach you need the pairs sorted by start; for the sweep you sort the two coordinate lists independently.
- **Returning the final `cur` instead of `best`.** Concurrency goes down at the end; the peak is what matters.

## The keystone

"Maximum number of overlapping intervals" is a sweep over `+1`/`-1` events, and the min-heap of end times is that sweep with the room pool made explicit. This is the counting core behind range-scheduling and CPU-task problems. Next, [Minimum Interval to Include Each Query](/citadel/dsa/minimum-interval-to-include-each-query) adds queries into the sweep.
