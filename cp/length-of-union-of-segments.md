---
title: Length of the Union of Segments - Sort and Sweep
description: Measuring how much of the line is covered by n intervals by sorting endpoints and sweeping a depth counter, and the coordinate-compressed segment-tree version for the 2-D rectangle-area analogue.
date: 2024-08-14
draft: false
slug: /cp/length-of-union-of-segments
tags:
  - Competitive Programming
  - Geometry
  - Sweep Line
---

Given $n$ intervals on a line, how long is the part of the line covered by at least one of them? Overlaps must not be double-counted. Sort the endpoints, sweep left to right tracking how many intervals are currently open, and add the gap between consecutive events whenever that count is positive.

## The problem

Intervals $[l_1, r_1], \dots, [l_n, r_n]$ (real or integer endpoints, possibly overlapping or nested). Output the measure of $\bigcup_i [l_i, r_i]$.

Example: $[1, 4], [2, 6], [8, 10]$ → covered length $= (6 - 1) + (10 - 8) = 7$.

## The idea

Turn each interval into two **events**: `(l, +1)` (an interval opens) and `(r, -1)` (an interval closes). Sort all $2n$ events by coordinate. Sweep: keep a running `open` counter; between event $i$ and event $i+1$, if `open > 0` the stretch $[\text{pos}_i, \text{pos}_{i+1}]$ is covered — add its length. Apply the event's delta as you pass it.

Ties: process opens before closes at the same coordinate if touching endpoints ($[1,2]$ and $[2,3]$) should count as connected; the delta ordering does not affect the *length* here, only a "number of connected components" variant.

## Algorithm

```python
def union_length(segments) -> float:
    events = []
    for l, r in segments:
        if l < r:
            events.append((l, +1))
            events.append((r, -1))
    events.sort()
    total = 0.0
    open_cnt = 0
    prev = None
    for pos, delta in events:
        if open_cnt > 0 and prev is not None:
            total += pos - prev
        open_cnt += delta
        prev = pos
    return total
```

Alternatively, sort intervals by left endpoint and merge: keep a current $[\text{cl}, \text{cr}]$; for each interval, if its left $\le \text{cr}$ extend $\text{cr} = \max(\text{cr}, r)$, else close out $[\text{cl}, \text{cr}]$ and start a new one. Same $O(n \log n)$, and it also gives you the merged intervals.

## Complexity

- **Time:** $O(n \log n)$ — the sort dominates.
- **Space:** $O(n)$.

## The 2-D analogue: area of a union of rectangles

The same sweep, one dimension up. Sweep a vertical line left to right; events are rectangle left edges (`+1`) and right edges (`-1`). At each event, the covered **height** on the sweep line (a 1-D union-length over $y$-intervals) times the horizontal gap to the next event is the area swept. Maintain the covered height with a **segment tree over compressed $y$-coordinates**, where each node stores `(count of full covers, covered length)` and a range-add of $\pm 1$; the root's covered length is the answer per slab. Total $O(n \log n)$.

```python
# sketch: sweep events (x, y1, y2, +/-1) sorted by x
# seg tree over sorted unique y's: update(y1, y2, delta); query = covered length at root
# area += covered_length * (x_next - x_cur)
```

The perimeter of the union, and the union of $k$-fold-covered regions, are variations on the same node aggregate.

## Common pitfalls

- **Zero-length intervals.** $l = r$ contributes nothing; filter them so they do not create paired events at the same point that cancel oddly.
- **Adding length before applying the delta.** The stretch between `prev` and `pos` is covered based on the `open_cnt` *before* this event. Add first, then update the counter.
- **`prev is None` on the first event.** Guard it; there is no interval before the first event.
- **Touching intervals and "components".** For total length, `[1,2] ∪ [2,3]` is length $2$ either way. For "how many maximal covered pieces", you must order opens before closes at equal coordinates.
- **2-D without coordinate compression.** Real or large $y$-coordinates need compression to a segment tree of size $O(n)$; a tree over raw coordinates is infeasible.

## The keystone

Convert intervals to $\pm 1$ endpoint events, sort, and sweep: the union length is the sum of gaps where the open-interval count is positive — $O(n \log n)$. One dimension up, sweep vertically and keep the covered height with a segment tree over compressed $y$'s to get the area of a union of rectangles.
