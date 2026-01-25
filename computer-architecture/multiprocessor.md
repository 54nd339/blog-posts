---
title: Beyond Single Cores - A Journey into Multiprocessor Architectures
description: Delving into the world of multiprocessors, from parallel architecture taxonomies and shared-memory systems (synchronization, coherence, interconnects) to distributed memory, clusters, and glimpses of non-von Neumann designs.
date: 2023-05-04
draft: false
slug: /pensieve/coa/multiprocessor
tags:
  - Computer Architecture
  - CS Basics
---

Welcome back, architecture aficionados! We've seen how instruction-level parallelism squeezes performance out of a single instruction stream. But to achieve even greater leaps in computational power, architects design systems with multiple processing units that work together. These **Multiprocessor Architectures** are the backbone of everything from high-performance servers to modern multi-core desktops and even supercomputers. Let's explore their designs and challenges.

## Taxonomy of Parallel Architectures

Before diving into specifics, it's helpful to classify parallel systems. **Flynn's Classification** is a widely used taxonomy based on the number of instruction streams and data streams a computer can process simultaneously:

1.  **SISD (Single Instruction stream, Single Data stream):** This is the traditional uniprocessor model. One instruction operates on one piece of data at a time.
2.  **SIMD (Single Instruction stream, Multiple Data streams):** A single instruction is executed concurrently on many different data elements. This is ideal for data-parallel tasks and is embodied in **array processors** and **vector processors**.
3.  **MISD (Multiple Instruction streams, Single Data stream):** Multiple instructions operate on a single stream of data. This category has very few practical implementations.
4.  **MIMD (Multiple Instruction streams, Multiple Data streams):** Multiple autonomous processors simultaneously execute different instructions on different data. This is the most general and flexible form of parallel processing and encompasses most multiprocessor systems and multicomputers.

MIMD architectures themselves can be further categorized based on how their memory is organized:

* **Shared Memory Multiprocessors:** All processors share access to a single, global physical address space. Communication between processors typically occurs implicitly by reading and writing shared variables in this common memory.
    * **UMA (Uniform Memory Access):** All processors have approximately the same latency to access any location in main memory. **Symmetric Multiprocessors (SMPs)** are a common example, often using a shared bus or a crossbar switch to connect processors to memory. Modern multi-core chips where cores share L2/L3 cache and main memory also fit this model.
    * **NUMA (Non-Uniform Memory Access):** Memory access time depends on the location of the data relative to the processor. Although there's a single address space, memory is physically distributed among processors (each processor has a "local" portion of memory). Accessing local memory is fast, while accessing remote memory (another processor's local memory) is slower due to traversal through an interconnection network.
* **Distributed Memory Multicomputers:** Each processor has its own private address space and cannot directly access another processor's memory. Processors communicate explicitly by sending and receiving messages over an interconnection network. **Cluster computers** are a prime example.

## Centralized Shared-Memory Architecture (SMPs/UMA)

In a centralized shared-memory system, typically an SMP, multiple processors are connected to a single, centrally located main memory, often via a shared bus or a more advanced interconnection network like a crossbar switch. Each processor usually has its own private cache(s) (e.g., L1, sometimes L2) to reduce memory latency and bus traffic.

### Synchronization
When multiple processors share data, **synchronization** is critical to coordinate their activities and prevent data corruption from concurrent access (race conditions). For example, if two processors try to update a shared counter simultaneously, the final value might be incorrect.
* **Need for Atomic Operations:** Operations like incrementing a shared counter need to be **atomic** – appearing as a single, indivisible step. Processors provide special atomic instructions (e.g., test-and-set, fetch-and-add, compare-and-swap) that can read, modify, and write a memory location in one uninterruptible operation. These are used to build higher-level synchronization primitives like locks (mutexes), semaphores, and barriers.

### Memory Consistency (Cache Coherence)
With multiple processors each having their own caches, copies of the same shared data block can exist in several caches simultaneously. If one processor modifies its cached copy, other caches (and main memory) now hold stale data. This is the **cache coherence problem**. Maintaining a consistent view of memory across all processors is vital.

