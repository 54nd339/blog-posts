---
title: Binary Search Patterns - Bisecting More Than Arrays
description: A guide to the Binary Search section of NeetCode 150 — the array is often not the thing you search; whenever a yes/no test is monotone in some parameter, you can bisect that parameter in logarithmic time.
date: 2021-07-19
draft: false
slug: /dsa/binary-search-patterns
tags:
  - Competitive Programming
  - Binary Search
  - Problem Solving
---

Binary search halves a search space each step. The leap in this section is realising that the space need not be a sorted array of values — it can be *any* range over which a predicate is monotone: an eating speed, a partition index, a position in a rotated array.

## The one requirement

There must be a value `x` such that the predicate "does `x` work?" is `False` for everything below it and `True` for everything at or above it (or vice versa). One clean flip. Given that, you can find the boundary in $O(\log(\text{range}))$ by repeatedly testing the midpoint.

## Searching an array of values

- [Binary Search](/citadel/dsa/binary-search) — the template itself, plus the lower-bound / upper-bound variants that generalise to "first index satisfying a predicate".
- [Search a 2D Matrix](/citadel/dsa/search-a-2d-matrix) — one coordinate transform turns the grid into a sorted 1-D array.
- [Time Based Key-Value Store](/citadel/dsa/time-based-key-value-store) — timestamps arrive sorted, so "latest value at or before `t`" is a bisect.

## Searching a rotated array

- [Find Minimum in Rotated Sorted Array](/citadel/dsa/find-minimum-in-rotated-sorted-array) — one comparison with the right endpoint says which sorted run the midpoint is in.
- [Search in Rotated Sorted Array](/citadel/dsa/search-in-rotated-sorted-array) — at each midpoint one half is a clean sorted run; test membership against its range.

## Binary search on the answer

- [Koko Eating Bananas](/citadel/dsa/koko-eating-bananas) — bisect the eating speed; `feasible(k)` is monotone.
- [Median of Two Sorted Arrays](/citadel/dsa/median-of-two-sorted-arrays) — bisect the partition index of the smaller array; the "left halves are all `<=` right halves" test is monotone in it.

## The boundary-search template

For "smallest `x` with `predicate(x)` true":

```python
lo, hi = MIN, MAX          # half-open: answer in [lo, hi]
while lo < hi:
    mid = lo + (hi - lo) // 2
    if predicate(mid):
        hi = mid           # mid might be the answer; keep it
    else:
        lo = mid + 1        # mid fails; discard it
return lo
```

The two rules that prevent every classic bug: when the midpoint might still be the answer, set `hi = mid` (not `mid - 1`); when it cannot, set `lo = mid + 1`. Pair that with `while lo < hi` so the loop ends with `lo == hi` on the boundary.

## Recognising it

Reach for binary search when:

- The input is sorted, or can be treated as sorted after a transform.
- The brute force tries every candidate answer from small to large, and "does this candidate work?" only ever goes from no to yes.
- You need $O(\log n)$ and a linear scan is too slow.

## Where this goes next

"Test a candidate, and the test is monotone" recurs in [greedy](/citadel/dsa/greedy) feasibility checks and in parametric-search problems on graphs like [Swim in Rising Water](/citadel/dsa/swim-in-rising-water).
