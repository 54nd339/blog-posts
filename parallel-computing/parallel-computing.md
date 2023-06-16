---
title: Parallel and Distributed Computing - Why, and the Cost Model
description: The case for many processors and the framework for judging them - shared versus distributed memory and Flynn's taxonomy, decomposition and mapping, the collective communication operations, and the performance model of speedup, efficiency, Amdahl's and Gustafson's laws, and isoefficiency.
date: 2023-06-16
draft: false
slug: /parallel-computing/parallel-computing
tags:
  - Parallel Computing
  - Distributed Systems
  - Architecture
---

A single processor has a speed ceiling set by physics — heat, power, the speed of light across a chip. Climate models, web-scale search, and training large neural networks all need more compute than that ceiling allows, so the work is split across many processing elements running at once. That is **parallel computing** (many processors, one machine, tightly coupled) shading into **distributed computing** (many autonomous machines over a network, loosely coupled, with partial failures to handle).

Splitting work introduces costs a sequential program never pays: processors have to communicate, coordinate, and stay balanced. This post is the framework — how parallel machines are organised, how you design for them, and how you tell whether a parallel program is actually any good. The [algorithm building blocks](/citadel/parallel-computing/parallel-algorithms) and the [hardware and programming models](/citadel/parallel-computing/parallel-architect) each have their own post.

## What makes it hard

Every parallel program fights the same five forces:

- **Communication overhead** — moving data between processors takes time the computation does not.
- **Synchronization** — coordinating so results are combined in the right order.
- **Load balancing** — keeping every processor equally busy; the slowest one sets the pace.
- **Fault tolerance** — in distributed settings, individual nodes and links fail mid-computation.
- **Scalability** — whether performance keeps improving as you add processors.

## How parallel machines are organised

### Flynn's taxonomy

Classify by how many instruction and data streams flow at once:

| | Single data | Multiple data |
| --- | --- | --- |
| **Single instruction** | SISD — an ordinary uniprocessor | SIMD — one instruction, many data lanes (vector units, GPUs) |
| **Multiple instruction** | MISD — rare; fault-tolerant / systolic arrays | MIMD — independent instruction streams (multicore, clusters) |

MIMD is the general case and the common one; SIMD is how GPUs and vector extensions get their throughput.

### Shared versus distributed memory

- **Shared-memory (multiprocessors)** — all processors read and write one global address space; communication is implicit through shared variables. Access is **UMA** (uniform — every processor equidistant from memory, as in a multicore desktop) or **NUMA** (non-uniform — access time depends on which memory bank, as in large multi-socket systems).
- **Distributed-memory (multicomputers)** — each processor has private memory; communication is explicit **message passing** over an interconnection network. Clusters and supercomputers.

The end of single-core scaling pushed **multicore** and **manycore** (GPU) designs into the mainstream, so parallel programming is now unavoidable even on one chip.

### The communication cost model

Sending an $m$-word message between two processors costs roughly

$$t_{comm} = t_s + m \cdot t_w,$$

where $t_s$ is the fixed startup **latency** and $t_w$ is the per-word transfer time (the inverse of **bandwidth**). Because $t_s$ dominates for small messages, batching many small messages into one is a recurring optimization. The **topology** of the network (bus, ring, mesh, hypercube, fat-tree) sets its diameter and bisection bandwidth; **routing** can be store-and-forward or cut-through, deterministic or adaptive.

## Designing a parallel algorithm

### Decomposition

Break the problem into tasks that can run concurrently:

- **Data (domain) decomposition** — split the data, give each task a piece (different regions of an image, different rows of a matrix).
- **Task (functional) decomposition** — split the work into distinct operations that can overlap (a pipeline of stages).

**Granularity** is the task size. Fine-grained tasks expose more parallelism but pay more communication and synchronization overhead per unit of work; coarse-grained tasks pay less overhead but may leave processors idle. The right point balances the two.

### Mapping and load balancing

Assign tasks to processors to minimise communication and keep load even:

- **Static mapping** — decided before execution from known task characteristics (block, cyclic, block-cyclic distributions of an array).
- **Dynamic load balancing** — adjusted at runtime when load drifts: task queues, or **work stealing**, where an idle processor takes work from a busy one.

Contain interaction overhead by maximising data locality, minimising message volume and frequency, and overlapping communication with computation.

### Algorithm models

Recurring skeletons:

- **Data-parallel** — every task does the same operation on its slice of a large dataset.
- **Task graph** — tasks are nodes, dependencies are edges; schedule respecting the edges.
- **Master-worker (work pool)** — a master hands tasks to a pool of workers.
- **Pipeline** — data streams through stages, each on a different processor.

## Collective communication operations

A handful of group communication patterns show up in almost every parallel algorithm. Naively each is a loop of point-to-point messages; in practice they use tree- or hypercube-shaped schedules to cut the step count.

| Operation | What it does |
| --- | --- |
| **One-to-all broadcast** | one process sends the same message to all |
| **All-to-one reduction** | all contribute a value; an associative operator (sum, max, OR) combines them at one process |
| **All-reduce** | reduction whose result is delivered to every process |
| **All-to-all broadcast** | every process sends its own distinct message to all |
| **Prefix sum (scan)** | process $P_i$ gets $x_0 \oplus x_1 \oplus \dots \oplus x_i$ for an associative $\oplus$ |
| **Scatter** | one process distributes a distinct item to each other |
| **Gather** | each process's item is collected at one process |
| **All-to-all personalized (total exchange)** | every process sends a *distinct* message to every other |
| **Circular shift** | $P_i$ sends to $P_{(i+k) \bmod p}$ |

## Measuring a parallel program

Let $T_s$ be the best sequential time and $T_p$ the time on $p$ processors.

- **Speedup** $S_p = T_s / T_p$. Ideal is $p$.
- **Efficiency** $E_p = S_p / p$. Ideal is 1 — every processor fully utilised.
- **Cost (work)** $W_p = p \cdot T_p$ — total processor-time. The system is **cost-optimal** if $W_p$ grows asymptotically like $T_s$.

### Amdahl's law

If a fraction $f$ of the work is inherently sequential, then no matter how many processors,

$$S_p \le \frac{1}{f + (1-f)/p} \xrightarrow[p \to \infty]{} \frac{1}{f}.$$

A program that is 5% sequential cannot exceed a $20\times$ speedup. This is the pessimistic view: fixed problem size.

### Gustafson's law

In practice you scale the problem up as you add processors. If the parallel part grows with $p$ while the sequential part stays fixed, the achievable **scaled speedup** is roughly $p - f(p-1)$ — near-linear. Which law applies depends on whether you are holding the problem fixed or the time fixed.

### Scaling and isoefficiency

- **Strong scaling** — speedup versus $p$ at fixed problem size $W$ (the Amdahl regime).
- **Weak scaling** — speedup versus $p$ with $W/p$ held constant (the Gustafson regime).
- **Isoefficiency function** — how fast $W$ must grow with $p$ to hold efficiency constant. A slowly growing isoefficiency function means the system scales well.

## The one idea to keep

Parallelism is a trade: you buy wall-clock time with communication, synchronization, and idle processors. Whether the trade pays off is not a matter of opinion — speedup, efficiency, and the isoefficiency function measure it, and Amdahl's law puts a hard ceiling on any fixed-size problem set by its sequential fraction. Design (decomposition, mapping, locality) is the work of pushing a real program toward that ceiling.
