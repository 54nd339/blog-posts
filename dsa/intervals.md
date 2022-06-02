---
title: Intervals - Sort First, Then One Sweep
description: A guide to the Intervals section of NeetCode 150 — every problem is sort the ranges by start or by end, then a single linear pass that merges, counts overlaps, or feeds a heap.
date: 2022-06-02
draft: false
slug: /dsa/intervals
tags:
  - Competitive Programming
  - Intervals
  - Sorting
---

An interval problem is almost always: **sort the intervals** (by start, or by end), then make **one pass**. The sort key and what you do in the pass are the only real decisions.

## The two sort keys

- **Sort by start** when you are *merging* or *inserting* — after a start-sort, an overlapping interval always overlaps the most recent one, so you only compare against the last kept entry.
- **Sort by end** when you are *packing the most non-overlapping intervals* — the earliest finish time leaves maximum room for the rest (activity selection).

## The overlap test

Two intervals `[a1, a2]` and `[b1, b2]` overlap iff `a1 <= b2 and b1 <= a2`. Touching (`a2 == b1`) counts as overlapping for merges but usually *not* for scheduling — check each problem's wording.

## Problem → move

- [Insert Interval](/citadel/dsa/insert-interval) — pre-sorted disjoint list; a three-phase pass: copy before, merge the overlapping run, copy after.
- [Merge Intervals](/citadel/dsa/merge-intervals) — sort by start; extend `res[-1]` or append.
- [Non-Overlapping Intervals](/citadel/dsa/non-overlapping-intervals) — sort by end; keep when `start >= prev_end`, else count a removal.
- [Meeting Rooms](/citadel/dsa/meeting-rooms) — sort by start; any adjacent overlap ⇒ false.
- [Meeting Rooms II](/citadel/dsa/meeting-rooms-ii) — min-heap of end times, or a `+1/-1` chronological sweep; the peak concurrency is the answer.
- [Minimum Interval to Include Each Query](/citadel/dsa/minimum-interval-to-include-each-query) — offline: sort queries, sweep intervals into a size-keyed heap, lazily drop expired ones.

## The sweep-line view

Any "how many overlap" question becomes: emit `(start, +1)` and `(end, -1)`, sort the events, and track a running counter. Its maximum is the peak overlap; the ranges where it is positive are the merged blocks. Tie-breaking at equal timestamps (does `+1` or `-1` go first) encodes whether touching intervals merge.

## Recognising it

- The input is a list of `[start, end]` pairs (or convertible to them — [Partition Labels](/citadel/dsa/partition-labels) makes each letter an interval).
- The question is about overlap, coverage, merging, or scheduling.
- Sorting by one endpoint makes the problem linear.

## Where this goes next

The heap-of-end-times and sweep-line ideas feed forward into scheduling and range-query problems; the "sort then single pass" discipline is the same one behind [Math & Geometry](/citadel/dsa/math-and-geometry) grid sweeps.
