---
title: Manhattan Distance - Rotating to Chebyshev and MST Tricks
description: The 45-degree rotation that turns L1 distance into L-infinity, why it linearises farthest-pair and diameter queries, and the four-quadrant sweep that builds a Manhattan minimum spanning tree in O(n log n).
date: 2024-09-03
draft: false
slug: /cp/manhattan-distance
tags:
  - Competitive Programming
  - Geometry
  - Computational Geometry
---

Manhattan ($L_1$) distance $|x_1 - x_2| + |y_1 - y_2|$ has absolute values that make it awkward to optimise. Rotating the plane by $45°$ converts it to Chebyshev ($L_\infty$) distance $\max(|x'_1 - x'_2|, |y'_1 - y'_2|)$, which separates into independent coordinate maxima — and that unlocks fast farthest-pair, diameter, and minimum-spanning-tree algorithms.

## The rotation

Map $(x, y) \mapsto (x + y,\ x - y) = (u, v)$. Then

$$|x_1 - x_2| + |y_1 - y_2| = \max\big(|u_1 - u_2|,\ |v_1 - v_2|\big).$$

So $L_1$ in the original plane **is** $L_\infty$ in the rotated plane (up to the fixed factor from not normalising; the identity above is exact). The inverse map is $(x, y) = \big(\tfrac{u+v}{2}, \tfrac{u-v}{2}\big)$.

## What this buys you

- **Farthest pair / diameter under $L_1$:** in rotated coordinates it is $\max(\max u - \min u,\ \max v - \min v)$ — $O(n)$, no pairwise comparison.
- **Maximum $L_1$ distance from a set to a query point:** track $\min/\max$ of $x + y$ and $x - y$; the answer is $\max(|q_x + q_y - \text{ext}|, \dots)$ over the four extremes — $O(1)$ per query after $O(n)$ preprocessing.
- **Minimum enclosing $L_1$ ball** ↔ minimum enclosing $L_\infty$ box in rotated space — an axis-aligned square, trivially the tightest bounding box.
- **Chebyshev ↔ Manhattan** also runs the other way: $L_\infty$ problems become $L_1$ problems via the same rotation.

## Manhattan minimum spanning tree

Building an MST on $n$ points with edge weights = $L_1$ distance would be $O(n^2)$ edges. The key fact: **for each point, only its nearest neighbour in each of 8 octants can be an MST edge** — and by symmetry you only need 4 of them. So there are $O(n)$ candidate edges, and [Kruskal](/citadel/algorithms/MinimumSpanningTree) on them gives the MST in $O(n \log n)$.

For one octant — the "$45°$ wedge" where $x_j \ge x_i$ and $x_j - x_i \ge y_j - y_i \ge 0$ — connect $i$ to the active points in that wedge as a sweep in increasing $x + y$ passes, using an ordered map keyed by $-y$. Erasing each point once it is matched keeps the total work $O(n \log n)$. Four coordinate transforms cover all the octants you need.

```python
from sortedcontainers import SortedList      # or any balanced-BST ordered container

def manhattan_mst_edges(points):
    ps = [list(p) for p in points]
    n = len(ps)
    idx = list(range(n))
    edges = []
    for k in range(4):
        idx.sort(key=lambda i: ps[i][0] + ps[i][1])          # sweep by x + y
        active = SortedList()                                # entries: (-y, i)
        for i in idx:
            # match against active points with y <= ps[i].y, in that order
            lo = active.bisect_left((-ps[i][1], -1))
            j_removed = []
            for pos in range(lo, len(active)):
                _, j = active[pos]
                dx, dy = ps[i][0] - ps[j][0], ps[i][1] - ps[j][1]
                if dy > dx:
                    break
                edges.append((dx + dy, i, j))
                j_removed.append(pos)
            for pos in reversed(j_removed):
                del active[pos]
            active.add((-ps[i][1], i))
        # rotate/reflect coordinates for the next octant
        for i in range(n):
            if k & 1:
                ps[i][0] = -ps[i][0]
            else:
                ps[i][0], ps[i][1] = ps[i][1], ps[i][0]
    return edges                                             # feed to Kruskal
```

Run [Kruskal](/citadel/algorithms/MinimumSpanningTree) on the returned $O(n)$ edges for the MST. (`SortedList` stands in for a balanced BST; with the stdlib only, a `bisect`-managed list works but insertions cost $O(n)$.)

## Common pitfalls

- **The factor of 2 in the inverse rotation.** $(x, y) = \big(\tfrac{u+v}{2}, \tfrac{u-v}{2}\big)$ — half-integer coordinates appear; keep everything in $(u, v)$ space if you can, or scale by 2.
- **Rotation is not a similarity you can ignore.** $(x+y, x-y)$ scales distances by $\sqrt2$ in $L_2$ but the $L_1 \leftrightarrow L_\infty$ identity is exact — don't accidentally divide by $\sqrt2$.
- **MST octants: 4 vs 8.** You need the nearest neighbour in 4 of the 8 octants (the other 4 give the same edges from the other endpoint). Using the wrong 4, or all 8 without dedup, still works but wastes time; using only 2 misses edges.
- **Ties in the sweep.** Points with equal $x + y$ or equal $y - x$ need a consistent tie-break, or a candidate edge is dropped.
- **3-D and beyond.** The rotation trick generalises to $L_1 \leftrightarrow L_\infty$ in any dimension, but $L_\infty$ in $d$ dims needs $2^{d-1}$ sign combinations — fine for $d = 3$, exponential after.

## The keystone

$(x, y) \mapsto (x + y, x - y)$ turns $L_1$ distance into $L_\infty$, which decouples into per-coordinate maxima — so $L_1$ diameter and farthest-from-query become $O(n)$ extremes. The Manhattan MST exploits that only 4 directional nearest-neighbours per point can be MST edges, giving $O(n \log n)$ via 4 Fenwick sweeps plus Kruskal.
