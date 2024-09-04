---
title: Minimum Enclosing Circle - Welzl's Randomized Algorithm
description: The smallest circle covering a point set, why its boundary is fixed by at most three points, and Welzl's expected-linear incremental algorithm that rebuilds the circle only when a new point falls outside.
date: 2024-09-04
draft: false
slug: /cp/minimum-enclosing-circle
tags:
  - Competitive Programming
  - Geometry
  - Randomized Algorithms
---

The smallest circle containing all $n$ given points is unique, and its boundary passes through at most **three** of them (or two, as a diameter). Welzl's algorithm finds it in **expected $O(n)$** time by adding points one at a time and only recomputing the circle when a point lands outside — a beautiful example of randomized incremental construction.

## The problem

Given points $P_1, \dots, P_n$, find the centre and radius of the smallest enclosing circle (a.k.a. minimum covering circle, smallest bounding circle).

Uses: "smallest radar range covering all cities", broadcast-tower placement, a step in some clustering and facility-location problems, and geometric outlier bounds.

## The idea

Two facts:

1. **The minimum enclosing circle is unique**, and its boundary contains either 2 points (they are a diameter) or 3 points (a circumscribed circle).
2. **Incremental property:** if $D$ is the MEC of points $P_1, \dots, P_{i-1}$ and $P_i \in D$, then $D$ is still the MEC of $P_1, \dots, P_i$. If $P_i \notin D$, then $P_i$ lies **on the boundary** of the new MEC.

Welzl: process points in random order, maintaining the current circle $D$. For each $P_i$ outside $D$, recompute $D$ as the MEC of $P_1, \dots, P_i$ with $P_i$ *forced onto the boundary*. That sub-problem recurses with one more boundary constraint; with 3 boundary points the circle is determined.

Because the order is random, the expected number of "point falls outside, rebuild" events is $O(\log n)$ per level and the total expected work is $O(n)$.

## Algorithm (iterative, three nested passes)

```python
import math, random

def _circle_from2(a, b):
    cx, cy = (a[0] + b[0]) / 2, (a[1] + b[1]) / 2
    r = math.hypot(a[0] - cx, a[1] - cy)
    return (cx, cy, r)

def _circle_from3(a, b, c):
    ax, ay = a; bx, by = b; cx, cy = c
    d = 2 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by))
    if abs(d) < 1e-12:
        return None                                   # collinear
    ux = ((ax*ax + ay*ay) * (by - cy) + (bx*bx + by*by) * (cy - ay) + (cx*cx + cy*cy) * (ay - by)) / d
    uy = ((ax*ax + ay*ay) * (cx - bx) + (bx*bx + by*by) * (ax - cx) + (cx*cx + cy*cy) * (bx - ax)) / d
    r = math.hypot(ax - ux, ay - uy)
    return (ux, uy, r)

def _in_circle(c, p, eps=1e-7):
    return math.hypot(p[0] - c[0], p[1] - c[1]) <= c[2] + eps

def min_enclosing_circle(points):
    pts = points[:]
    random.shuffle(pts)
    c = None
    for i, p in enumerate(pts):
        if c is not None and _in_circle(c, p):
            continue
        c = (p[0], p[1], 0.0)                          # p on boundary
        for j in range(i):
            q = pts[j]
            if _in_circle(c, q):
                continue
            c = _circle_from2(p, q)                    # p, q on boundary
            for k in range(j):
                r = pts[k]
                if _in_circle(c, r):
                    continue
                c3 = _circle_from3(p, q, r)            # p, q, r on boundary
                if c3 is not None:
                    c = c3
    return c                                           # (cx, cy, radius)
```

The three nested loops look like $O(n^3)$, but the `continue` short-circuits fire almost always; the expected total iteration count is $O(n)$.

## Complexity

- **Time:** expected $O(n)$ (the randomised analysis); worst case $O(n^3)$ but astronomically unlikely with a shuffle.
- **Space:** $O(n)$ for the shuffled copy.

## Common pitfalls

- **Forgetting to shuffle.** Adversarial input order can hit the $O(n^3)$ worst case; `random.shuffle` first.
- **Epsilon in `_in_circle`.** A boundary point should test as "inside"; use a small positive tolerance ($10^{-7}$), scaled to the coordinate magnitude if inputs are large.
- **Collinear triple.** `_circle_from3` returns `None`; the loop then keeps the two-point circle, which is correct (three collinear points cannot all be on a circle unless two coincide).
- **Duplicate / coincident points.** Harmless but wasteful; dedupe if $n$ is large.
- **All points identical or $n \le 1$.** Return a zero-radius circle at that point; guard $n = 0$.
- **Integer input, exact output wanted.** The centre is generally irrational (circumcentre); this problem is inherently floating point.

## The keystone

The minimum enclosing circle is pinned by at most 3 boundary points, and adding a point that is already inside never changes the circle. Welzl processes points in random order, rebuilding only on an outside point (which must then lie on the new boundary), and three nested short-circuited passes give expected $O(n)$.
