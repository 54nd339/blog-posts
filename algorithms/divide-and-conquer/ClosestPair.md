---
title: Closest Pair of Points - O(n log n) with a Divide-and-Conquer Strip
description: Given n points in the plane, find the two closest together. Checking every pair is O(n^2); splitting the plane in half, recursing, and carefully merging across the split line brings it to O(n log n).
date: 2021-02-17
draft: false
slug: /algorithms/ClosestPair
tags:
  - Algorithms
  - Divide and Conquer
---

Given $n$ points in the plane, which two are closest? With a handful of points you measure all $\binom{n}{2}$ distances. With a million points that's $10^{12}$ distance calculations. Divide and conquer gets it to $O(n \log n)$ — but the merge step needs a genuine geometric argument to stay linear.

## The plan

Sort the points by $x$. Split at the median $x$ into a left half and a right half, recurse on each, and let $d$ be the smaller of the two closest-pair distances found.

The catch: the true closest pair might straddle the split, one point in each half. Such a pair can only beat $d$ if both points lie within distance $d$ of the split line — a vertical **strip** of width $2d$. So the merge step is: gather the strip, and check pairs within it.

Naively the strip could hold all $n$ points, and checking all pairs in it is $O(n^2)$ again. The rescue is geometric: sort the strip by $y$, and for each point you only need to compare against the next few points in that order. Any two strip points closer than $d$ have $y$-coordinates within $d$ of each other, and a $d \times 2d$ rectangle can hold at most 8 points that are all pairwise at least $d$ apart (one per roughly $d/2 \times d/2$ cell, split across the two halves). So each point compares against a constant number of neighbours — the strip scan is $O(n)$.

## The code

```python
import math

def closest_pair(points):
    px = sorted(points)                      # by x, then y
    py = sorted(points, key=lambda p: p[1])  # by y

    def dist(a, b):
        return math.hypot(a[0] - b[0], a[1] - b[1])

    def brute(pts):
        best = math.inf
        for i in range(len(pts)):
            for j in range(i + 1, len(pts)):
                best = min(best, dist(pts[i], pts[j]))
        return best

    def rec(px, py):
        n = len(px)
        if n <= 3:
            return brute(px)

        mid = n // 2
        midx = px[mid][0]
        qx, rx = px[:mid], px[mid:]
        left = set(qx)                        # split py by which half each point went to
        qy = [p for p in py if p in left]
        ry = [p for p in py if p not in left]

        d = min(rec(qx, qy), rec(rx, ry))

        strip = [p for p in py if abs(p[0] - midx) < d]
        for i in range(len(strip)):
            j = i + 1
            while j < len(strip) and strip[j][1] - strip[i][1] < d:
                d = min(d, dist(strip[i], strip[j]))
                j += 1
        return d

    return rec(px, py)


pts = [(2, 3), (12, 30), (40, 50), (5, 1), (12, 10), (3, 4)]
assert abs(closest_pair(pts) - math.hypot(1, 1)) < 1e-9   # (2,3)-(3,4)
```

The `py` list stays sorted by $y$ at every level, so the strip is already $y$-ordered when it's filtered out of `py` — no re-sorting inside the recursion.

## Cost

Each level does an $O(n)$ partition of `py` and an $O(n)$ strip scan, then two half-size recursive calls:

$$T(n) = 2T(n/2) + O(n) = O(n \log n).$$

The one-time initial sorts are also $O(n \log n)$. Space is $O(n)$ for the coordinate lists plus $O(\log n)$ recursion.

## The takeaway

The divide and the conquer are routine; the whole algorithm lives or dies on the merge. The insight that a bounded region can hold only a constant number of mutually-$d$-separated points is what turns a quadratic-looking scan into a linear one — a recurring move in computational geometry (it shows up again in sweep-line algorithms). The recurrence $2T(n/2) + O(n)$ is the same one behind [merge sort](/citadel/algorithms/SortingSearching) and [the FFT](/citadel/algorithms/FastFourierTransform).
