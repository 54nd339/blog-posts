---
title: Parallel Hardware and Programming Models - Caches, GPUs, MPI, OpenMP, CUDA
description: "The machines and the code that drives them - cache coherence by snooping and directories, NUMA and transactional memory, GPU architecture and the SIMT model, the parallel programming patterns, and the four dominant models: MPI for distributed memory, Pthreads and OpenMP for shared memory, CUDA for GPUs."
date: 2023-06-25
draft: false
slug: /parallel-computing/parallel-architect
tags:
  - Parallel Computing
  - Architecture
  - Distributed Systems
---

The [cost model](/citadel/parallel-computing/parallel-computing) tells you whether a parallel program is good; the [algorithm models](/citadel/parallel-computing/parallel-algorithms) tell you how to design one. This post is the concrete layer between them: the hardware features that make shared memory work (and where it breaks), the GPU as a parallel engine, and the four programming models you actually write code in.

## Cache coherence

In a shared-memory system, each processor caches data for speed — and now two caches can hold stale copies of the same memory location. **Cache coherence** is the machinery that keeps them consistent.

- **Private caches** (per-processor L1/L2) are fast but create the coherence problem. **Shared caches** (an L3 across cores) sidestep it at that level but can bottleneck. Real systems layer both.
- **Snooping protocols** — on a bus where every cache can watch every transaction. When a processor writes a cached block, it broadcasts; other caches snoop the bus and invalidate or update their copies. State machines like **MSI**, **MESI**, and **MOESI** label each cache line (Modified, Exclusive, Shared, Invalid, Owned). Simple, but limited by bus bandwidth to modest processor counts.
- **Directory protocols** — for larger, non-bus systems. A **directory** records which processors share or own each block; on a write, only the caches that actually hold a copy get invalidation messages. No broadcast, so it scales further.

**NUMA** (non-uniform memory access) means access latency depends on which memory bank; **NUCA** applies the same idea to a large distributed last-level cache, either statically mapped (**S-NUCA**) or dynamically migrating hot data closer to its user (**D-NUCA**). **Inclusion** policies (data in L1 must also be in L2) simplify coherence; **exclusion** (in one or the other, not both) grows effective capacity.

**Transactional memory** raises the abstraction: mark a block of code as a transaction that executes atomically and in isolation, like a database transaction, and let the system detect and resolve conflicts. It comes as **software** (STM) or **hardware-assisted** (HTM), the latter faster.

## GPUs

Built for graphics, GPUs are now general parallel processors with thousands of simple cores, tuned for throughput on data-parallel work. They run **SIMT** (Single Instruction, Multiple Threads) — a SIMD variant where many threads execute the same instruction on different data but keep independent control state.

A modern GPU is a set of **streaming multiprocessors** (SMs), each with many cores and a hierarchical memory: fast on-chip shared memory per SM, L1/L2 caches, and large high-bandwidth global device memory. Hardware schedulers juggle thousands of threads to hide memory latency. **GPGPU** — general-purpose computing on GPUs — applies this to non-graphics work: deep learning, scientific simulation, image and video processing, cryptography, financial modelling, medical imaging.

Vector addition $C = A + B$ is the canonical example: each element $C_i = A_i + B_i$ is one thread, and thousands run at once. The programmer's job is thinking in thousands of threads, managing the host↔device memory transfer, and respecting the memory hierarchy.

## Parallel programming patterns

Reusable structures that recur across problems and models:

- **Fork-join** — a master thread spawns workers, waits for all to finish, continues.
- **Pipeline** — data flows through stages, each stage on its own processor.
- **Map** — apply a function independently to every element (scaled vector $y_i = a x_i + y_i$; each Mandelbrot pixel).
- **Reduce** — combine a collection to one value with an associative operator.
- **Scan** — prefix sums/products across a collection.
- **Stencil** — update each grid element from its neighbours (image filters, PDE solvers).
- **Gather / scatter / pack** — collect distributed items, distribute items, or compress out unused elements.
- **Recurrence** — each term depends on earlier ones; parallelisable only for certain forms.

Fusing patterns (map-then-reduce, map-then-scan) avoids materialising the intermediate — the idea behind MapReduce.

## The four programming models

### MPI — distributed memory

The **Message Passing Interface** is the standard for clusters and supercomputers. Each **process** has private memory; they exchange data by explicit messages. Core concepts: **communicators** (process groups), point-to-point `MPI_Send` / `MPI_Recv`, and collective operations `MPI_Bcast`, `MPI_Reduce`, `MPI_Scatter`, `MPI_Gather`, `MPI_Alltoall` with optimised implementations.

To approximate an integral by the trapezoidal rule, split the interval across processes, each sums its local trapezoids, then `MPI_Reduce` adds the partials into the global answer. **Derived datatypes** let one call send a non-contiguous layout (a matrix column); **MPI-IO** handles parallel reads and writes of large datasets. **Non-blocking** sends and receives let a process compute while messages move, hiding latency.

### Pthreads — shared memory, low level

**POSIX threads** create threads within one process that share memory. `pthread_create` / `pthread_join` manage lifetimes; **mutexes** give mutual exclusion; **condition variables** (paired with a mutex) let threads wait for a predicate; **read-write locks** allow concurrent readers but exclusive writers, worthwhile when reads dominate. The programmer must watch **false sharing** — unrelated variables on the same cache line ping-ponging between cores — and lay data out to avoid it.

### OpenMP — shared memory, directive based

**OpenMP** parallelises C/C++/Fortran through compiler directives — higher level than Pthreads. Annotate a loop with `#pragma omp parallel for` and its iterations are split across a thread team. **Loop scheduling** (`static`, `dynamic`, `guided`, `runtime`) controls how iterations are handed out. Shared-data correctness uses `critical`, `atomic`, or locks.

### CUDA — GPUs

NVIDIA's model for GPGPU. You write **kernels** — functions run by many GPU threads — in an extension of C/C++. Threads are organised as a **grid** of **blocks** of **threads**; threads in a block cooperate through shared memory and synchronise with `__syncthreads()`. The memory model exposes **global** (large, off-chip), **shared** (fast, per-block), **constant** (cached, read-only broadcast), and **texture** (2-D spatial locality) memory, plus per-thread registers. **Atomics** (`atomicAdd`, `atomicCAS`) make read-modify-write indivisible; **streams** let kernel launches and memory copies overlap; **events** time and synchronise host-device work.

| Model | Memory | Level | Where it runs |
| --- | --- | --- | --- |
| MPI | distributed | explicit messages | clusters, supercomputers |
| Pthreads | shared | low-level threads | one multicore node |
| OpenMP | shared | directives | one multicore node |
| CUDA | GPU hierarchy | kernels + threads | NVIDIA GPUs |

## The one idea to keep

Shared memory is a convenient illusion the hardware maintains at real cost — coherence protocols, NUMA latency, false sharing — and the programming model you pick is largely a choice about how much of that cost and control you want to see. MPI makes communication explicit and scales across machines; OpenMP hides threading behind pragmas on one node; CUDA trades a steep memory model for a GPU's throughput. None removes the need for the [decomposition and load-balancing](/citadel/parallel-computing/parallel-computing) work.
