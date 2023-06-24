---
title: Designing Parallel Algorithms - PRAM Models and Building Blocks
description: "The theory side of parallel computing - PRAM models (EREW, CREW, CRCW) and interconnection topologies, then the standard parallelised algorithms: prefix sum, comparison-exchange sorts, matrix transpose and Cannon's multiplication, parallel graph traversal and Boruvka's MST, plus why parallel search can beat linear speedup."
date: 2023-06-24
draft: false
slug: /parallel-computing/parallel-algorithms
tags:
  - Parallel Computing
  - Parallel Algorithms
  - Architecture
---

Throwing processors at a sequential algorithm rarely helps — the algorithm has to be rebuilt so its work genuinely decomposes. This post is the design side: the abstract machine models you reason about parallel algorithms on, and a catalogue of the classic problems (prefix sums, sorting, matrix operations, graph traversal) rebuilt for parallel execution. The [cost model](/citadel/parallel-computing/parallel-computing) — speedup, efficiency, Amdahl's law — is the yardstick throughout.

## Models to design against

### PRAM

The **Parallel Random Access Machine** is the RAM model with $p$ processors sharing one memory, used to reason about SIMD shared-memory algorithms. Variants differ on simultaneous access to the *same* cell:

- **EREW** (Exclusive Read, Exclusive Write) — no concurrent access to one location. The most restrictive, closest to real hardware.
- **CREW** (Concurrent Read, Exclusive Write) — many processors may read one location at once; writes stay exclusive.
- **CRCW** (Concurrent Read, Concurrent Write) — concurrent writes allowed, with a rule to resolve them (common: all writers must agree; arbitrary: one wins; priority: lowest-indexed wins).

An algorithm's cost on EREW is the honest one; CRCW can be faster but assumes hardware that does not exist at scale.

### Interconnection topologies

For distributed-memory machines, the network shape drives the algorithm:

| Topology | Diameter (of $n$ nodes) | Good for |
| --- | --- | --- |
| Linear array | $n-1$ | pipelined comparison-exchange |
| 2-D mesh | $2(\sqrt{n}-1)$ | local data dependencies (PDEs, image processing) |
| Tree | $2\log n$ | broadcast, reduction, divide-and-conquer |
| Hypercube ($2^k$ nodes) | $k = \log n$ | low diameter, each node has $k$ neighbours |

Broadcasting one value costs $O(n)$ on a linear array, $O(\sqrt{n})$ on a mesh, $O(\log n)$ on a hypercube or tree — the same operation, three costs.

## Prefix sum (scan)

Given $x_0, \dots, x_{n-1}$, compute every prefix $s_k = x_0 \oplus \dots \oplus x_k$ for an associative $\oplus$. Sequentially it is $O(n)$ and looks unparallelisable — each prefix depends on the last. But associativity lets you compute it as a balanced tree of partial combinations (an up-sweep of partial sums, then a down-sweep distributing them), giving $O(\log n)$ time on $n/\log n$ EREW processors. Prefix sum is a workhorse: it underlies parallel array compaction, radix sort, and address calculation, so its $O(\log n)$ cost sets a floor for many other algorithms.

## Sorting

### Comparison-exchange on a linear array

**Odd-even transposition sort**: in odd phases, each odd-indexed processor compare-exchanges with its right neighbour; in even phases, each even-indexed one does. After $n$ phases the $n$ elements on $n$ processors are sorted. It is $O(n)$ — no better than sequential in total work, but it maps perfectly onto a linear array with only nearest-neighbour communication.

### Enumeration sort

For each element, count how many others are smaller; that count is its final rank. On an EREW PRAM with $n^2$ processors the counting is $O(\log n)$; with $n$ processors it is slower but still parallel.

### Sorting networks and distributed sorts

**Bitonic sort** is a fixed network of compare-exchange operations that sorts in $O(\log^2 n)$ depth regardless of input — data-independent, so it suits hardware and GPUs. On a mesh of $P = N \times N$ processors, ShearSort and odd-even-merge variants sort $P$ elements in $O(\sqrt{P})$ time. On a hypercube, **hyperquicksort** recursively splits the cube by dimension, partitions and exchanges data across each split, then sorts locally. **Sample sort** generalises: choose $p-1$ splitters, have each processor bucket its local data by them, redistribute so processor $i$ holds global bucket $i$, and sort locally.

