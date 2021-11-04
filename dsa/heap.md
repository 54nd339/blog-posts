---
title: Heap and Priority Queue - Cheap Access to the Extreme
description: A guide to the Heap section of NeetCode 150 — when an algorithm repeatedly needs the current minimum or maximum, a binary heap makes that O(log n) instead of O(n), and a size-k heap answers top-k in O(n log k).
date: 2021-11-04
draft: false
slug: /dsa/heap
tags:
  - Competitive Programming
  - Heap
  - Design
---

A binary heap gives $O(1)$ read and $O(\log n)$ removal of the smallest (or largest) element, and $O(\log n)$ insertion. Reach for one whenever an algorithm's inner loop is "take the current extreme, do something, maybe put a new value back".

Python's `heapq` is a **min-heap**. For a max-heap, negate values on the way in and out.

## The recurring uses

**Repeated-extreme simulation.** Each round operates on the current max or min.

- [Last Stone Weight](/citadel/dsa/last-stone-weight) — smash the two heaviest, push back the difference.
- [Task Scheduler](/citadel/dsa/task-scheduler) (heap version) — run the most frequent ready task each slot.

**Streaming top-k / order statistics.** Keep a size-`k` heap so each update is $O(\log k)$.

- [Kth Largest Element in a Stream](/citadel/dsa/kth-largest-element-in-a-stream) — a size-`k` min-heap; its root is the answer.
- [K Closest Points to Origin](/citadel/dsa/k-closest-points-to-origin) — size-`k` max-heap by distance (or quickselect for one-shot).
- [Kth Largest Element in an Array](/citadel/dsa/kth-largest-element-in-an-array) — size-`k` heap, or quickselect for average $O(n)$.

**k-way merge.** Merge many sorted sources by heaping their current fronts.

- [Design Twitter](/citadel/dsa/design-twitter) — merge per-user tweet histories by timestamp.

**Two heaps facing each other.** Partition a stream into halves.

- [Find Median from Data Stream](/citadel/dsa/find-median-from-data-stream) — max-heap lower half, min-heap upper half, kept size-balanced.

## Heap vs the alternatives

- **Full sort** — use when you need *everything* ordered, or the data is static and you query many times.
- **Quickselect** — one-shot "k-th element" or "k smallest, unordered" in average $O(n)$; no good for streams.
- **Bucket / counting** — beats a heap when the sort key is an integer in a small known range.
- **Heap** — the streaming answer, and the answer when you repeatedly need the extreme *and* insert new values.

## Recognising it

- "k-th largest / smallest", "top k", "k closest", "k most frequent" → size-`k` heap (or quickselect).
- "median of a stream", "sliding median" → two heaps.
- "merge k sorted ..." → heap of fronts.
- A simulation whose every step touches the current max/min → heap.

## Where this goes next

Heaps are the priority queue inside Dijkstra and Prim ([Advanced Graphs](/citadel/dsa/advanced-graphs)), where "closest unvisited node" is exactly a repeated-min query.
