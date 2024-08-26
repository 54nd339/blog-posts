---
title: A Pair of Intersecting Segments - The Sweep-Line Approach
description: Detecting whether any two of n segments cross in O(n log n) by sweeping a vertical line and keeping the segments it currently touches ordered by height, checking only newly adjacent pairs.
date: 2024-08-26
draft: false
slug: /cp/intersecting-segments-sweep
tags:
  - Competitive Programming
  - Geometry
  - Sweep Line
---

Testing all $\binom{n}{2}$ pairs of segments for intersection is $O(n^2)$. The Shamos-Hoey sweep does "**does any pair intersect?**" in $O(n \log n)$: move a vertical line left to right, keep the segments it currently crosses in an ordered set by $y$, and only ever test **adjacent** segments in that order — because the first intersection to occur makes two segments adjacent just before it.

## The problem

- **Detection (Shamos-Hoey):** do any two of $n$ line segments intersect? Yes/no (and a witness pair).
- **Reporting (Bentley-Ottmann):** list *all* $k$ intersection points in $O((n + k)\log n)$.

## The idea

**Events**, sorted by $x$:

- a **left endpoint** — the segment enters the sweep; insert it into the ordered structure at its $y$-position, and test it against its new neighbours above and below.
- a **right endpoint** — the segment leaves; before removing it, test its former upper and lower neighbours against each other (they become adjacent).
- (Bentley-Ottmann only) an **intersection point** — swap the two segments' order in the structure, and test the newly adjacent pairs.

The ordered structure (a balanced BST) is keyed by the $y$-coordinate of each segment *at the current sweep $x$*. The comparator is an [orientation test](/citadel/cp/basic-geometry), not a stored number, since the order changes as the sweep moves.

**Why adjacency suffices:** consider the leftmost intersection point $P$ of two segments $s, t$. Just left of $P$'s $x$, $s$ and $t$ are adjacent in the sweep order (nothing crosses between them before $P$, or that crossing would be further left). So the algorithm tests $s$ vs $t$ at the event that made them adjacent.

## Algorithm (detection)

```python
import bisect

def orient(a, b, c):
    return (b[0]-a[0])*(c[1]-a[1]) - (b[1]-a[1])*(c[0]-a[0])

def segments_intersect(a, b, c, d):
    d1, d2 = orient(a, b, c), orient(a, b, d)
    d3, d4 = orient(c, d, a), orient(c, d, b)
    if ((d1 > 0) != (d2 > 0)) and ((d3 > 0) != (d4 > 0)) and d1 and d2 and d3 and d4:
        return True
    def on(p, q, r):
        return orient(p, q, r) == 0 and min(p[0],q[0]) <= r[0] <= max(p[0],q[0]) and \
               min(p[1],q[1]) <= r[1] <= max(p[1],q[1])
    return on(a, b, c) or on(a, b, d) or on(c, d, a) or on(c, d, b)

def any_intersection(segments):
    # normalise each segment so p[0] <= q[0]
    segs = [tuple(sorted(s)) for s in segments]
    events = []
    for i, (p, q) in enumerate(segs):
        events.append((p[0], 0, i))              # 0 = left endpoint (process before removals)
        events.append((q[0], 1, i))              # 1 = right endpoint
    events.sort()

    def y_at(i, x):
        (px, py), (qx, qy) = segs[i]
        if qx == px:
            return py
        return py + (qy - py) * (x - px) / (qx - px)

    active = []                                  # list of segment indices, kept sorted by y_at

    def add(i, x):
        key = y_at(i, x)
        lo = bisect.bisect_left([y_at(j, x) for j in active], key)
        active.insert(lo, i)
        for nb in (lo - 1, lo + 1):
            if 0 <= nb < len(active) and nb != lo:
                if segments_intersect(*segs[i], *segs[active[nb]]):
                    return True
        return False

    def remove(i, x):
        pos = active.index(i)
        above = active[pos + 1] if pos + 1 < len(active) else None
        below = active[pos - 1] if pos - 1 >= 0 else None
        active.pop(pos)
        if above is not None and below is not None:
            return segments_intersect(*segs[above], *segs[below])
        return False

    for x, typ, i in events:
        if typ == 0:
            if add(i, x):
                return True
        else:
            if remove(i, x):
                return True
    return False
```

(The `active` list here is a plain list with $O(n)$ insert/remove for clarity — a balanced BST or `SortedList` keyed by the current $y$ makes each event $O(\log n)$, hence $O(n \log n)$ overall.)

## Complexity

- **Detection:** $O(n \log n)$ with a balanced structure.
- **Reporting all $k$ intersections:** $O((n + k)\log n)$ — the intersection events add $k$ more sweep stops.
- **Space:** $O(n)$ (plus a priority queue of pending intersection events for Bentley-Ottmann).

## Common pitfalls

- **Comparator stability.** The sweep order is by $y$ *at the current $x$*; recomputing it on the fly must be consistent, or the BST corrupts. Many implementations sweep at $x + \varepsilon$ conceptually to break ties at shared endpoints.
- **Vertical segments.** $qx = px$; give them a defined $y$-key (e.g. the lower endpoint) and process carefully — they are the classic edge case.
- **Shared endpoints.** Decide whether segments that merely touch at an endpoint count as "intersecting". The orientation test with `on(...)` says yes; exclude it if the problem wants proper crossings only.
- **Event order at equal $x$.** Process left endpoints before right endpoints (and, for Bentley-Ottmann, intersection events in between) so a segment is in `active` when its neighbour is inserted.
- **Floating point in `y_at`.** For integer input, compare $y$-keys by cross-multiplication to stay exact rather than computing a fractional $y$.

## The keystone

Sweeping a vertical line and keeping the crossed segments ordered by height means the first pair to intersect is adjacent in that order just before the crossing — so testing only newly-adjacent pairs at each endpoint event finds an intersection (if one exists) in $O(n \log n)$. Adding intersection events to the sweep upgrades it to report all $k$ crossings.
