---
title: Point in Polygon - Ray Casting and the O(log n) Convex Test
description: Deciding whether a point is inside an arbitrary simple polygon by counting ray crossings, the winding-number alternative, and the angular binary search that answers it in O(log n) for a convex polygon.
date: 2024-08-20
draft: false
slug: /cp/point-in-polygon
tags:
  - Competitive Programming
  - Geometry
  - Polygons
---

"Is point $Q$ inside this polygon?" For a general simple polygon it is $O(n)$ ray casting; for a convex polygon, an $O(\log n)$ angular binary search answers many queries fast. Both hinge on the [orientation test](/citadel/cp/basic-geometry) and both need an explicit decision about points *on* the boundary.

## The problem

Given a simple polygon (vertices in order) and a query point $Q$, return `INSIDE`, `OUTSIDE`, or `ON_BOUNDARY`.

## General polygon: ray casting

Shoot a ray from $Q$ (conventionally in the $+x$ direction) and count how many polygon edges it crosses. **Odd** ⇒ inside, **even** ⇒ outside. First, check each edge for "$Q$ is on this edge" and return `ON_BOUNDARY`.

The fiddly part is edges that pass exactly through $Q$'s $y$-coordinate at a vertex — count an edge only if one endpoint is strictly above $Q.y$ and the other is at or below (a half-open rule), so a vertex is not double-counted.

```python
def orient(a, b, c):
    return (b[0]-a[0])*(c[1]-a[1]) - (b[1]-a[1])*(c[0]-a[0])

def on_segment(a, b, p):
    return orient(a, b, p) == 0 and \
           min(a[0], b[0]) <= p[0] <= max(a[0], b[0]) and \
           min(a[1], b[1]) <= p[1] <= max(a[1], b[1])

def point_in_polygon(poly, q):
    n = len(poly)
    inside = False
    for i in range(n):
        a, b = poly[i], poly[(i + 1) % n]
        if on_segment(a, b, q):
            return "ON_BOUNDARY"
        # half-open crossing rule: does the edge straddle q.y with a-below/b-above or vice versa?
        if (a[1] > q[1]) != (b[1] > q[1]):
            # x-coordinate of the edge at height q.y
            x_cross = a[0] + (q[1] - a[1]) * (b[0] - a[0]) / (b[1] - a[1])
            if x_cross > q[0]:
                inside = not inside
    return "INSIDE" if inside else "OUTSIDE"
```

### Winding number (integer-exact alternative)

Sum the signed angle subtended by each edge as seen from $Q$; a full $\pm 2\pi$ means inside. Implemented without trig: accumulate whether each edge crosses the positive-$x$ ray from $Q$, with a $\pm 1$ per crossing by direction. Robust for self-touching cases and works in pure integers (no division) if you compare `orient` and $y$-signs instead of computing `x_cross`.

## Convex polygon: O(log n) per query

Preprocess: pick vertex $P_0$; the other vertices $P_1, \dots, P_{n-1}$ are sorted by angle around $P_0$ (they already are, since the polygon is convex and given in order). For a query $Q$:

1. If $Q$ is outside the wedge between rays $P_0P_1$ and $P_0P_{n-1}$ (check `orient(P0, P1, Q)` and `orient(P0, P_{n-1}, Q)` signs), it is outside.
2. Binary search for the sector: the largest $i$ with `orient(P0, P_i, Q) >= 0`. Then $Q$ is inside iff it is on the inner side of edge $P_i P_{i+1}$: `orient(P_i, P_{i+1}, Q) >= 0`.

```python
def prep_convex(poly):
    # poly given CCW; returns (P0, sorted fan) -- here just poly itself
    return poly

def point_in_convex(poly, q):
    n = len(poly)
    p0 = poly[0]
    if orient(p0, poly[1], q) < 0 or orient(p0, poly[n - 1], q) > 0:
        return "OUTSIDE"
    lo, hi = 1, n - 1
    while hi - lo > 1:
        mid = (lo + hi) // 2
        if orient(p0, poly[mid], q) >= 0:
            lo = mid
        else:
            hi = mid
    d = orient(poly[lo], poly[lo + 1], q)
    if d < 0:
        return "OUTSIDE"
    if d == 0 or orient(p0, poly[1], q) == 0 or orient(p0, poly[n - 1], q) == 0:
        return "ON_BOUNDARY"
    return "INSIDE"
```

(Requires CCW order and no three collinear vertices for the clean version; handle collinear boundary explicitly.)

## Complexity

- **Ray casting:** $O(n)$ per query, $O(1)$ preprocessing.
- **Convex binary search:** $O(\log n)$ per query after $O(n)$ (or $O(n\log n)$ if you must sort the fan).
- **Many queries, arbitrary polygon:** [point location](/citadel/cp/point-location) with a persistent sweep gives $O(\log n)$ per query after $O(n \log n)$.

## Common pitfalls

- **Boundary policy.** Decide once whether `ON_BOUNDARY` counts as inside; check it *before* the crossing count so a point on an edge is not misclassified by the parity.
- **Vertex-grazing rays.** The half-open rule `(a.y > q.y) != (b.y > q.y)` counts each edge consistently; using `>=` on both sides double-counts vertices.
- **`x_cross` division by zero.** Horizontal edges have `b.y == a.y`; the half-open test already skips them (both endpoints on the same side of `q.y`), so the division is safe — but only if you keep that guard.
- **Convex test needs CCW and convexity.** Feeding it a CW polygon or a slightly non-convex one gives wrong answers silently. Normalise with the [signed area](/citadel/cp/polygon-area).
- **Integer exactness.** Ray casting with `x_cross` uses floating point. The winding-number / pure-`orient` version stays exact for integer input — prefer it when coordinates are integers.

## The keystone

For any simple polygon, cast a ray and count edge crossings with a half-open rule — odd is inside — after a boundary check. For a convex polygon, binary-search the angular sector around one vertex and test the far edge, giving $O(\log n)$ per query.
