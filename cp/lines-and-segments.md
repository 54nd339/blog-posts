---
title: Lines and Segments - Intersection Tests and Points
description: The orientation-based test for whether two segments cross, the collinear-overlap special case, line-line intersection by Cramer's rule, and how to keep it exact for integer input.
date: 2024-08-13
draft: false
slug: /cp/lines-and-segments
tags:
  - Competitive Programming
  - Geometry
  - Computational Geometry
---

"Do these two segments intersect?" and "where do these two lines meet?" are the second thing you need after the [cross-product primitive](/citadel/cp/basic-geometry). The segment test is a pair of orientation checks plus a collinear special case; the line-line point is Cramer's rule on a $2 \times 2$ system.

## The problem

- **Segment intersection test:** do segments $AB$ and $CD$ share at least one point? (Boolean.)
- **Segment intersection set:** the shared point, or the overlapping sub-segment if they are collinear, or empty.
- **Line intersection point:** given lines through $AB$ and $CD$, their unique crossing (or "parallel" / "same line").

## The idea

### Segment test by straddling

Segments $AB$ and $CD$ properly cross iff $C$ and $D$ are on **opposite sides** of line $AB$ **and** $A$ and $B$ are on opposite sides of line $CD$:

$$\text{orient}(A, B, C)\cdot\text{orient}(A, B, D) < 0 \quad\text{and}\quad \text{orient}(C, D, A)\cdot\text{orient}(C, D, B) < 0.$$

If any orientation is $0$, an endpoint lies on the other segment's line — check whether it is actually *on the segment* (within the bounding box, or `dot(P - X, P - Y) <= 0`). The fully collinear case (all four orientations involving one pair are $0$) is a 1-D overlap test on the projections.

### Line intersection by Cramer's rule

Write line $AB$ as $\vec{r} = A + t(B - A)$ and line $CD$ as $\vec{r} = C + s(D - C)$. Setting them equal and solving the $2 \times 2$ system: let $\vec u = B - A$, $\vec v = D - C$, $\vec w = C - A$. If $\vec u \times \vec v = 0$ the lines are parallel. Otherwise

$$t = \frac{\vec w \times \vec v}{\vec u \times \vec v}, \qquad \text{intersection} = A + t\,\vec u.$$

$t \in [0, 1]$ (and the analogous $s \in [0, 1]$) iff the crossing is within both segments.

## Algorithm

```python
def orient(a, b, c):
    return (b[0]-a[0])*(c[1]-a[1]) - (b[1]-a[1])*(c[0]-a[0])

def on_segment(a, b, p):                    # p is known collinear with a,b
    return min(a[0], b[0]) <= p[0] <= max(a[0], b[0]) and \
           min(a[1], b[1]) <= p[1] <= max(a[1], b[1])

def segments_intersect(a, b, c, d) -> bool:
    o1, o2 = orient(a, b, c), orient(a, b, d)
    o3, o4 = orient(c, d, a), orient(c, d, b)
    if (o1 > 0) != (o2 > 0) and o1 != 0 and o2 != 0 and \
       (o3 > 0) != (o4 > 0) and o3 != 0 and o4 != 0:
        return True                        # proper crossing
    if o1 == 0 and on_segment(a, b, c): return True
    if o2 == 0 and on_segment(a, b, d): return True
    if o3 == 0 and on_segment(c, d, a): return True
    if o4 == 0 and on_segment(c, d, b): return True
    return False

def line_intersection(a, b, c, d):
    ux, uy = b[0]-a[0], b[1]-a[1]
    vx, vy = d[0]-c[0], d[1]-c[1]
    denom = ux*vy - uy*vx
    if denom == 0:
        return None                        # parallel or identical
    wx, wy = c[0]-a[0], c[1]-a[1]
    t = (wx*vy - wy*vx) / denom
    return (a[0] + t*ux, a[1] + t*uy)

def segment_intersection_point(a, b, c, d):
    p = line_intersection(a, b, c, d)
    if p is None:
        return None                        # handle collinear-overlap separately
    if on_segment(a, b, p) and on_segment(c, d, p):
        return p
    return None
```

## Complexity

$O(1)$ per pair. Testing *all* pairs among $n$ segments is $O(n^2)$; use the [sweep-line method](/citadel/cp/intersecting-segments-sweep) for $O(n \log n)$ "does any pair cross".

## Common pitfalls

- **Only checking one straddle.** Both conditions are required. $C, D$ on opposite sides of $AB$ is necessary but not sufficient — $A, B$ must also straddle $CD$.
- **Endpoint touches.** `orient == 0` means "on the line", not "on the segment". Add the bounding-box / dot-product check. Decide up front whether a shared endpoint counts as intersecting.
- **Collinear overlap.** When all relevant orientations are $0$, the segments are on one line; intersection is an interval, found by sorting the four points along the line.
- **Floating-point `denom == 0`.** With integer input it is exact; with doubles use an epsilon, and be aware that near-parallel lines give a huge, imprecise intersection point.
- **Overflow.** `orient` on $10^9$-scale integer coordinates overflows 64-bit; Python is safe, C++ needs `__int128` or `long double` for the test.

## The keystone

Two segments cross iff each straddles the other's supporting line — two products of orientation signs, both negative — with `orient == 0` triggering an on-segment endpoint check and the all-collinear case handled as a 1-D interval overlap. The exact meeting point of two lines is Cramer's rule: $t = (\vec w \times \vec v)/(\vec u \times \vec v)$.
