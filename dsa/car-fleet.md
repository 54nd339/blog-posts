---
title: Car Fleet - Sort by Position, Stack by Arrival Time
description: Counting how many clumps of cars reach the destination together — sorting cars front to back and comparing free-run arrival times on a stack settles it in one pass.
date: 2021-07-16
draft: false
slug: /dsa/car-fleet
tags:
  - Competitive Programming
  - Stack
  - Sorting
---

Stop thinking about positions over time and think about *arrival times*. Process cars from the one closest to the target backward: a car behind either catches the fleet ahead (same arrival time or sooner) and merges, or it never does and starts its own fleet.

## Description

`n` cars drive toward a `target` on a one-lane road. Car `i` starts at `position[i]` with speed `speed[i]`. A faster car catches a slower one and then travels at the slower car's speed — they become a fleet. Cars cannot pass. Return the number of fleets that arrive at the target. A car that catches another exactly at the target counts as merged.

**Example**

```
Input:  target = 12, position = [10,8,0,5,3], speed = [2,4,1,1,3]
Output: 3

Input:  target = 10, position = [3], speed = [3]
Output: 1
```

**Constraints**

- $1 \le n \le 10^5$
- $0 < \text{target} \le 10^6$; positions are distinct; $0 < \text{speed}[i] \le 10^6$

## Prerequisites

- Sorting by a key.
- Monotonic [stack](/citadel/dsa/stack) of arrival times.

## Approach 1: Stack

### Intuition

Sort cars from nearest the target to farthest. Each car's free-run arrival time is `(target - position) / speed`. Push arrival times; if a car's time is `<=` the stack top, it has caught the fleet ahead — pop it (it merges, contributing nothing). The stack height is the fleet count.

### Algorithm

1. Zip `(position, speed)` and sort descending by position.
2. For each car, `t = (target - position) / speed`.
3. If the stack is non-empty and `t <= stack[-1]`, skip (merges). Else push `t`.
4. Return `len(stack)`.

```python
def carFleet(target: int, position: list[int], speed: list[int]) -> int:
    cars = sorted(zip(position, speed), reverse=True)
    stack: list[float] = []
    for pos, spd in cars:
        t = (target - pos) / spd
        if stack and t <= stack[-1]:
            continue
        stack.append(t)
    return len(stack)
```

### Complexity

- **Time:** $O(n \log n)$ — the sort dominates; the pass is $O(n)$.
- **Space:** $O(n)$ for the sorted list and stack.

## Approach 2: Iteration

### Intuition

Same idea without materialising a stack: carry a single `slowest` arrival time for the fleet currently in front. A new car exceeding it starts a fresh fleet.

### Algorithm

1. Sort `(position, speed)` descending by position.
2. `fleets = 0`, `slowest = -1`.
3. For each car, `t = (target - position) / speed`. If `t > slowest`, `fleets += 1` and `slowest = t`.
4. Return `fleets`.

```python
def carFleet(target: int, position: list[int], speed: list[int]) -> int:
    cars = sorted(zip(position, speed), reverse=True)
    fleets = 0
    slowest = -1.0
    for pos, spd in cars:
        t = (target - pos) / spd
        if t > slowest:
            fleets += 1
            slowest = t
    return fleets
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$ for the sorted list ($O(1)$ extra beyond it).

## Common Pitfalls

- **Sort direction.** Front to back means *descending* position. A car behind (smaller position) merges when its arrival time does not exceed the lead's.
- **Comparing to the immediate predecessor's raw time instead of the fleet lead's.** Because a merged car adds nothing, the stack top (or `slowest`) always holds the current lead — the right thing to compare against.
- **The merge boundary is `<=`.** A car catching the fleet exactly at the target still merges.

## The keystone

Some motion problems have no clean simulation but do have an invariant once you sort by the right key and reduce each item to a scalar — here "arrival time if unobstructed", processed front to back. The stack keeps only the elements that survive as leaders, the same role it plays in [Daily Temperatures](/citadel/dsa/daily-temperatures).
