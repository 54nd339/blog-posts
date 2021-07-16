---
title: Largest Rectangle in Histogram - Every Bar Gets Its Widest Rectangle
description: The largest axis-aligned rectangle under a histogram — testing bar pairs is quadratic, but a monotonic stack computes each bar's left and right span in one linear pass.
date: 2021-07-16
draft: false
slug: /dsa/largest-rectangle-in-histogram
tags:
  - Competitive Programming
  - Stack
  - Monotonic Stack
---

Every maximal rectangle has some bar as its limiting height. For each bar `h`, the widest rectangle of height `h` extends left to the first shorter bar and right to the first shorter bar. A monotonic stack finds both boundaries for every bar in linear time.

## Description

Given `heights`, the bar heights of a histogram with unit-width bars, return the area of the largest rectangle that fits entirely under the bars.

**Example**

```
Input:  heights = [2,1,5,6,2,3]
Output: 10          # bars 5 and 6, width 2, height 5

Input:  heights = [2,4]
Output: 4
```

**Constraints**

- $1 \le \text{heights.length} \le 10^5$
- $0 \le \text{heights}[i] \le 10^4$

## Prerequisites

- Monotonic [stack](/citadel/dsa/stack) for "previous/next smaller element".
- Segment tree with a range-min index (Approach 2).

## Approach 1: Brute Force

### Intuition

Fix each bar as the limiting height; expand left and right while bars stay at least that tall.

### Algorithm

1. For each `i`: walk `left` down while `heights[left - 1] >= heights[i]`; walk `right` up while `heights[right + 1] >= heights[i]`.
2. Area is `heights[i] * (right - left + 1)`.

```python
def largestRectangleArea(heights: list[int]) -> int:
    n = len(heights)
    best = 0
    for i in range(n):
        left = i
        while left > 0 and heights[left - 1] >= heights[i]:
            left -= 1
        right = i
        while right < n - 1 and heights[right + 1] >= heights[i]:
            right += 1
        best = max(best, heights[i] * (right - left + 1))
    return best
```

### Complexity

- **Time:** $O(n^2)$ — a flat histogram makes every expansion span the whole array.
- **Space:** $O(1)$.

## Approach 2: Divide and Conquer (Segment Tree)

### Intuition

The shortest bar in a range must be the limiting height of *some* rectangle spanning the whole range. The best rectangle is the max of: that full-width one, the best in the left subrange, and the best in the right subrange. A segment tree finds the range-min index in $O(\log n)$.

### Algorithm

1. Build a segment tree over `heights` storing the index of the minimum in each range.
2. `solve(L, R)`: if `L > R`, return `0`; let `m` be the min-height index in `[L, R]`; return `max(heights[m] * (R - L + 1), solve(L, m - 1), solve(m + 1, R))`.

```python
def largestRectangleArea(heights: list[int]) -> int:
    n = len(heights)
    if n == 0:
        return 0
    INF = float("inf")
    size = 1
    while size < n:
        size *= 2
    tree = [(INF, -1)] * (2 * size)          # (height, index)
    for i, h in enumerate(heights):
        tree[size + i] = (h, i)
    for i in range(size - 1, 0, -1):
        tree[i] = min(tree[2 * i], tree[2 * i + 1])

    def min_idx(lo: int, hi: int) -> int:     # inclusive
        lo += size
        hi += size + 1
        best = (INF, -1)
        while lo < hi:
            if lo & 1:
                best = min(best, tree[lo]); lo += 1
            if hi & 1:
                hi -= 1; best = min(best, tree[hi])
            lo >>= 1
            hi >>= 1
        return best[1]

    def solve(l: int, r: int) -> int:
        if l > r:
            return 0
        m = min_idx(l, r)
        return max(heights[m] * (r - l + 1), solve(l, m - 1), solve(m + 1, r))

    return solve(0, n - 1)
```

### Complexity

- **Time:** $O(n \log n)$ average; $O(n^2)$ worst case (a sorted histogram makes the recursion one-sided).
- **Space:** $O(n)$ for the tree plus recursion.

## Approach 3: Stack (precompute boundaries)

