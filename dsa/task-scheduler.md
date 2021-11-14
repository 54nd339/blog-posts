---
title: Task Scheduler - The Most Frequent Task Sets the Pace
description: The minimum time to run tasks with a cooldown between identical ones — a max-heap simulates it, but a formula built from the highest task frequency gives the answer directly.
date: 2021-11-14
draft: false
slug: /dsa/task-scheduler
tags:
  - Competitive Programming
  - Heap
  - Greedy
---

The schedule length is dictated by the most frequent task. If task `A` occurs `maxf` times, it needs `maxf - 1` gaps of length `n` between its runs; other tasks (and idles) fill those gaps. Everything else is arithmetic.

## Description

Given an array `tasks` of CPU task labels and an integer `n`, where the same task must be separated by at least `n` intervals, return the minimum number of intervals (including idles) to finish all tasks.

**Example**

```
Input:  tasks = ["A","A","A","B","B","B"], n = 2
Output: 8          # A B idle A B idle A B

Input:  tasks = ["A","A","A","B","B","B"], n = 0
Output: 6
```

**Constraints**

- $1 \le \text{tasks.length} \le 10^4$
- Tasks are uppercase letters; $0 \le n \le 100$.

## Prerequisites

- Frequency counting.
- Max-heap + a cooldown queue (Approach 2); the "frame" counting argument (Approaches 3, 4).

## Approach 1: Brute Force (simulation)

### Intuition

Step through time. Each interval, run the available task with the highest remaining count that is not in cooldown; otherwise idle.

### Algorithm

1. Count task frequencies; track the last time each task ran.
2. For `t = 0, 1, 2, ...`: among tasks with remaining count and `t - last[task] > n`, pick the one with the highest remaining count; run it (decrement, set `last`). If none, idle.
3. Stop when all counts hit zero; return `t + 1`.

```python
from collections import Counter

def leastInterval(tasks: list[str], n: int) -> int:
    counts = Counter(tasks)
    last = {task: -n - 1 for task in counts}
    remaining = sum(counts.values())
    t = 0
    while remaining:
        ready = [task for task in counts
                 if counts[task] > 0 and t - last[task] > n]
        if ready:
            task = max(ready, key=lambda x: counts[x])
            counts[task] -= 1
            last[task] = t
            remaining -= 1
        t += 1
    return t
```

### Complexity

- **Time:** $O(\text{total time} \cdot 26)$.
- **Space:** $O(26)$.

## Approach 2: Max-Heap

### Intuition

Process in rounds of `n + 1` slots. Each round, pop up to `n + 1` most-frequent tasks from a max-heap, run each once, and stash those with remaining count in a cooldown list to be re-added after the round.

### Algorithm

1. Build a max-heap of frequencies. `time = 0`.
2. While the heap is non-empty: for `i` in `0 .. n`: if the heap is non-empty, pop `f`; if `f - 1 > 0`, add `(f - 1, i)` to a temp list. `time += 1` — but if the heap and temp list are both empty, stop early.
3. Push each `(f - 1)` from the temp list back onto the heap.
4. Return `time`.

```python
import heapq
from collections import Counter

def leastInterval(tasks: list[str], n: int) -> int:
    heap = [-c for c in Counter(tasks).values()]
    heapq.heapify(heap)
    time = 0
    while heap:
        temp = []
        for _ in range(n + 1):
            if heap:
                f = -heapq.heappop(heap)
                if f - 1 > 0:
                    temp.append(-(f - 1))
            time += 1
            if not heap and not temp:
                break
        for f in temp:
            heapq.heappush(heap, f)
    return time
```

### Complexity

- **Time:** $O(m)$ overall (`m = len(tasks)`); heap ops are on at most 26 items.
- **Space:** $O(1)$ — at most 26 distinct tasks.

## Approach 3: Greedy (count idle slots)

### Intuition

Lay out the most frequent task first: `A _ _ A _ _ A`. That skeleton has `maxf - 1` gaps of size `n`, i.e. `(maxf - 1) * n` empty slots. Every other task fills up to `maxf - 1` of them. Whatever idle remains is added to the task count.

### Algorithm

1. Sort the frequencies descending; `maxf = counts[0]` is the skeleton task.
2. `idle = (maxf - 1) * n`.
3. For every *other* count `c`, `idle -= min(maxf - 1, c)` (a tied task fills `maxf - 1` gaps).
4. Return `len(tasks) + max(0, idle)`.

```python
from collections import Counter

def leastInterval(tasks: list[str], n: int) -> int:
    counts = sorted(Counter(tasks).values(), reverse=True)
    maxf = counts[0]
    idle = (maxf - 1) * n
    for c in counts[1:]:
        idle -= min(maxf - 1, c)
    return len(tasks) + max(0, idle)
```

### Complexity

- **Time:** $O(m)$.
- **Space:** $O(1)$.

## Approach 4: Math (direct formula)

### Intuition

`(maxf - 1)` full frames of length `n + 1`, plus a final partial frame holding every task that shares the maximum frequency. If tasks are plentiful enough that no idling is needed, the answer is just `len(tasks)`.

### Algorithm

1. `maxf = max frequency`; `max_count = number of tasks with that frequency`.
2. `frame_time = (maxf - 1) * (n + 1) + max_count`.
3. Return `max(len(tasks), frame_time)`.

```python
from collections import Counter

def leastInterval(tasks: list[str], n: int) -> int:
    counts = Counter(tasks)
    maxf = max(counts.values())
    max_count = sum(1 for c in counts.values() if c == maxf)
    return max(len(tasks), (maxf - 1) * (n + 1) + max_count)
```

### Complexity

- **Time:** $O(m)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Ignoring the `max(len(tasks), ...)` clamp.** When there are many distinct tasks, the frames fill with real work and never idle; the answer is then simply the task count, which can exceed the frame formula.
- **Counting the trailing frame as full length `n + 1`.** The last frame only needs room for the tasks tied at `maxf`, hence `+ max_count`, not `+ (n + 1)`.
- **Greedy: subtracting more than `maxf - 1` from `idle` per task.** A task can fill at most one slot per gap, so its contribution is capped at `maxf - 1`.

## The keystone

When a scheduling constraint pins the timeline to one dominant quantity (here, the top frequency), you can often skip the simulation and compute the answer from that quantity plus corrections. The heap version is the fallback when the constraint is less regular.