* **Snooping Protocols:** Suitable for bus-based SMPs. Each cache controller "snoops" (monitors) the shared bus for memory transactions initiated by other processors.
    * **Write-Invalidate:** When a processor writes to a shared block in its cache, it broadcasts an invalidation signal on the bus. Other caches holding a copy of that block mark their copy as invalid. Subsequent reads by those processors will miss, forcing them to fetch the updated version. This is the most common approach.
    * **Write-Update (or Write-Broadcast):** When a processor writes to a shared block, it broadcasts the new data on the bus. Other caches holding a copy update their versions with the new data. This can use more bus bandwidth but might reduce latency for subsequent reads from other processors.
    * **Cache States:** Snooping protocols often define states for each cache line (e.g., **Invalid**, **Shared** (clean, multiple copies exist), **Exclusive/Modified** (dirty, this cache has the only valid/most up-to-date copy)). When a bus transaction is snooped, the cache controller transitions the state of its block accordingly and takes action (e.g., provide data if it has the modified copy, invalidate its copy).
* **Directory-Based Protocols:** More scalable for larger systems without a single shared bus (like NUMA systems). A central **directory** structure (associated with main memory) keeps track of the sharing status of each memory block: which block is cached by which processor(s), and whether it's modified.
    * On a read miss, the directory is consulted. If the block is clean and shared, the directory allows memory to supply it. If another processor has it exclusively (dirty), the read request is forwarded to that processor, which supplies the data and possibly updates memory.
    * On a write miss or a write to a shared clean block, the directory sends point-to-point invalidation (or update) messages only to those caches that actually hold a copy, rather than broadcasting to all caches.

### Interconnection Networks
How processors and memory modules are connected significantly impacts performance and scalability.
* **Shared Bus:** Simplest and most common for smaller SMPs. All processors, memory, and I/O share a single bus.
    * **Pros:** Low cost, simple.
    * **Cons:** Becomes a bottleneck as the number of processors increases due to limited bandwidth and contention. Snooping protocols work well here.
* **Crossbar Switch:** Provides a dedicated path between each processor and each memory module. Allows multiple simultaneous transfers if there's no memory module conflict.
    * **Pros:** High bandwidth, low latency for non-conflicting accesses.
    * **Cons:** Very complex and expensive, especially for a large number of processors/memory modules (cost grows as $N \times M$).
* **Multiport Memories:** Memory modules have multiple independent access ports, allowing several processors to access the same module simultaneously (as long as they target different internal banks or addresses).
* **Multistage Interconnection Networks (MINs):** Composed of smaller switching elements arranged in stages (e.g., Omega network, Butterfly network). Offer a compromise between the cost of a crossbar and the limited bandwidth of a bus. Latency is typically higher than a crossbar.

**(Question:** Why are snooping protocols less suitable for very large shared-memory systems?)
*Answer: Snooping requires broadcasting all memory requests to all cache controllers, which overwhelms a single bus and doesn't scale well. Directory protocols use point-to-point messages, making them more scalable.*

## Distributed Shared-Memory (DSM) Architecture (NUMA)

In a DSM system, memory is physically distributed among the processors (each processor has a "local" memory module), but the system presents a single, global address space to all processors. This is a hallmark of NUMA architectures.
* A processor can access its own local memory quickly.
* Accessing memory physically located with another processor (remote memory) incurs higher latency as the request and data must traverse an interconnection network.
* **Cache coherence** is typically maintained using directory-based protocols, as there isn't usually a single bus for all processors to snoop.
* **Advantages:** More scalable than centralized shared-memory systems because memory bandwidth scales with the number of processors.
* **Challenges:** Performance is highly dependent on **data locality**. Programs must be optimized to minimize remote memory accesses. The OS and hardware may try to migrate data closer to the processors that frequently use it.

## Cluster Computers

A **cluster** is a collection of interconnected, typically commodity, standalone computers (nodes – e.g., PCs, workstations, or servers) that work together as a single, integrated computing resource.
* **Architecture:** Each node is a complete computer with its own processor(s), memory, and OS instance. Nodes are connected via a high-speed network (e.g., Gigabit Ethernet, InfiniBand).
* **Memory Model:** Fundamentally a distributed memory architecture. There is no hardware-supported global address space spanning all nodes.
* **Communication:** Inter-node communication relies on explicit **message passing** libraries like MPI (Message Passing Interface) or PVM (Parallel Virtual Machine).
* **Advantages:**
    * **Scalability:** Can scale to hundreds or thousands of nodes.
    * **Fault Tolerance:** If one node fails, the rest of the cluster can often continue operating (though the running application might need specific support for this).
    * **Cost-Effectiveness:** Built using readily available, often inexpensive, components.
