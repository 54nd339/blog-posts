---
title: Convex Hull - Andrew's Monotone Chain and Graham Scan
description: Building the smallest convex polygon containing a point set in O(n log n) by sorting and sweeping a stack that only ever turns one way, plus the collinear-point policy and rotating-calipers follow-ups.
date: 2024-08-22
draft: false
slug: /cp/convex-hull-construction
tags:
  - Competitive Programming
  - Geometry
  - Convex Hull
---

The convex hull of a point set is the smallest convex polygon containing all of them — the shape a rubber band snaps to. Andrew's monotone chain builds it in $O(n \log n)$: sort the points, then sweep left-to-right and right-to-left maintaining a stack that only ever makes turns in one direction. It is the launching point for diameter, width, and many other "extremal" queries.

## The problem

Given $n$ points, output the vertices of their convex hull in counter-clockwise order. Decide up front: keep or drop points that lie **on** a hull edge (collinear).

Example: points $(0,0), (1,1), (2,2), (2,0), (0,2)$ → hull $(0,0), (2,0), (2,2), (0,2)$; the point $(1,1)$ is interior, and whether $(0,0)$–$(2,2)$'s midpoint... it is not on the hull here.

## The idea (monotone chain)

1. **Sort** the points by $x$, breaking ties by $y$.
2. **Lower hull:** iterate left to right; push each point, and while the last three points make a non-left turn (`orient <= 0`), pop the middle one.
3. **Upper hull:** iterate right to left with the same rule.
4. Concatenate, dropping the duplicated endpoints.

Because the points are $x$-sorted, each chain is a function of $x$ and the stack only turns one way; every point is pushed and popped at most once, so the sweep is $O(n)$ after the $O(n \log n)$ sort.

`orient <= 0` (pop on collinear) **excludes** collinear points from the hull; `orient < 0` **keeps** them.

## Algorithm

```python
def convex_hull(points):
    pts = sorted(set(points))
    if len(pts) <= 2:
        return pts

    def cross(o, a, b):
        return (a[0]-o[0])*(b[1]-o[1]) - (a[1]-o[1])*(b[0]-o[0])

    lower = []
    for p in pts:
        while len(lower) >= 2 and cross(lower[-2], lower[-1], p) <= 0:
            lower.pop()
        lower.append(p)

    upper = []
    for p in reversed(pts):
        while len(upper) >= 2 and cross(upper[-2], upper[-1], p) <= 0:
            upper.pop()
        upper.append(p)

    return lower[:-1] + upper[:-1]                # CCW, no repeated endpoints
```

**Graham scan** is the alternative: pick the lowest point, sort the rest by polar angle around it, then one stack pass. Same complexity; monotone chain avoids `atan2` and is usually less error-prone.

## Follow-ups that need the hull

- **Diameter** (farthest pair) — [rotating calipers](/citadel/cp/manhattan-distance): walk two antipodal pointers around the hull, $O(n)$.
- **Width** (minimum over directions of the projection span) — calipers over each edge as the base.
- **Smallest enclosing rectangle** — one edge of the optimum rectangle lies on a hull edge; try all, $O(n)$ with calipers.
- **[Minkowski sum](/citadel/cp/minkowski-sum)** of two convex polygons — merge their edge vectors by angle, $O(n + m)$.
- **Convex polygon intersection**, **point in convex polygon** in $O(\log n)$ (see [point in polygon](/citadel/cp/point-in-polygon)).
- **Dynamic hull** (insertions) — a balanced BST keyed by angle, $O(\log^2 n)$ per update.

## Complexity

- **Time:** $O(n \log n)$, dominated by the sort. The two sweeps are $O(n)$.
- **Space:** $O(n)$.

## Common pitfalls

- **Collinear policy inconsistency.** Pick `<= 0` (drop collinear) or `< 0` (keep) and use the **same** comparator in both chains. Mixing them produces a broken polygon.
- **Fewer than 3 distinct points.** Return the points as-is; the stack logic needs at least 3.
- **Duplicate points.** `sorted(set(points))` removes them; duplicates can make `cross` zero and confuse the pop rule.
- **All points collinear.** With `<= 0` you get the two extreme points; with `< 0` you get all of them in a line — decide which the problem wants.
- **Output orientation.** Monotone chain as written yields CCW. If you need CW, reverse, or swap the chain order.
- **Integer overflow in `cross`.** For $10^9$-scale coordinates the product overflows 64-bit; Python is fine, C++ needs `long long` at minimum, often `__int128`.

## The keystone

Sort by $x$, then sweep a stack that pops whenever the last three points fail to turn left — twice, once each direction — and you have the hull in $O(n \log n)$. The comparator's `<=` vs `<` is the single knob for excluding or keeping collinear points, and the hull is the entry point to calipers-based extremal queries.