## Matrix operations

### Transpose

On an $N \times N$ mesh holding $A[i,j]$ at $P_{i,j}$, transposing means routing each element to $P_{j,i}$ — done in $O(N)$ by moving along rows then columns. On an EREW PRAM with $N^2$ processors, each element writes itself to its transposed address; computing those addresses takes $O(1)$ with concurrent index reads, $O(\log N)$ with a true prefix-sum address calculation.

### Cannon's algorithm

For $C = A \times B$ on an $N \times N$ mesh: skew $A$ left and $B$ up so each processor initially holds a matching pair, then repeat $N$ times — multiply-accumulate locally, circular-shift $A$'s rows left and $B$'s columns up by one. Total time $O(N)$ with only nearest-neighbour communication and no processor ever holding more than one block of each matrix.

### Matrix-vector product

$y = Ax$: distribute $A$ by rows and each processor computes $y_i = \sum_j A_{ij} x_j$ after the vector $x$ is broadcast; or distribute by columns, have each processor compute a partial vector, and sum them with an all-to-one reduction.

## Numerical problems

- **Linear systems** $Ax = b$ — parallel Gaussian elimination (row/column broadcasts and reductions on a mesh or hypercube) for dense systems; iterative Jacobi or Gauss-Seidel relaxation, which parallelise naturally; graph-partitioning approaches for sparse systems.
- **PDEs** — discretise the domain into a grid; each processor owns a subgrid and exchanges only boundary values with its neighbours each step. This maps directly onto a mesh.
- **Monte Carlo** — embarrassingly parallel: each processor runs independent samples. The catch is the **parallel random number generator** — you need many statistically independent, reproducible streams, one per processor.
- **Eigenvalues** — parallelise the matrix operations (multiplications, factorisations) inside the QR or Jacobi iteration.

## Graph algorithms

- **Parallel BFS** — process the graph frontier by frontier; all vertices at the current level expand their neighbours in parallel, then synchronise before the next level.
- **Parallel DFS** — hard, because DFS is inherently sequential (backtracking). Strategies: partition the search tree across processors and use dynamic **work stealing** when one finishes early.
- **Minimum spanning tree** — [Borůvka's algorithm](/citadel/algorithms/MinimumSpanningTree) is naturally parallel: every component simultaneously picks its cheapest outgoing edge, all chosen edges are added, components merge; repeat. Prim's and Kruskal's parallelise less cleanly.
- **Connected components** — iterative merging with pointer jumping on a forest representation.
- **Transitive closure** — repeated squaring of the adjacency matrix, or a parallel [Floyd-Warshall](/citadel/algorithms/PathFinding); on a SIMD machine with $n^2$ processors this is efficient.
- **Shortest paths** — parallel [Dijkstra](/citadel/algorithms/PathFinding) partitions vertices across processors, each tracking its own tentative distances; processors synchronise to find the global minimum-distance vertex and send update messages when neighbours live elsewhere. The global priority queue is the bottleneck.
- **Game tree search** — alpha-beta pruning parallelises awkwardly: pruning decisions in one branch depend on results from another, so naive distribution does redundant work.

## Speedup anomalies in search

Parallel search can be strange. **Superlinear speedup** ($S_p > p$) happens when splitting the search space lets one processor stumble onto the solution far sooner than the sequential order would have reached it. The reverse, **deceleration**, happens when parallelisation causes redundant exploration or wastes work on branches the sequential algorithm would have pruned. Both mean speedup for search is workload-dependent in a way it is not for sorting or matrix multiply.

## The one idea to keep

A parallel algorithm is designed against a model — PRAM for shared memory, a specific topology for distributed memory — and the model's access and communication rules decide what "fast" costs. A small kit of primitives recurs everywhere: prefix sum for anything cumulative, compare-exchange for sorting, nearest-neighbour exchange for grids, and cheapest-edge selection for graphs. Rebuilding a sequential algorithm around those primitives is the actual work; adding processors to the original is not.
