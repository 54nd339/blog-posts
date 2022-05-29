---
title: Partition Labels - Extend the Cut to Every Letter's Last Home
description: Splitting a string into the most parts so no letter spans two parts — precompute each letter's last index, then sweep, closing a part when the scan reaches the farthest last-index seen so far.
date: 2022-05-29
draft: false
slug: /dsa/partition-labels
tags:
  - Competitive Programming
  - Greedy
  - Hash Table
---

A part cannot end before the last occurrence of any letter it contains. Record `last[ch]` for every letter. Sweep with a running `end = max(last[ch])` over the letters seen since the part began; when the index catches up to `end`, cut.

## Description

Given a string `s`, partition it into as many parts as possible so that each letter appears in at most one part. Return a list of the part sizes, in order.

**Example**

```
Input:  s = "ababcbacadefegdehijhklij"
Output: [9,7,8]

Input:  s = "eccbbbbdec"
Output: [10]
```

**Constraints**

- $1 \le \text{s.length} \le 500$
- `s` is lowercase English letters.

## Prerequisites

- Precomputing the last index of each character.
- The greedy sweep: close a segment when the cursor meets the running maximum last-index.

## Approach 1: Last-occurrence sweep

### Intuition

`last[ch]` = final index of `ch`. Walk the string; keep `end` = the largest `last[ch]` among characters in the current part, and `start` = where the part began. When `i == end`, the part `[start, i]` is complete — record `i - start + 1`, then start a new part at `i + 1`.

### Algorithm

1. `last = {ch: i for i, ch in enumerate(s)}`.
2. `start = 0`, `end = 0`, `res = []`.
3. For `i, ch` in `enumerate(s)`: `end = max(end, last[ch])`. If `i == end`: `res.append(i - start + 1)`; `start = i + 1`.
4. Return `res`.

```python
def partitionLabels(s: str) -> list[int]:
    last = {ch: i for i, ch in enumerate(s)}
    res = []
    start = end = 0
    for i, ch in enumerate(s):
        end = max(end, last[ch])
        if i == end:
            res.append(i - start + 1)
            start = i + 1
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ — the `last` map holds at most 26 entries.

## Common Pitfalls

- **Cutting at the first character's last index.** You must extend `end` by every character encountered *within* the part, not just the one that opened it.
- **Comparing `i` to `last[ch]` alone.** The close condition is `i == end`, where `end` is the running max across the whole current part.
- **Off-by-one on the size.** The part `[start, i]` inclusive has `i - start + 1` characters.
- **Rebuilding `last` inside the loop.** Compute it once up front.

## The keystone

"A boundary must stretch to cover the farthest dependency seen so far" is the interval-merging instinct applied to a string. It is essentially [Merge Intervals](/citadel/dsa/merge-intervals) where each letter contributes the interval `[first, last]` and you report the merged block sizes.
