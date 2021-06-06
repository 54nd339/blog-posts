---
title: Container With Most Water - Always Move the Shorter Wall
description: Choosing two lines that hold the most water — the quadratic check of all pairs collapses to one linear sweep because moving the taller of two pointers can never improve the area.
date: 2021-06-06
draft: false
slug: /dsa/container-with-most-water
tags:
  - Competitive Programming
  - Two Pointers
  - Greedy
---

The volume between two lines is `min(height[i], height[j]) * (j - i)` — the shorter wall sets the depth, the gap sets the width. Checking every pair is $O(n^2)$; the linear solution rests on one observation about which pointer to move.

## Description

Given `height`, where `height[i]` is the height of a vertical line at position `i`, pick two lines so the container they form with the x-axis holds the most water. Return that maximum area.

**Example**

```
Input:  height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
Output: 49          # lines at index 1 and 8: min(8, 7) * (8 - 1)

Input:  height = [1, 1]
Output: 1
```

**Constraints**

- $2 \le \text{height.length} \le 10^5$
- $0 \le \text{height}[i] \le 10^4$

## Prerequisites

- [Two pointers](/citadel/dsa/two-pointers) converging from the ends.
- The greedy argument that a discarded endpoint pairs worse with everything still in range.

## Approach 1: Brute Force

### Intuition

Try all `i < j`, compute the area, keep the largest.

### Algorithm

1. Initialise `best = 0`.
2. For every pair `i < j`, compute `min(height[i], height[j]) * (j - i)` and update `best`.
3. Return `best`.

```python
def maxArea(height: list[int]) -> int:
    best = 0
    n = len(height)
    for i in range(n):
        for j in range(i + 1, n):
            best = max(best, min(height[i], height[j]) * (j - i))
    return best
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(1)$.

## Approach 2: Two Pointers

### Intuition

Start with the widest container: `lo = 0`, `hi = n - 1`. Its area is limited by the shorter wall. Moving the taller wall inward cannot raise the effective height and loses width — strictly worse. So always move the shorter wall, hoping for a taller one.

### Algorithm

1. `lo = 0`, `hi = n - 1`, `best = 0`.
2. While `lo < hi`:
   - Compute `area = min(height[lo], height[hi]) * (hi - lo)`; update `best`.
   - If `height[lo] < height[hi]`, `lo += 1`; else `hi -= 1`.
3. Return `best`.

```python
def maxArea(height: list[int]) -> int:
    lo, hi = 0, len(height) - 1
    best = 0
    while lo < hi:
        area = min(height[lo], height[hi]) * (hi - lo)
        best = max(best, area)
        if height[lo] < height[hi]:
            lo += 1
        else:
            hi -= 1
    return best
```

### Complexity

- **Time:** $O(n)$ — one pointer advances each step; they meet after `n - 1` steps.
- **Space:** $O(1)$.

## Common Pitfalls

- **The width is `hi - lo`,** the number of unit gaps between the lines — not `hi - lo + 1` and not the count of lines between them.
- **Moving the taller pointer, or both.** When `height[lo] <= height[hi]`, every container using `lo` with some `j < hi` has smaller width and height at most `height[lo]`, so its area cannot beat the one just recorded — discarding `lo` is safe. Ties can go either way.
- **Recording the area after moving a pointer.** Compute and update `best` first, then move.

## The keystone

When the objective is `width * min(two endpoints)` and both start at the extremes, the greedy rule "sacrifice the limiting endpoint" turns an all-pairs search into a linear sweep — because the discarded endpoint provably pairs worse with everything still in range. That reasoning also drives [Two Sum II](/citadel/dsa/two-sum-ii) and [Trapping Rain Water](/citadel/dsa/trapping-rain-water).