### Intuition

Compute, for every bar, the index of the nearest shorter bar to its left and to its right, each with one monotonic-stack pass. Then every area is $O(1)$.

### Algorithm

1. `left[i]` = index of nearest bar to the left with strictly smaller height, or `-1`. Compute left to right, popping bars `>= heights[i]`.
2. `right[i]` = symmetric, or `n`.
3. Area for `i` is `heights[i] * (right[i] - left[i] - 1)`.

```python
def largestRectangleArea(heights: list[int]) -> int:
    n = len(heights)
    left = [-1] * n
    right = [n] * n
    stack = []
    for i in range(n):
        while stack and heights[stack[-1]] >= heights[i]:
            stack.pop()
        left[i] = stack[-1] if stack else -1
        stack.append(i)
    stack.clear()
    for i in range(n - 1, -1, -1):
        while stack and heights[stack[-1]] >= heights[i]:
            stack.pop()
        right[i] = stack[-1] if stack else n
        stack.append(i)
    return max((heights[i] * (right[i] - left[i] - 1) for i in range(n)), default=0)
```

### Complexity

- **Time:** $O(n)$ — two passes, each amortised linear.
- **Space:** $O(n)$ for the boundary arrays and stack.

## Approach 4: Stack (One Pass, start-index pairs)

### Intuition

Hold a stack of `(start_index, height)` in increasing height order. When a shorter bar arrives, pop each taller bar and close its rectangle, extending the current bar's start back to the earliest popped start.

### Algorithm

1. For each `(i, h)`: set `start = i`.
2. While the stack is non-empty and its top height `> h`: pop `(s, height)`; update `best` with `height * (i - s)`; set `start = s`.
3. Push `(start, h)`.
4. After the loop, for each remaining `(s, height)`: update `best` with `height * (n - s)`.

```python
def largestRectangleArea(heights: list[int]) -> int:
    n = len(heights)
    best = 0
    stack: list[tuple[int, int]] = []
    for i, h in enumerate(heights):
        start = i
        while stack and stack[-1][1] > h:
            s, height = stack.pop()
            best = max(best, height * (i - s))
            start = s
        stack.append((start, h))
    for s, height in stack:
        best = max(best, height * (n - s))
    return best
```

### Complexity

- **Time:** $O(n)$ — each bar pushed and popped once.
- **Space:** $O(n)$.

## Approach 5: Stack (Optimal, sentinel index)

### Intuition

Same as Approach 4 with an index stack and an appended sentinel. Iterating `i` from `0` to `n` (a virtual bar of height `0` at the end) flushes everything.

### Algorithm

1. For `i` in `0 .. n`, with `h = 0` when `i == n` else `heights[i]`:
   - While the stack is non-empty and `heights[stack[-1]] > h`: pop `mid`; `width = i` if the stack is now empty else `i - stack[-1] - 1`; update `best` with `heights[mid] * width`.
   - Push `i`.

```python
def largestRectangleArea(heights: list[int]) -> int:
    stack: list[int] = []
    best = 0
    for i in range(len(heights) + 1):
        h = 0 if i == len(heights) else heights[i]
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            best = max(best, height * width)
        stack.append(i)
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **The width after a pop (Approach 5).** Right boundary is `i`; left boundary is the *new* stack top, not the popped index minus one — every bar between them is at least `height`. So `width = i - stack[-1] - 1`, and `width = i` only when the stack emptied.
- **Forgetting the sentinel / final flush.** Without the height-`0` bar (Approach 5) or the post-loop pass (Approach 4), the final increasing run is never counted.
- **`>` vs `>=` when popping.** Use `>` so equal-height bars are handled by whichever is processed last; using `>=` still gives the right max but pops more eagerly.

## The keystone

"How far does my value dominate left and right?" is answered for the whole array in $O(n)$ by a monotonic stack — the pop that removes a bar is exactly when its far boundary is found. The same computation turns the maximal-rectangle-in-a-binary-matrix problem into one histogram per row, and it is the tool behind [Trapping Rain Water](/citadel/dsa/trapping-rain-water)'s stack solution.
