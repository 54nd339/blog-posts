---
title: Finding the Largest Zero Submatrix - Histogram Plus Stack
description: Reducing "biggest all-zero rectangle in a binary matrix" to n instances of largest-rectangle-in-histogram by treating each row as the floor of a bar chart of consecutive zeros above it.
date: 2024-04-29
draft: false
slug: /cp/largest-zero-submatrix
tags:
  - Competitive Programming
  - Dynamic Programming
  - Stack
---

Given a binary matrix, find the largest axis-aligned rectangle containing only zeros. The trick is to see each row as the base of a histogram whose bar heights are "how many zeros in a row, going straight up from here" — then the answer is the best over $n$ runs of the classic [largest rectangle in a histogram](/citadel/dsa/largest-rectangle-in-histogram).

## The problem

Input: an $n \times m$ grid of $0$s and $1$s. Output: the maximum area of a sub-rectangle whose every cell is $0$ (some variants ask for the largest *square*, or the rectangle itself).

Example:

```
0 0 1 0
0 0 0 0
1 0 0 0
```

The largest all-zero rectangle is the $2 \times 3$ block at rows $1$–$2$, columns $1$–$3$ — area $6$.

## The idea

Scan rows top to bottom, maintaining `height[c]` = the number of consecutive zeros ending at the current row in column $c$:

$$\text{height}[c] \mathrel{+}= 1 \text{ if cell is } 0, \qquad \text{height}[c] = 0 \text{ if cell is } 1.$$

After processing row $r$, `height` is a histogram. The largest all-zero rectangle whose **bottom edge lies on row $r$** is exactly the largest rectangle under that histogram. Take the max over all rows.

The histogram sub-problem — largest rectangle under bars of given heights — is solved in $O(m)$ with a monotonic stack: push bar indices while heights increase; when a shorter bar appears, pop and, for each popped bar, its rectangle extends from the bar after the new stack top to the current index.

## How it works

Row 0 `0 0 1 0` → `height = [1, 1, 0, 1]`; largest histogram rectangle area $1$.

Row 1 `0 0 0 0` → `height = [2, 2, 1, 2]`; largest rectangle: width $4$ at height $1$ → area $4$.

Row 2 `1 0 0 0` → `height = [0, 3, 2, 3]`; largest rectangle: bars $[3, 2, 3]$ give height $2$ over width $3$ → area $6$. That is the answer.

## Algorithm

```python
def largest_rectangle_in_histogram(h: list[int]) -> int:
    stack, best = [], 0                       # stack holds indices, heights increasing
    for i in range(len(h) + 1):
        cur = h[i] if i < len(h) else 0       # virtual 0 at the end flushes the stack
        while stack and h[stack[-1]] >= cur:
            top = stack.pop()
            left = stack[-1] + 1 if stack else 0
            best = max(best, h[top] * (i - left))
        stack.append(i)
    return best

def largest_zero_submatrix(grid: list[list[int]]) -> int:
    if not grid or not grid[0]:
        return 0
    m = len(grid[0])
    height = [0] * m
    best = 0
    for row in grid:
        for c in range(m):
            height[c] = height[c] + 1 if row[c] == 0 else 0
        best = max(best, largest_rectangle_in_histogram(height))
    return best
```

## Complexity

- **Time:** $O(nm)$ — each of the $n$ rows costs $O(m)$ to update `height` and $O(m)$ for the stack pass (every index pushed and popped once).
- **Space:** $O(m)$ for `height` and the stack.

## Variations

- **Largest all-zero square.** DP: `dp[r][c] = min(dp[r-1][c], dp[r][c-1], dp[r-1][c-1]) + 1` when the cell is $0$; answer is $(\max dp)^2$. $O(nm)$, no stack.
- **Return the rectangle, not just the area.** Track the row, the popped height, and the left/right bounds when `best` updates.
- **At most $k$ ones allowed.** No longer a clean histogram; needs a different approach (e.g. two-pointer on rows with a per-column count).
- **Maximal rectangle of 1s** (LeetCode 85) — identical, with the height rule inverted (`+1` on a `1`).

## Common pitfalls

- **`>=` vs `>` when popping.** Use `>=` so equal-height bars are handled by the later index; with `>` you can under-count width. Either is correct if the width formula matches, but be consistent.
- **Forgetting the sentinel.** Append a virtual height $0$ (loop to `len(h)` inclusive) so the stack fully drains and the last bars are measured.
- **Resetting `height` on a `1`.** A `1` breaks the vertical run — set `height[c] = 0`, do not just skip.
- **Width off by one.** After popping `top`, the rectangle spans `(left_boundary + 1 .. i - 1)`, width `i - left_boundary - 1`; with `left = stack[-1] + 1 if stack else 0` it is `i - left`.
- **Empty grid / empty row.** Guard `n == 0` and `m == 0`.

## The keystone

Turn each row into a histogram of "zeros stacked above this cell", and the largest all-zero rectangle with its bottom on that row is the largest rectangle under the histogram — one monotonic-stack pass. Sweeping all $n$ rows gives the global answer in $O(nm)$.
