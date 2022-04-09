---
title: Multiprocessors - Shared Memory, Coherence, and Clusters
description: How multiple processors are put together - Flynn's taxonomy, shared-memory UMA and NUMA versus distributed-memory clusters, the synchronization and cache-coherence problems that shared memory creates and the snooping and directory protocols that solve them, interconnection networks, and non-von-Neumann models.
date: 2022-04-09
draft: false
slug: /computer-architecture/multiprocessor
tags:
  - Computer Architecture
  - Parallelism
  - Multiprocessors
---

[Instruction-level parallelism](/citadel/computer-architecture/ilp) wrings speed from a single instruction stream and then hits diminishing returns. Past that point you add processors. But "add processors" immediately raises a design question — do they share one memory or each keep their own? — and whichever you pick, a coordination problem follows: keeping their caches consistent, or passing messages between them.

This post is the space of answers: Flynn's classification, shared-memory versus distributed-memory organisation, the coherence and synchronization machinery that shared memory needs, the networks that wire it all together, and a look at architectures that drop the von Neumann model entirely. The parallel *programming* and *algorithm* side is a [separate category](/citadel/parallel-computing/parallel-computing).

## Flynn's taxonomy

Classify by how many instruction streams and data streams run at once:

- **SISD** — one of each. The traditional uniprocessor.
- **SIMD** — one instruction stream, many data streams. [Array and vector processors](/citadel/computer-architecture/ilp); today, SIMD extensions and GPUs.
- **MISD** — many instruction streams on one data stream. Almost no practical examples.
- **MIMD** — many autonomous processors on many data streams. Most multiprocessors and all multicomputers.

MIMD splits by memory organisation:

- **Shared memory** — one global physical address space; processors communicate implicitly by reading and writing shared variables.
  - **UMA** (uniform memory access) — every processor reaches any location in about the same time. A **symmetric multiprocessor (SMP)** on a shared bus or crossbar; also a multicore chip sharing L3 and main memory.
  - **NUMA** (non-uniform memory access) — one address space, but memory is physically split among processors; local access is fast, remote access goes over a network and is slower.
- **Distributed memory** — each processor has a private address space and cannot touch another's; communication is explicit **message passing**. Clusters.

## Centralised shared memory (SMP / UMA)

Several processors, each with private caches, on a shared bus or crossbar to one main memory. Two problems come with the shared data.

### Synchronization

If two processors increment a shared counter at once, one update can be lost — a race. The fix is **atomic** read-modify-write instructions the hardware guarantees indivisible: **test-and-set**, **fetch-and-add**, **compare-and-swap**. From those you build the higher-level primitives — locks (mutexes), semaphores, barriers. The [operating-system treatment](/citadel/operating-system/process-thread) covers those primitives and the classic synchronization problems in depth.

### Cache coherence

The same memory block can sit in several caches. One processor writes its copy, and every other copy — plus main memory — is now stale. Two protocol families keep the view consistent.

**Snooping** (bus-based). Every cache controller watches the shared bus for other processors' transactions.

- *Write-invalidate* — on a write to a shared block, the writer broadcasts an invalidation; other caches drop their copy and re-fetch on next use. The common choice.
- *Write-update* — the writer broadcasts the new data and other caches update in place. Lower read latency elsewhere, but more bus traffic.
- Each line carries a state: **Invalid**, **Shared** (clean, possibly several copies), or **Exclusive/Modified** (dirty, this cache holds the only valid copy). Snooped transactions drive the state transitions. The refined versions — MSI, MESI, MOESI — are in [parallel hardware](/citadel/parallel-computing/parallel-architect).

**Directory-based.** A **directory** attached to memory records, for each block, which caches hold it and whether it's modified. On a miss or a write, it sends point-to-point messages only to the actual sharers instead of broadcasting. That's why snooping doesn't scale past a single bus and directories do.

## Interconnection networks

How processors and memory modules connect sets the bandwidth ceiling:

- **Shared bus** — cheapest, simplest, snooping-friendly; a bottleneck as processors are added.
- **Crossbar switch** — a dedicated path for every processor–memory pair; non-conflicting accesses proceed in parallel. Cost grows as $N \times M$, so it's expensive at scale.
- **Multiport memory** — memory modules with several independent access ports.
- **Multistage networks** (Omega, Butterfly) — small switching elements arranged in stages; a middle ground between a bus and a crossbar in cost and bandwidth, with higher latency than a crossbar.

## Distributed shared memory (NUMA)

Memory is placed with the processors but presented as one global address space. Local memory is fast; remote memory traverses the interconnect. Coherence is directory-based (there's no single bus to snoop). NUMA **scales** because aggregate memory bandwidth grows with the processor count — but performance depends heavily on **data locality**, so programs (and the OS) work to keep data near the processors that use it and to minimise remote accesses.

## Clusters

A **cluster** is a set of commodity computers — each a complete machine with its own processor, memory, and OS — connected by a fast network (Gigabit Ethernet, InfiniBand) and used as one resource. It's fundamentally distributed memory: no hardware global address space, communication through message-passing libraries like **MPI** or **PVM**. Clusters scale to thousands of nodes, keep running when a node fails, and are built from inexpensive parts. A software DSM layer can fake shared memory on top, at a large performance cost.

## Non-von-Neumann models

Specialised architectures that abandon the stored-program, program-counter model:

- **Dataflow** — no program counter. An operation is a node in a dataflow graph and *fires* as soon as all its input tokens arrive, exposing parallelism naturally. Matching tokens and handling data structures proved hard enough that no general-purpose dataflow machine stuck, though the ideas live on in stream processing.
- **Reduction** — based on functional programming: a program is an expression, evaluated by **reducing** it to a value, demand-driven (evaluate a sub-expression only when its result is needed). Independent sub-expressions reduce in parallel. Implemented as string or graph reduction.
- **Systolic arrays** — a regular grid of simple identical cells. Data is pumped rhythmically from the edges, each cell doing a small computation and passing results to its neighbour, deeply pipelined. Regular wiring makes it VLSI-friendly; it excels at matrix multiplication, FIR/IIR filtering, and dynamic-programming kernels — but each array is built for one algorithm.

## The one idea to keep

Adding processors is the easy part; the architecture is defined by the memory model, and every model charges a coordination fee. Shared memory buys easy communication and pays with cache-coherence traffic and the need for atomic synchronization primitives. Distributed memory buys scalability and pays by making every exchange an explicit message. NUMA and clusters are points on that trade-off, and the non-von-Neumann models are bets that a different execution model exposes parallelism more directly.
