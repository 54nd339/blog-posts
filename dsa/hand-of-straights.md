---
title: Hand of Straights - Always Start a Run at the Smallest Card
description: Whether a hand splits into consecutive groups of a fixed size — repeatedly take the smallest remaining card as the start of a run and remove the next groupSize consecutive values.
date: 2022-05-27
draft: false
slug: /dsa/hand-of-straights
tags:
  - Competitive Programming
  - Greedy
  - Hash Table
---

The smallest card left must begin a run — nothing smaller exists to precede it. So peel runs from the bottom: take the minimum value `v`, then require and consume one each of `v, v+1, ..., v+groupSize-1`. If any is missing, it is impossible.

## Description

Given an integer array `hand` and an integer `groupSize`, return `true` if the cards can be rearranged into groups of `groupSize` consecutive cards.

**Example**

```
Input:  hand = [1,2,3,6,2,3,4,7,8], groupSize = 3
Output: true   ([1,2,3], [2,3,4], [6,7,8])

Input:  hand = [1,2,3,4,5], groupSize = 4
Output: false
```

**Constraints**

- $1 \le \text{hand.length} \le 10^4$
- $0 \le \text{hand}[i] \le 10^9$

## Prerequisites

- Counting multiplicities with a hash map.
- A min-heap or sorted keys to always reach the smallest remaining value.

## Approach 1: Counter + min-heap

### Intuition

Count each value. Push the distinct values into a min-heap. Pop the smallest `v`; it needs `count[v]` runs starting at `v`, so decrement `count[v..v+groupSize-1]` by `count[v]`. Any shortfall means failure.

### Algorithm

1. If `len(hand) % groupSize != 0`, return `False`.
2. `count = Counter(hand)`; heapify its keys.
3. While the heap is non-empty: peek the smallest `v`; `need = count[v]`. For `x` from `v` to `v + groupSize - 1`: if `count[x] < need`, return `False`; `count[x] -= need`; if `count[x] == 0` and `x` is the heap min, pop it (only valid to pop when `x == v` at the front).
4. Return `True`.

```python
import heapq
from collections import Counter

def isNStraightHand(hand: list[int], groupSize: int) -> bool:
    if len(hand) % groupSize:
        return False
    count = Counter(hand)
    heap = list(count)
    heapq.heapify(heap)

    while heap:
        v = heap[0]
        need = count[v]
        for x in range(v, v + groupSize):
            if count[x] < need:
                return False
            count[x] -= need
        while heap and count[heap[0]] == 0:
            heapq.heappop(heap)
    return True
```

### Complexity

- **Time:** $O(n \log n + n \cdot \text{groupSize})$.
- **Space:** $O(n)$.

## Approach 2: Counter + sorted keys

### Intuition

Same greedy, but instead of a heap, iterate the sorted distinct values; when a value still has a positive count, it must be a run start.

### Algorithm

1. Divisibility check. `count = Counter(hand)`.
2. For `v` in `sorted(count)`: `start = count[v]`; if `start > 0`, for `x` from `v` to `v + groupSize - 1`: if `count[x] < start`, return `False`; `count[x] -= start`.
3. Return `True`.

```python
from collections import Counter

def isNStraightHand(hand: list[int], groupSize: int) -> bool:
    if len(hand) % groupSize:
        return False
    count = Counter(hand)
    for v in sorted(count):
        start = count[v]
        if start > 0:
            for x in range(v, v + groupSize):
                if count[x] < start:
                    return False
                count[x] -= start
    return True
```

### Complexity

- **Time:** $O(n \log n + n \cdot \text{groupSize})$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Skipping the divisibility check.** If `len(hand)` is not a multiple of `groupSize`, stop immediately.
- **Not scaling by `count[v]`.** If the smallest value appears 3 times, you must open 3 runs at once and consume 3 of each subsequent value — decrement by `count[v]`, not by 1.
- **Re-processing a value whose count hit 0.** In the sorted-keys version, guard with `if start > 0`; a value fully consumed by earlier runs is not a new start.
- **Assuming `groupSize == 1` needs special handling.** It trivially succeeds; the loop bodies still work.

## The keystone

"The smallest unmatched element forces the next move" is a classic greedy trigger — there is no freedom about where the lowest card goes. The same "peel from the minimum" logic solves the divide-array-into-consecutive-subsequences family.
