---
title: Half-Plane Intersection - Sort by Angle, Incremental Deque
description: Computing the convex region satisfying a set of linear inequalities by sorting the bounding lines by direction and pushing them onto a double-ended queue, popping half-planes made redundant from either end.
date: 2024-09-03
draft: false
slug: /cp/halfplane-intersection
tags:
  - Competitive Programming
  - Geometry
  - Convex Hull
---

A half-plane is "everything on one side of a line" — a linear inequality $ax + by \le c$. The intersection of $n$ half-planes is a convex region (possibly empty, a point, or unbounded). Sorting the half-planes by the direction of their boundary and sweeping a **double-ended queue** builds the region in $O(n \log n)$ — the dual of building a [convex hull](/citadel/cp/convex-hull-construction).

## The problem

Given half-planes $H_i = \{P : \vec n_i \times (P - Q_i) \ge 0\}$ (each stored as a point $Q_i$ on the boundary and a direction vector $\vec d_i$, with "inside" being to the **left** of $\vec d_i$), compute the boundary polygon of $\bigcap_i H_i$, or report that it is empty.

Uses: feasible region of linear constraints (2-variable LP), the set of points that can "see" all of a polygon (the **kernel** of a star-shaped polygon), the region within distance $r$ of every one of a set of lines, Voronoi cells (each cell is a half-plane intersection).

## The idea

Sort the half-planes by the polar angle of their direction $\vec d_i$. Process them in that order, maintaining a deque of half-planes whose boundaries form the current region's edges in CCW order:

- **Pop from the back** while the last two half-planes' boundary intersection lies *outside* (to the right of) the new half-plane — that last half-plane is now redundant.
- **Pop from the front** symmetrically (the new half-plane can make an early one redundant too).
- **Push the new half-plane to the back.**

After all are processed, do one more cleanup pass popping the back against the front (the deque is circular). The surviving half-planes' consecutive boundary intersections are the region's vertices.

Parallel half-planes with the same direction: keep only the more restrictive one. Two with **opposite** directions that do not overlap ⇒ empty intersection.

## Algorithm

```python
import math

def halfplane_intersection(planes, eps=1e-9):
    # each plane: (px, py, dx, dy)  -- boundary through (px,py) in direction (dx,dy); inside is left
    def angle(h): return math.atan2(h[3], h[2])

    def out(h, p):                                # is point p strictly right of (outside) h?
        return (h[2] * (p[1] - h[1]) - h[3] * (p[0] - h[0])) < -eps

    def inter(a, b):                              # intersection of the two boundary lines
        den = a[2] * b[3] - a[3] * b[2]
        t = ((b[0] - a[0]) * b[3] - (b[1] - a[1]) * b[2]) / den
        return (a[0] + t * a[2], a[1] + t * a[3])

    planes = sorted(planes, key=angle)
    dq = []
    for h in planes:
        while len(dq) >= 2 and out(h, inter(dq[-1], dq[-2])):
            dq.pop()
        while len(dq) >= 2 and out(h, inter(dq[0], dq[1])):
            dq.pop(0)
        if dq and abs(angle(dq[-1]) - angle(h)) < eps:
            # parallel: keep the more restrictive (the one whose inside is further along its normal)
            if out(dq[-1], (h[0], h[1])):
                dq[-1] = h
            continue
        dq.append(h)
    while len(dq) >= 3 and out(dq[0], inter(dq[-1], dq[-2])):
        dq.pop()
    while len(dq) >= 3 and out(dq[-1], inter(dq[0], dq[1])):
        dq.pop(0)
    if len(dq) < 3:
        return None                              # empty or degenerate
    return [inter(dq[i], dq[(i + 1) % len(dq)]) for i in range(len(dq))]
```

For a bounded result on unbounded inputs, add four half-planes forming a large box.

## Complexity

- **Time:** $O(n \log n)$ — the sort, then each half-plane pushed and popped at most once.
- **Space:** $O(n)$.

## Common pitfalls

- **Angle ties.** Half-planes with the same direction must be de-duplicated to the tightest one *before* the deque logic, or the intersection computations divide by zero.
- **`atan2` precision at the seam.** Sorting by `atan2` has a discontinuity at $\pm\pi$; ensure a consistent tie-break, or sort by a half-plane bucket + cross product like [angle_cmp](/citadel/cp/basic-geometry).
- **Empty vs unbounded.** Fewer than 3 surviving half-planes means the region is empty, a half-line, or a point — distinguish if the problem needs it. An unbounded feasible region needs the bounding box trick to produce a polygon.
- **Orientation convention.** "Inside is to the left of $\vec d$" must be consistent for every input half-plane; a sign error on one flips its meaning and the result is wrong, not obviously so.
- **`den == 0` in `inter`.** Parallel boundaries; the angle-tie check should have caught them first.

## The keystone

Sort the half-planes by boundary direction, then push them onto a deque, popping from either end any half-plane whose contribution is cut off by the newcomer. It is the convex hull algorithm in the dual: $O(n \log n)$ for the feasible region of $n$ linear inequalities, which is also how you compute a polygon's kernel and Voronoi cells.