* Software DSM layers can sometimes be implemented on top of clusters to provide a shared-memory programming model, but this typically incurs significant performance overhead.

## Non von Neumann Architectures

While the von Neumann architecture (stored program, sequential instruction fetch) dominates computing, alternative models have been proposed, primarily for specialized parallel processing:

### 1. Data Flow Computers
* **Concept:** Operates on the principle of **data availability**. An instruction (or operation) is ready to execute as soon as all its required input operands (data tokens) arrive. There's no program counter.
* **Execution:** Instructions are represented as nodes in a data flow graph. Data tokens flow along arcs. When a node receives all its input tokens, it "fires" (executes) and produces output tokens that are sent to subsequent nodes.
* **Parallelism:** Inherent parallelism is exposed as multiple nodes can fire concurrently if their data is available.
* **Challenges:** Efficiently managing and matching data tokens, handling data structures, and managing the "explosion" of parallelism can be difficult. Practical general-purpose data flow machines have been elusive, but the concepts influence some stream processing and parallel programming models.

### 2. Reduction Computer Architectures
* **Concept:** Based on functional programming principles. A program is viewed as an application of functions to values. Execution proceeds by **reducing** expressions to their simplest form (i.e., their values). This is often **demand-driven**: an expression is evaluated only when its result is needed.
* **Example:** To evaluate `( (2+3) * (4-1) )`, the `+` and `-` operations might be reduced in parallel, followed by the `*` operation.
* **Parallelism:** Achieved by concurrently reducing independent sub-expressions.
* **Challenges:** Efficiently managing the reduction process, handling shared sub-expressions, and garbage collection. String reduction and graph reduction are common implementation techniques.

### 3. Systolic Architectures
* **Concept:** A regular array of simple, identical processing elements (PEs or cells) where data flows rhythmically from cell to cell in a pipelined fashion. Each cell performs a small computation on the data passing through it and then passes the result to its neighbor.
* **Operation:** Think of it like the heart pumping blood (data) through a network of vessels (cells), with each cell doing a bit of work. Data is typically pumped in from the edges of the array and results emerge from other edges.
* **Characteristics:** High degree of pipelining and parallelism, regular structure making them suitable for VLSI implementation, fixed interconnection patterns.
* **Applications:** Well-suited for specific, compute-intensive tasks with regular data flow, such as matrix multiplication, signal processing (e.g., FIR/IIR filters), image processing, and dynamic programming algorithms.
* **Challenges:** Not general-purpose; designing the specific systolic array for a given algorithm can be complex.

**(Discussion Point:** Data flow architectures seem inherently parallel. Why do you think they haven't become mainstream for general-purpose computing?)

## Conclusion: The Expanding Universe of Parallelism

The world of multiprocessor architectures is diverse and constantly evolving. From tightly coupled shared-memory systems that require sophisticated coherence and synchronization mechanisms, to loosely coupled clusters leveraging message passing, each design offers different trade-offs in terms of scalability, programmability, and cost. Beyond the dominant von Neumann paradigm, specialized architectures like data flow, reduction, and systolic systems showcase alternative ways to harness parallelism for specific problem domains. The continuous quest for more computational power ensures that research and innovation in multiprocessor design will remain a vibrant field.

**Key Takeaways:**

* Flynn's Classification (SISD, SIMD, MISD, MIMD) provides a broad taxonomy for parallel architectures.
* Centralized shared-memory (UMA/SMP) systems provide a single address space but require careful synchronization and cache coherence (snooping or directory-based).
* Distributed shared-memory (NUMA) offers scalability with a shared address space but makes data locality crucial.
* Cluster computers use message passing between independent nodes for high scalability and cost-effectiveness.
* Non von Neumann architectures like data flow, reduction, and systolic systems offer alternative models for parallel computation, often tailored for specific tasks.
