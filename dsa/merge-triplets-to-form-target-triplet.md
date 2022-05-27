---
title: Merge Triplets to Form Target Triplet - Ignore Anything That Overshoots
description: Whether repeated element-wise max of chosen triplets can produce a target — discard every triplet that exceeds the target in any position, then check the survivors hit each target coordinate.
date: 2022-05-27
draft: false
slug: /dsa/merge-triplets-to-form-target-triplet
tags:
  - Competitive Programming
  - Greedy
  - Arrays
---

The merge operation is element-wise `max`, which never decreases a value. So any triplet with a component larger than the target is poison — using it once ruins that coordinate forever. Keep only triplets that are `<=` target in all three slots, then ask whether those alone can reach each target component.

## Description

You are given a list of `triplets` and a `target` triplet. In one operation you pick two triplets and replace them with their element-wise maximum. Return `true` if you can obtain `target` as one of the triplets after some operations.

**Example**

```
Input:  triplets = [[2,5,3],[1,8,4],[1,7,5]], target = [2,7,5]
Output: true

Input:  triplets = [[3,4,5],[4,5,6]], target = [3,2,5]
Output: false
```

**Constraints**

- $1 \le \text{triplets.length} \le 10^5$
- $1 \le \text{triplet}[i]_j, \text{target}_j \le 1000$

## Prerequisites

- Element-wise `max` is monotone: it never shrinks a coordinate.
- Filter-then-verify greedy.

## Approach 1: Filter and check coverage

### Intuition

Discard any triplet that has some component strictly greater than the matching target component. Among the rest, you may freely merge all of them (merging can only help). Return `True` iff, across the survivors, each of the three positions equals the target somewhere.

### Algorithm

1. `found = [False, False, False]`.
2. For each triplet `t`: if `t[0] > target[0]` or `t[1] > target[1]` or `t[2] > target[2]`, skip. Else for `j` in `0..2`: if `t[j] == target[j]`, `found[j] = True`.
3. Return `all(found)`.

```python
def mergeTriplets(triplets: list[list[int]], target: list[int]) -> bool:
    found = [False, False, False]
    for a, b, c in triplets:
        if a > target[0] or b > target[1] or c > target[2]:
            continue
        if a == target[0]:
            found[0] = True
        if b == target[1]:
            found[1] = True
        if c == target[2]:
            found[2] = True
    return all(found)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Keeping a triplet that overshoots in one slot but matches in another.** One use permanently pushes that slot above the target; it must be discarded entirely, matches elsewhere notwithstanding.
- **Checking `>=` instead of `==` for coverage.** After filtering, no survivor exceeds the target, so `t[j] == target[j]` is the right test; `>=` would be equivalent only because `>` is already excluded, but `==` is clearer.
- **Requiring one triplet to match all three.** Different survivors can supply different coordinates; the merge combines them.
- **Forgetting that merging is optional and order-free.** You can merge every survivor together, so coverage is all that matters.

## The keystone

When an operation is monotone, elements that push past the goal are simply unusable — filter them out and the problem shrinks to "do the safe elements cover every requirement". Recognising a monotone operation is what makes the greedy filter valid.
