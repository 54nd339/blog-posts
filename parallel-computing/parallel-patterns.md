---
title: Parallel Patterns and Performance Models
description: Parallel programs are built from a small vocabulary - map, reduce, scan, stencil, pipeline - and their speedup is bounded by two laws. Amdahl says a fixed problem's serial fraction caps you hard; Gustafson says a growing problem lets you keep scaling. The roofline model tells you whether you are compute- or memory-bound.
date: 2023-07-21
draft: false
slug: /parallel-computing/parallel-patterns
tags:
  - Parallel Computing
  - Parallel Algorithms
---

Most parallel code is an instance of one of a handful of patterns. Recognising which one you have tells you how to decompose the work, where the communication is, and roughly what speedup to expect. And two simple formulas — Amdahl's law and Gustafson's law — set the ceiling: they explain why some programs stop speeding up at 8 cores and others scale to 8,000.

This post is that vocabulary and those bounds.

## Decomposition

Before patterns, the split:

- **Data parallelism** — the same operation applied to many data elements independently (add two arrays, apply a filter to every pixel). Scales with data size; the default target for GPUs and SIMD.
- **Task parallelism** — different operations running concurrently (one thread parses, one compresses, one writes). Scales with the number of distinct tasks, which is usually small and fixed.

**Granularity** is the size of the unit of parallel work. Too fine and scheduling/synchronization overhead dominates; too coarse and load is uneven and some cores finish early and idle. The sweet spot is problem- and machine-dependent.

## The pattern vocabulary

- **Map** — apply `f` to every element, independently. Embarrassingly parallel: split the array across cores, no communication. `y[i] = f(x[i])`.
- **Reduce** — combine all elements with an associative operator into one value (sum, max, logical-and). Parallelised as a **tree**: each core reduces its chunk locally, then log(P) rounds combine the partials. O(n/P + log P).
- **Scan (prefix sum)** — `y[i] = x[0] ⊕ x[1] ⊕ … ⊕ x[i]`. Looks inherently sequential; isn't. A work-efficient parallel scan does an up-sweep (build partial sums up a tree) then a down-sweep (push prefixes back down) in 2·log(n) steps. The building block for stream compaction, radix sort, and allocation ("where does each thread's output go?").
- **Stencil** — each output element is a function of a fixed-shape neighbourhood of the input (blur, heat diffusion, cellular automata, PDE solvers). Parallel by tiling; the communication is **halo exchange** — each tile needs a border of its neighbours' data every step.
- **Pipeline** — a sequence of stages, each processing a stream; stage `k+1` starts on item `i` while stage `k` works on item `i+1`. Parallelism = number of stages, throughput limited by the slowest stage.
- **Fork–join** — recursively split a task into subtasks, run them in parallel, join their results (parallel merge sort, quicksort). Backed by a **work-stealing** scheduler.
- **Map–reduce** — map over a huge dataset, group by key, reduce per group. The distributed-systems pattern; the [message-queue](/citadel/system-design/distributed-message-queue) and big-data stacks are built on it.

Real programs compose these: a physics step is a stencil; the convergence check on top is a reduce; the whole simulation is a pipeline of I/O, compute, and output.

## Scheduling

- **Static** — assign work to cores up front. Zero runtime overhead, best when every unit takes the same time. Falls apart under irregular workloads.
- **Dynamic** — cores pull work from a shared queue as they finish. Handles imbalance; the queue can become a bottleneck.
- **Work-stealing** (Cilk, Java's ForkJoinPool, Go's scheduler, Rust's Rayon) — each core has its own deque; when it empties, it *steals* from a random busy core's other end. Low contention (mostly local), self-balancing, provably good bounds for fork–join. The standard for irregular parallelism.

## Amdahl's law

If a fraction `s` of a program is inherently serial and `(1 − s)` is perfectly parallelisable, the speedup on `P` processors is:

$$\text{speedup}(P) = \frac{1}{s + \frac{1 - s}{P}}$$

As `P → ∞`, speedup → `1/s`. If 5% of the work is serial, the maximum speedup is **20×**, no matter how many cores you throw at it. At `P = 100` you're already at ~16.8×; the next 900 cores buy you 3× more. This is why "just add cores" hits a wall — and why finding and removing serial sections (a global lock, a sequential I/O, an unparallelised setup phase) matters more than adding hardware.

## Gustafson's law

Amdahl fixes the problem size. Gustafson observes that in practice, **more cores are used to solve bigger problems in the same time**, not the same problem faster. If the *parallel* work grows with `P` while the serial part stays roughly constant, scaled speedup is:

$$\text{speedup}(P) = P - s \cdot (P - 1)$$

which is nearly linear in `P`. A weather model given 1000× the cores runs at 1000× the resolution, not 1000× faster at the old resolution — and there the serial fraction is a shrinking share of a growing whole.

The two laws aren't in conflict; they describe two regimes:

- **Strong scaling** (Amdahl) — fixed total problem, more cores. Bounded by the serial fraction. What you measure when you say "does my program get faster on more cores?"
- **Weak scaling** (Gustafson) — problem grows with cores, work per core fixed. Can stay near-linear. What supercomputing actually does.

The **Karp–Flatt metric** works backward from measured speedups to *estimate* the serial fraction, which reveals whether your scaling loss is intrinsic serialism or growing overhead (communication, synchronization, load imbalance).

## The roofline model

Once you're parallel, are you limited by compute or by memory bandwidth? Plot attainable performance (FLOP/s) against **arithmetic intensity** (FLOPs per byte moved from memory):

- A **sloped roof** — `intensity × peak_bandwidth` — for low-intensity kernels: you're **memory-bound**, and the fix is reducing data movement (blocking for cache, better layout, fusing passes), not more FLOPs.
- A **flat roof** — peak compute — for high-intensity kernels: you're **compute-bound**, and the fix is better vectorization, more cores, or a faster instruction mix.

A dense matrix multiply (O(n³) work over O(n²) data) has high intensity and lives under the flat roof; a vector add (one add per two loads and a store) has intensity ~1/12 and is firmly memory-bound. The model tells you which optimisation is even worth attempting.

## The one idea to keep

Parallel programs are compositions of a few patterns — map and reduce and scan and stencil and pipeline — and the pattern tells you the decomposition and where the communication lives. Speedup on a fixed problem is capped by Amdahl at `1/s`, so a 5% serial section limits you to 20× however many cores you have; scaling the problem with the cores (Gustafson) is how supercomputing stays near-linear. Once parallel, use the roofline model to decide whether you're fighting compute or memory bandwidth, because the two need opposite fixes.
