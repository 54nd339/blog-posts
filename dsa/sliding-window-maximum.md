---
title: Sliding Window Maximum - A Deque That Stays Sorted
description: The maximum of every length-k window — rescanning is O(nk) and a heap gives O(n log n), but a monotonic deque of indices keeps the current max at its front in amortised O(n).
date: 2021-06-28
draft: false
slug: /dsa/sliding-window-maximum
tags:
  - Competitive Programming
  - Sliding Window
  - Monotonic Deque
---

The linear solution keeps a deque of *indices* whose values are strictly decreasing, so the front is always the window's maximum and stale indices fall off both ends.

## Description

Given `nums` and a window size `k`, return an array of the maximum of each contiguous window as it slides left to right.

**Example**

```
Input:  nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]

Input:  nums = [1], k = 1
Output: [1]
```

**Constraints**

- $1 \le \text{nums.length} \le 10^5$
- $-10^4 \le \text{nums}[i] \le 10^4$
- $1 \le k \le \text{nums.length}$

## Prerequisites

- [Sliding window](/citadel/dsa/sliding-window), fixed size.
- Monotonic deque; binary heaps with lazy deletion; (optionally) segment trees for range-max.

## Approach 1: Brute Force

### Intuition

For each window position, scan its `k` elements for the max.

### Algorithm

1. For `i` from `0` to `n - k`: append `max(nums[i:i+k])` to the output.

```python
def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    return [max(nums[i:i + k]) for i in range(len(nums) - k + 1)]
```

### Complexity

- **Time:** $O(n \cdot k)$.
- **Space:** $O(1)$ beyond the output.

## Approach 2: Segment Tree

### Intuition

A segment tree answers range-maximum queries in $O(\log n)$. Build it once, then query `[i, i + k - 1]` for every window.

### Algorithm

1. Build a max segment tree over `nums`.
2. For each window start `i`, query the range `[i, i + k - 1]` and append the result.

```python
def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    n = len(nums)
    size = 1
    while size < n:
        size *= 2
    tree = [float("-inf")] * (2 * size)
    for i, x in enumerate(nums):
        tree[size + i] = x
    for i in range(size - 1, 0, -1):
        tree[i] = max(tree[2 * i], tree[2 * i + 1])

    def query(lo: int, hi: int) -> int:      # inclusive
        lo += size
        hi += size + 1
        res = float("-inf")
        while lo < hi:
            if lo & 1:
                res = max(res, tree[lo]); lo += 1
            if hi & 1:
                hi -= 1; res = max(res, tree[hi])
            lo >>= 1
            hi >>= 1
        return res

    return [query(i, i + k - 1) for i in range(n - k + 1)]
```

### Complexity

- **Time:** $O(n \log n)$ — an $O(\log n)$ query per window.
- **Space:** $O(n)$ for the tree. Overkill here, but the right tool when the array also gets point updates.

## Approach 3: Heap

### Intuition

Push `(value, index)` into a max-heap as the window advances; before reading the max, discard any top whose index has left the window.

### Algorithm

1. For each `i`: push `(-nums[i], i)`.
2. Once `i >= k - 1`: while the heap top's index is `<= i - k`, pop it; then record `-heap[0][0]`.

```python
import heapq

def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    heap: list[tuple[int, int]] = []
    out = []
    for i, x in enumerate(nums):
        heapq.heappush(heap, (-x, i))
        if i >= k - 1:
            while heap[0][1] <= i - k:
                heapq.heappop(heap)
            out.append(-heap[0][0])
    return out
```

### Complexity

- **Time:** $O(n \log n)$ — each element is pushed once and popped at most once.
- **Space:** $O(n)$ for the heap.

## Approach 4: Dynamic Programming (block maxima)

### Intuition

Split the array into blocks of size `k`. `leftMax[i]` is the running max from the start of `i`'s block; `rightMax[i]` from the end of `i`'s block backward. Any window `[i, i + k - 1]` spans two blocks, and its max is `max(rightMax[i], leftMax[i + k - 1])`.

### Algorithm

1. Build `leftMax`: reset at each index divisible by `k`, otherwise carry the running max.
2. Build `rightMax`: reset at each block boundary from the right.
3. For each window start `i`, output `max(rightMax[i], leftMax[i + k - 1])`.

```python
def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    n = len(nums)
    left = [0] * n
    right = [0] * n
    for i in range(n):
        left[i] = nums[i] if i % k == 0 else max(left[i - 1], nums[i])
    for i in range(n - 1, -1, -1):
        right[i] = nums[i] if i % k == k - 1 or i == n - 1 else max(right[i + 1], nums[i])
    return [max(right[i], left[i + k - 1]) for i in range(n - k + 1)]
```

### Complexity

- **Time:** $O(n)$ — three linear passes.
- **Space:** $O(n)$ for the two arrays.

## Approach 5: Deque

### Intuition

Maintain a deque of indices, front to back, whose `nums` values are strictly decreasing. A new index evicts every smaller value from the back (they can never be the max again); the front expires when it slides out of the window; the front is the current maximum.

### Algorithm

1. For each index `i`, value `x`:
   - While the deque is non-empty and `nums[deque[-1]] <= x`, pop the back.
   - Append `i`.
   - If `deque[0] <= i - k`, pop the front.
   - If `i >= k - 1`, append `nums[deque[0]]` to the output.

```python
from collections import deque

def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    dq = deque()
    out = []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            out.append(nums[dq[0]])
    return out
```

### Complexity

- **Time:** $O(n)$ amortised — each index enters and leaves the deque once.
- **Space:** $O(k)$ — the deque holds at most one window of indices.

## Common Pitfalls

- **Storing values instead of indices (heap and deque).** You need the index to test whether the front/top has expired.
- **Front-expiry test.** It uses `i - k`: an index that far back is exactly `k` positions before `i`, so outside a size-`k` window. Apply it *after* appending `i`.
- **Off-by-one in the output range.** There are `n - k + 1` windows.

## The keystone

A monotonic deque answers "extreme of a sliding window" in amortised $O(1)$ per step because it discards any element dominated by a newer, better one. The same structure computes "next greater element" and powers [Daily Temperatures](/citadel/dsa/daily-temperatures) and [Largest Rectangle in Histogram](/citadel/dsa/largest-rectangle-in-histogram).
