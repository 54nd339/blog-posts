---
title: Garbage Collection - Reachability, Pauses, and Algorithms
description: How a runtime reclaims memory nothing points at any more. Reachability from roots, the stop-the-world problem, and the algorithm family from mark-sweep through generational to G1.
date: 2025-04-15
draft: false
slug: /interview/garbage-collection
tags:
  - System Design
  - Interview Prep
  - Memory
---

In C you call `free` and hope you got it right — not too early (dangling pointer), not twice (double free), not never (leak). A **garbage collector** removes that decision: the runtime periodically finds objects the program can no longer reach and reclaims their memory automatically. Java, C#, Go, Python, and JavaScript all do this, and the differences between their collectors come down to one trade — how long the application must pause while collection happens.

This covers the idea GC rests on, the pause problem, and the algorithms in order of sophistication.

## Reachability

A collector decides what is garbage by what is **reachable**. Starting from a set of **roots** — local variables and parameters on live thread stacks, static fields, values in CPU registers, references held by native code — it follows every reference. Any object reached is **live**; any object not reached, directly or through a chain, is garbage, and its memory is safe to reclaim. This is why reference cycles among unreachable objects are still collected: unreachable from a root is the only test.

Two facts shape almost every real collector:

- **Stop-the-world.** To see a consistent object graph, many collectors must pause all application threads during collection. Those pauses are what users feel as jank, and cutting them is the main axis of GC design.
- **The generational hypothesis** — *most objects die young*. A freshly allocated object is very likely to become garbage almost immediately; one that has survived a while will probably keep surviving.

## The algorithms

**Mark and sweep.** Two phases: *mark* — walk from the roots, set a bit on every live object; *sweep* — scan the whole heap linearly, and add every unmarked object's memory to a free list. Simple, and it handles cycles. But it leaves the heap **fragmented** — many small non-contiguous holes, so a large allocation can fail with plenty of total free space — and both phases are stop-the-world.

**Mark and compact.** Mark as before, then *compact*: slide all live objects to one end of the heap, closing the gaps, and fix up every pointer to a moved object. Fragmentation is gone and new allocation is fast, but the compaction pass is expensive and lengthens the pause.

**Copying (semi-space).** Split the heap into two equal halves, *from-space* and *to-space*. Allocate only in from-space, by bumping a pointer. When it fills, copy every live object into to-space — laid down contiguously, so to-space is compacted for free — update pointers, declare from-space empty, and swap the roles. Allocation and collection are both fast when few objects are live. The cost is stark: half the heap is unusable at any moment.

**Generational.** Apply the hypothesis. Split the heap into a **young generation** (new objects; often an Eden space plus two survivor spaces) and an **old generation** (objects that have survived several collections are *promoted* here). *Minor GC* collects only the young generation, frequently and fast — usually a copying collector between Eden and survivor — because most of what it finds is already dead. *Major GC* collects the old generation, rarely and more slowly, often with mark-compact. The catch: a pointer from an old object to a young one must be found during a minor GC without scanning the whole old generation, which needs bookkeeping — **card tables** or **remembered sets** — updated on every such write.

**Concurrent.** Do the expensive phases, especially marking, while the application runs, so the stop-the-world time drops to a few short pauses (initial root scan, final mark). It needs careful synchronisation because the application mutates the graph mid-collection. Java's CMS was an early one, low-latency for the old generation but prone to fragmentation and floating garbage.

**G1 (Garbage-First).** The modern default for large heaps. The heap is carved into many equal **regions**, each dynamically Eden, survivor, or old (a "humongous" object can span several). G1 marks concurrently across the whole heap, then, knowing which regions hold the most garbage, collects those first by *evacuation* — copying their live objects into fresh regions during a pause whose length it sizes to a user-set goal. Predictable pauses on big heaps, at some throughput overhead.

## In practice

- **Java** ships several: Serial, Parallel (throughput), G1 (balanced, default), and the low-pause ZGC and Shenandoah.
- **.NET** has a generational collector in the CLR; **V8** (JavaScript) a sophisticated generational one; **Go** a concurrent tri-colour mark-sweep tuned for low latency.
- **Python** is mainly reference counting, with a supplemental cyclic collector for reference cycles that counting alone leaks.
- **Tuning** — generation sizes, a pause-time goal, the choice of collector — is a real lever for latency-sensitive versus throughput-oriented workloads.

## GC does not mean no leaks

A collector reclaims what is *unreachable*, not what is *unneeded*. A cache that never evicts, a listener list that never unsubscribes, a static map that only grows — all keep their entries reachable, so the collector leaves them alone and memory climbs. These "logical leaks" are the memory bug that survives into managed languages, and they are found with a heap profiler, not a `free`. The heap itself and its place in the [memory hierarchy](/citadel/interview/computer-memories) is the other half of the picture.
