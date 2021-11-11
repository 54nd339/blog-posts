---
title: Last Stone Weight - Always Smash the Two Heaviest
description: Repeatedly destroying the two largest stones and returning the difference — a max-heap serves up the two heaviest in O(log n) each round.
date: 2021-11-11
draft: false
slug: /dsa/last-stone-weight
tags:
  - Competitive Programming
  - Heap
  - Sorting
---

Each round needs the two current maxima, and it produces at most one new value. A max-heap gives you the maxima in $O(\log n)$ and reinserts the result in $O(\log n)$ — the natural fit.

## Description

You have stones with positive integer weights. Each turn, take the two heaviest stones `x <= y` and smash them: if `x == y` both are destroyed; otherwise the heavier becomes `y - x`. Return the weight of the last remaining stone, or `0` if none remain.

**Example**

```
Input:  stones = [2,7,4,1,8,1]
Output: 1

Input:  stones = [1]
Output: 1
```

**Constraints**

- $1 \le \text{stones.length} \le 30$
- $1 \le \text{stones}[i] \le 1000$

## Prerequisites

- Max-heap via `heapq` with negated values.
- Bucket/counting for a bounded value range (Approach 4).

## Approach 1: Sorting

### Intuition

Sort, pop the two largest off the end, push back the difference if non-zero, repeat.

### Algorithm

1. While `len(stones) > 1`: sort ascending; `y = stones.pop()`, `x = stones.pop()`; if `y > x`, append `y - x`.
2. Return `stones[0]` if any, else `0`.

```python
def lastStoneWeight(stones: list[int]) -> int:
    while len(stones) > 1:
        stones.sort()
        y = stones.pop()
        x = stones.pop()
        if y > x:
            stones.append(y - x)
    return stones[0] if stones else 0
```

### Complexity

- **Time:** $O(n^2 \log n)$ — a sort every round, up to `n` rounds.
- **Space:** $O(1)$ or $O(n)$ depending on the sort.

## Approach 2: Binary Search insertion

### Intuition

Keep the list sorted; instead of re-sorting each round, binary-search the correct slot for the new difference and insert it.

### Algorithm

1. Sort once.
2. While `len > 1`: pop the last two; if the difference `d > 0`, use `bisect` to insert `d` in order.
3. Return the remaining stone or `0`.

```python
from bisect import insort

def lastStoneWeight(stones: list[int]) -> int:
    stones.sort()
    while len(stones) > 1:
        y = stones.pop()
        x = stones.pop()
        if y > x:
            insort(stones, y - x)
    return stones[0] if stones else 0
```

### Complexity

- **Time:** $O(n^2)$ — each insertion shifts up to `n` elements.
- **Space:** $O(1)$ / $O(n)$.

## Approach 3: Heap

### Intuition

A max-heap (negate values, since `heapq` is a min-heap) hands you the two heaviest in $O(\log n)$ and reinserts the difference in $O(\log n)$.

### Algorithm

1. Build a max-heap of `-w` for each stone.
2. While more than one element: pop `y` and `x` (as positives). If `y != x`, push `-(y - x)`.
3. Return `-heap[0]` if non-empty, else `0`.

```python
import heapq

def lastStoneWeight(stones: list[int]) -> int:
    heap = [-w for w in stones]
    heapq.heapify(heap)
    while len(heap) > 1:
        y = -heapq.heappop(heap)
        x = -heapq.heappop(heap)
        if y != x:
            heapq.heappush(heap, -(y - x))
    return -heap[0] if heap else 0
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Approach 4: Bucket Sort

### Intuition

Weights are `<= 1000`, so count occurrences per weight. Walk from the heaviest weight down, carrying a single "pending" heavier stone; when you meet a stone, smash it against the pending one.

### Algorithm

1. `bucket[w]` = count of stones of weight `w`, for `w` in `1..1000`.
2. Scan `biggest` from `1000` down. Skip empty buckets.
3. If `bucket[biggest]` is even, all those stones pair off and cancel — clear the bucket.
4. If it is odd, one stone of weight `biggest` is left. Find the next non-empty lighter bucket `j`; if none, `biggest` is the last stone. Otherwise remove one stone of weight `j` and add one of weight `biggest - j` (if `biggest - j == j`, these cancel). Continue the scan.

```python
def lastStoneWeight(stones: list[int]) -> int:
    bucket = [0] * 1001
    for w in stones:
        bucket[w] += 1

    biggest = 1000
    while biggest > 0:
        if bucket[biggest] == 0:
            biggest -= 1
            continue
        if bucket[biggest] % 2 == 0:
            bucket[biggest] = 0
            continue
        bucket[biggest] = 0                 # one leftover of weight `biggest`
        j = biggest - 1
        while j > 0 and bucket[j] == 0:
            j -= 1
        if j == 0:
            return biggest
        bucket[j] -= 1
        bucket[biggest - j] += 1            # if biggest - j == j, net cancel
    return 0
```

### Complexity

- **Time:** $O(n + w)$ where `w` is the maximum weight.
- **Space:** $O(w)$.

## Common Pitfalls

- **`heapq` is a min-heap.** Negate on the way in and out for max-heap behaviour, and mind the sign when reading `heap[0]`.
- **Pushing a zero difference.** When `x == y` both stones vanish; only push when `y > x`.
- **Bucket-sort bookkeeping.** The carry between weights and the re-placement of `first - second` are fiddly; the heap version is the one to reach for unless the value range is tiny and `n` is large.

## The keystone

"Repeatedly take the extreme, transform, reinsert" is the canonical heap loop. Whenever a simulation always operates on the current maximum or minimum, a priority queue turns each round from $O(n)$ into $O(\log n)$.
