---
title: Activity Selection - Greedily Scheduling the Most Non-Overlapping Tasks
description: Given tasks with start and finish times competing for one resource, pick the largest set that never overlaps. Sorting by finish time and always taking the next compatible task is optimal, and an exchange argument proves it.
date: 2021-02-28
draft: false
slug: /algorithms/ActivitySelection
tags:
  - Algorithms
  - Greedy
---

One conference room, a list of meeting requests each with a start and finish time. Book the most meetings you can without any two overlapping. This is **activity selection**: given $n$ activities $(s_i, f_i)$, find the largest set in which no two intervals intersect.

There are $2^n$ subsets to consider, but you don't need to. Sort by finish time and repeatedly grab the earliest-finishing activity that still fits — that's optimal.

## Why finish time

Picking the activity that *frees the room soonest* leaves the most time for everything after it. Picking the shortest activity, or the one that starts earliest, can both go wrong: a short activity in the middle of the day can block two others, and an early-starting one can run long.

The greedy rule:

1. Sort activities by finish time.
2. Take the first one.
3. Scan the rest; take an activity whenever its start is $\ge$ the finish of the last one taken.

Trace it on $(1,4), (3,5), (0,6), (5,7), (3,9), (5,9), (6,10), (8,11), (8,12), (2,14), (12,16)$ — already finish-sorted. Take $(1,4)$. $(3,5)$ starts at 3 < 4, skip; $(0,6)$ skip; $(5,7)$ starts at 5 ≥ 4, take. Now skip everything starting before 7 until $(8,11)$, take. Then $(8,12)$, $(2,14)$ start too early; $(12,16)$ starts at 12 ≥ 11, take. Result: four activities, $(1,4), (5,7), (8,11), (12,16)$.

## The code

```python
def activity_selection(activities):          # activities: list of (start, finish)
    chosen = []
    last_finish = float("-inf")
    for start, finish in sorted(activities, key=lambda a: a[1]):
        if start >= last_finish:
            chosen.append((start, finish))
            last_finish = finish
    return chosen


acts = [(1, 4), (3, 5), (0, 6), (5, 7), (3, 9), (5, 9),
        (6, 10), (8, 11), (8, 12), (2, 14), (12, 16)]
picked = activity_selection(acts)
assert len(picked) == 4
assert picked == [(1, 4), (5, 7), (8, 11), (12, 16)]
```

## Why it's optimal

An **exchange argument**. Let $a_1$ be the first activity by finish time (the greedy pick), and let $A$ be any optimal solution, sorted by finish time. If $A$'s first activity is some $a_k \ne a_1$, then because $f_1 \le f_k$, swapping $a_k$ for $a_1$ keeps every activity in $A$ pairwise disjoint (the new one finishes no later, so it still doesn't collide with $A$'s second activity) and doesn't change the count. So there's an optimal solution containing $a_1$ — the **greedy choice property**.

Once $a_1$ is fixed, the rest of the problem is activity selection on just the activities starting at or after $f_1$ — a smaller instance of the same problem, whose optimal solution combines with $a_1$ to give the optimal whole (**optimal substructure**). Induct, and greedy is optimal.

## Cost

The sort dominates: **$O(n \log n)$**. The scan is $O(n)$. Space is $O(n)$ for the output.

## The takeaway

Activity selection is the cleanest example of when greedy works: a local rule (finish earliest) provably extends to a global optimum. The same two-part argument — greedy choice plus optimal substructure — justifies [Huffman coding](/citadel/algorithms/HuffmanCoding), [minimum spanning trees](/citadel/algorithms/MinimumSpanningTree), and [fractional knapsack](/citadel/algorithms/FractionalKnapsack). Add a *value* to each activity and ask for maximum total value instead of maximum count, and greedy breaks — that version (weighted interval scheduling) needs [dynamic programming](/citadel/algorithms/01Kanpsack).
