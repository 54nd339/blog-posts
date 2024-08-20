---
title: Minkowski Sum of Convex Polygons - Merging Edge Sequences
description: Adding two convex polygons pointwise by merging their edge vectors in angular order, why the result has at most the combined edge count, and the collision-detection use where A minus B contains the origin.
date: 2024-08-20
draft: false
slug: /cp/minkowski-sum
tags:
  - Competitive Programming
  - Geometry
  - Convex Hull
---

The Minkowski sum $A \oplus B = \{a + b : a \in A,\ b \in B\}$ of two convex polygons is another convex polygon, and it is cheap to build: sort both polygons' edge vectors by angle and merge them into one edge sequence. The classic use is collision detection — two convex shapes overlap iff $A \oplus (-B)$ contains the origin.

## The problem

Given convex polygons $A$ (vertices $a_0, \dots, a_{n-1}$ CCW) and $B$ ($b_0, \dots, b_{m-1}$ CCW), compute $A \oplus B$ as a convex polygon.

Example: a square $\oplus$ a square (axis-aligned) is a larger square; a triangle $\oplus$ a segment is a pentagon (or a quadrilateral if the segment is parallel to an edge).

## The idea

Walk the boundary of $A \oplus B$. Its edges are **exactly the edges of $A$ and the edges of $B$**, sorted by polar angle. Intuition: as you traverse $A \oplus B$ CCW, at each moment you are advancing along either an edge of $A$ or an edge of $B$ — whichever edge direction comes next in angular order. So:

1. Rotate each polygon so its lowest (then leftmost) vertex is first.
2. Two pointers $i$ over $A$'s edges, $j$ over $B$'s edges. Start the output at $a_0 + b_0$.
3. Repeatedly append the current point plus whichever edge vector ($a_{i+1} - a_i$ or $b_{j+1} - b_j$) has the smaller polar angle (advance both on a tie), advancing that pointer.

The output has at most $n + m$ vertices.

## Algorithm

```python
def cross(o, a, b):
    return (a[0]-o[0])*(b[1]-o[1]) - (a[1]-o[1])*(b[0]-o[0])

def _reorder(poly):
    k = min(range(len(poly)), key=lambda i: (poly[i][1], poly[i][0]))
    return poly[k:] + poly[:k]

def minkowski_sum(A, B):
    A, B = _reorder(A), _reorder(B)
    A = A + [A[0], A[1]]                          # sentinels to read edges past the end
    B = B + [B[0], B[1]]
    res = []
    i = j = 0
    n, m = len(A) - 2, len(B) - 2
    while i < n or j < m:
        res.append((A[i][0] + B[j][0], A[i][1] + B[j][1]))
        ea = (A[i + 1][0] - A[i][0], A[i + 1][1] - A[i][1])
        eb = (B[j + 1][0] - B[j][0], B[j + 1][1] - B[j][1])
        c = ea[0] * eb[1] - ea[1] * eb[0]         # angle comparison via cross product
        if j >= m or (i < n and c > 0):
            i += 1
        elif i >= n or (j < m and c < 0):
            j += 1
        else:                                    # parallel edges: consume both
            i += 1
            j += 1
    return res
```

Both inputs must be convex, CCW, and free of consecutive collinear vertices for the clean version.

## Uses

- **Collision detection (GJK-flavoured).** Convex $A$ and $B$ intersect iff $0 \in A \oplus (-B)$, where $-B = \{-b : b \in B\}$. Build the Minkowski sum once, then a [point-in-convex-polygon](/citadel/cp/point-in-polygon) test.
- **Minimum distance between two convex polygons** = distance from the origin to $\partial(A \oplus (-B))$.
- **"Can this convex robot translate from here to there without hitting an obstacle"** — configuration-space obstacle is obstacle $\oplus (-\text{robot})$.
- **Summing many polygons** — fold the pairwise sum; the total edge count is the sum of all edge counts, so it stays linear overall.
- **Polygon "thickening"** — $P \oplus D$ with $D$ a small disk (approximated by a regular polygon) rounds/expands $P$.

## Complexity

- **Time:** $O(n + m)$ after ensuring both polygons are CCW (an $O(n)$ check / reorder; $O(n \log n)$ if you must hull them first).
- **Space:** $O(n + m)$.

## Common pitfalls

- **Non-convex input.** The edge-merge only works for convex polygons. Hull them first if needed.
- **Winding order.** Both must be CCW. A CW polygon gives a reflected, wrong sum.
- **Starting vertex.** Both polygons must start at their lowest vertex (bottom-most, then left-most) so the angular merge begins from the same reference; skipping `_reorder` misaligns the sweep.
- **Parallel edges.** When an edge of $A$ is parallel to an edge of $B$ (same direction), advance **both** pointers and emit the combined edge — otherwise you get a spurious zero-length or duplicated vertex.
- **Collinear vertices in the output.** The result may have three consecutive collinear points where an $A$ edge and a $B$ edge line up; clean them if the consumer needs strict convexity.

## The keystone

The Minkowski sum of two convex polygons has edges that are precisely the union of both polygons' edges sorted by angle — so a two-pointer merge of the edge vectors builds it in $O(n + m)$. Its headline application: convex shapes $A$, $B$ collide iff $A \oplus (-B)$ contains the origin.
