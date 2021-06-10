---
title: Trapping Rain Water - Water Is Set by the Smaller Wall
description: How much rain an elevation map traps — each position holds min of the tallest bar to its left and right minus its own height, computed by prefix arrays, a stack, or two pointers in O(1) space.
date: 2021-06-10
draft: false
slug: /dsa/trapping-rain-water
tags:
  - Competitive Programming
  - Two Pointers
  - Dynamic Programming
---

The water sitting on top of bar `i` rises until it would spill over the lower of the two tallest walls on either side: `water[i] = min(maxLeft[i], maxRight[i]) - height[i]`. Everything after that is about computing those two maxima cheaply.

## Description

Given `height`, an elevation map of unit-width bars, compute how much water is trapped after rain.

**Example**

```
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Input:  height = [4,2,0,3,2,5]
Output: 9
```

**Constraints**

- $1 \le \text{height.length} \le 2 \times 10^4$
- $0 \le \text{height}[i] \le 10^5$

## Prerequisites

- Prefix / suffix maxima.
- Monotonic [stack](/citadel/dsa/stack) (Approach 3).
- [Two pointers](/citadel/dsa/two-pointers) advancing the side with the smaller running max (Approach 4).

## Approach 1: Brute Force

### Intuition

For each bar, the water above it is bounded by the tallest bar to its left and the tallest to its right (inclusive of itself). Scan outward both ways.

### Algorithm

1. For each index `i`: scan `0 .. i` for `leftMax`, scan `i .. n-1` for `rightMax`.
2. Add `min(leftMax, rightMax) - height[i]` to the total.

```python
def trap(height: list[int]) -> int:
    n = len(height)
    total = 0
    for i in range(n):
        left = max(height[:i + 1])
        right = max(height[i:])
        total += min(left, right) - height[i]
    return total
```

### Complexity

- **Time:** $O(n^2)$ — two scans per position.
- **Space:** $O(1)$ (using explicit loops instead of slices).

## Approach 2: Prefix & Suffix Arrays

### Intuition

The two scans repeat work. Precompute `leftMax[i]` and `rightMax[i]` once each, then every position is $O(1)$.

### Algorithm

1. Build `leftMax` left to right: `leftMax[i] = max(leftMax[i-1], height[i])`.
2. Build `rightMax` right to left similarly.
3. Sum `min(leftMax[i], rightMax[i]) - height[i]` over all `i`.

```python
def trap(height: list[int]) -> int:
    n = len(height)
    if n == 0:
        return 0
    left_max = [0] * n
    right_max = [0] * n
    left_max[0] = height[0]
    for i in range(1, n):
        left_max[i] = max(left_max[i - 1], height[i])
    right_max[n - 1] = height[n - 1]
    for i in range(n - 2, -1, -1):
        right_max[i] = max(right_max[i + 1], height[i])
    return sum(min(left_max[i], right_max[i]) - height[i] for i in range(n))
```

### Complexity

- **Time:** $O(n)$ — three linear passes.
- **Space:** $O(n)$ for the two arrays.

## Approach 3: Stack

### Intuition

Keep a stack of indices with decreasing heights. When a taller bar arrives, it closes off a basin: the popped bar is the floor, the new stack top is the left wall, the current bar is the right wall.

### Algorithm

1. For each index `i`:
   - While the stack is non-empty and `height[i] > height[stack[-1]]`:
     - Pop `mid` (the basin floor).
     - If the stack is now empty, break.
     - `width = i - stack[-1] - 1`; `bounded = min(height[i], height[stack[-1]]) - height[mid]`.
     - Add `width * bounded` to the total.
   - Push `i`.

```python
def trap(height: list[int]) -> int:
    stack: list[int] = []
    total = 0
    for i, h in enumerate(height):
        while stack and h > height[stack[-1]]:
            mid = stack.pop()
            if not stack:
                break
            width = i - stack[-1] - 1
            bounded = min(h, height[stack[-1]]) - height[mid]
            total += width * bounded
        stack.append(i)
    return total
```

### Complexity

- **Time:** $O(n)$ — each index is pushed and popped once.
- **Space:** $O(n)$ for the stack.

## Approach 4: Two Pointers

### Intuition

Keep `lo`, `hi` at the ends with running maxima `left_max`, `right_max`. If `left_max <= right_max`, the water at `lo` is decided entirely by `left_max` (a taller-or-equal wall is guaranteed on the right), so process `lo` and step it in. Otherwise mirror with `hi`.

### Algorithm

1. `lo = 0`, `hi = n - 1`, `left_max = right_max = 0`, `total = 0`.
2. While `lo < hi`:
   - If `height[lo] <= height[hi]`: `left_max = max(left_max, height[lo])`; add `left_max - height[lo]`; `lo += 1`.
   - Else: `right_max = max(right_max, height[hi])`; add `right_max - height[hi]`; `hi -= 1`.

```python
def trap(height: list[int]) -> int:
    lo, hi = 0, len(height) - 1
    left_max = right_max = 0
    total = 0
    while lo < hi:
        if height[lo] <= height[hi]:
            left_max = max(left_max, height[lo])
            total += left_max - height[lo]
            lo += 1
        else:
            right_max = max(right_max, height[hi])
            total += right_max - height[hi]
            hi -= 1
    return total
```

### Complexity

- **Time:** $O(n)$ — one pointer moves per step.
- **Space:** $O(1)$.

## Common Pitfalls

- **Non-inclusive maxima (Approach 1/2).** `leftMax` and `rightMax` must include `i`, so a local peak contributes exactly `0` and needs no negative clamp.
- **Seeding the recurrences with `0` (Approach 2).** `left_max[0]` must be `height[0]`, `right_max[n-1]` must be `height[n-1]`.
- **Updating the running max after adding to the total (Approach 4).** Update first, so a new local peak adds `0`, not a negative amount.
- **Branching on `left_max` vs `right_max` instead of `height[lo]` vs `height[hi]` (Approach 4).** Both work but must be consistent; `height[lo] <= height[hi]` directly proves a tall-enough right wall exists.

## The keystone

The water over a cell is a `min` of two directional maxima. Prefix/suffix arrays compute it in $O(n)$ space; two pointers get it to $O(1)$ by always working from the side whose bounding wall is already the smaller. The stack view — pair each bar with the previous taller bar — is the bridge to [Largest Rectangle in Histogram](/citadel/dsa/largest-rectangle-in-histogram).
