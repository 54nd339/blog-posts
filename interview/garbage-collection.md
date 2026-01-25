---
title: The Unseen Janitor - A Deep Dive into Garbage Collectors and Their Algorithms
description: Ever wondered how languages like Java, C#, Python, or Go manage memory automatically? We explore Garbage Collection (GC), its core concepts (heap, reachability, stop-the-world), and common algorithms like Mark-Sweep, Copying, Generational, and G1.
date: 2024-08-24
draft: true
slug: /pensieve/garbage-collection
tags:
  - interview
  - SDE
---

Hey everyone, and welcome back to the blog! As software developers, especially those of us working with modern languages like Java, C#, Python, Go, or JavaScript, we often take automatic memory management for granted. We create objects, use them, and then... they just disappear when we're done, right? Well, not quite by magic! Behind this convenience is a sophisticated process known as **Garbage Collection (GC)**.

The Garbage Collector is like an unsung hero, a diligent janitor working tirelessly in the background of our applications, cleaning up memory that's no longer needed. This frees us from the often error-prone task of manual memory deallocation (think `malloc`/`free` in C/C++), preventing many common bugs like memory leaks and dangling pointers. But how does this automatic memory janitor actually work? Let's dive into the world of garbage collection and explore some of its common algorithms.

## What is Garbage Collection? The Automatic Memory Janitor
**Garbage Collection (GC)** is an automatic memory management process that reclaims memory occupied by objects that are no longer in use by the program. These unused objects are referred to as "garbage." The primary goals of GC are:
* To free developers from the burden of manual memory management.
* To prevent memory leaks (where memory is allocated but never released, eventually exhausting system resources).
* To reduce common programming errors associated with manual memory management, like dangling pointers (pointers to deallocated memory) or double frees.

Garbage collectors usually run as a separate thread or process within the language's runtime environment (like the Java Virtual Machine - JVM, or the .NET Common Language Runtime - CLR).

## Core Concepts in Garbage Collection

Before exploring specific algorithms, let's understand some fundamental concepts:

* **Heap:** This is the region of memory where dynamically allocated objects (objects created at runtime using `new` or similar constructs) reside. The garbage collector primarily operates on the heap.
* **Reachability:** This is the fundamental principle most garbage collectors use to identify which objects are still in use ("live") and which are garbage. An object is considered **live** if it is **reachable** from a set of "root" objects.
    * **Roots:** These are the starting points for the GC's object graph traversal. Roots typically include:
        * Local variables and parameters on active thread stacks.
        * Static variables (global variables).
        * Objects currently held in CPU registers.
        * Objects referenced by JNI (Java Native Interface) code.
* **Unreachable Objects (Garbage):** Any object that is no longer reachable from any root, either directly or indirectly through a chain of references, is considered garbage and its memory can be safely reclaimed.
* **"Stop-the-World" Pauses:** Many traditional GC algorithms require pausing all application threads while the collection cycle (identifying garbage and reclaiming memory) runs. This is done to ensure that the GC gets a consistent view of the memory and object relationships. These pauses can affect application responsiveness and are a major focus of optimization in modern GC designs.
* **Generational Hypothesis:** This is a key observation that drives many modern GC strategies: **"most objects die young."** This means that newly created objects have a high probability of becoming garbage very quickly, while objects that have survived for a significant amount of time are likely to continue to survive for much longer.

## A Tour of Common Garbage Collection Algorithms

Different garbage collectors employ various algorithms, each with its own trade-offs in terms of pause times, throughput, memory overhead, and complexity.

### 1. Mark and Sweep
This is one of the oldest and simplest GC algorithms. It works in two main phases:
* **Mark Phase:**
    1.  The GC starts from the root objects.
    2.  It traverses the entire graph of reachable objects (following references from one object to another).
    3.  Every object encountered during this traversal is "marked" as live (e.g., a bit is set in the object's header).
* **Sweep Phase:**
    1.  After the marking phase, the GC scans the entire heap linearly.
    2.  Any object that was not marked during the mark phase is considered garbage.
    3.  The memory occupied by these unmarked objects is reclaimed and added to a "free list" of available memory blocks that can be used for future allocations.
* **Pros:**
    * Relatively simple to implement.
    * Handles cyclic references correctly (objects that refer to each other but are not reachable from roots are still collected).
* **Cons:**
    * **Memory Fragmentation:** After several GC cycles, the heap can become fragmented with many small, non-contiguous free memory blocks. This can make it difficult to allocate larger objects even if there's enough total free memory.
    * **"Stop-the-World" Pauses:** Both the mark and sweep phases typically require pausing all application threads, which can lead to noticeable application freezes. The sweep phase can be slow if the heap is large and has many garbage objects scattered around.

### 2. Mark and Compact (or Mark-Sweep-Compact)
This algorithm addresses the fragmentation issue of Mark and Sweep.
* **How it Works:**
    1.  **Mark Phase:** Same as in Mark and Sweep – identify and mark all live objects.
    2.  **(Sweep Phase - Implicit):** The sweep phase implicitly identifies garbage by knowing what's marked.
    3.  **Compaction Phase:** After marking, all live (marked) objects are moved to one end of the heap, filling in the gaps left by garbage objects. This eliminates fragmentation by creating a large contiguous block of free memory at the other end. Pointers to the moved objects must be updated.
* **Pros:**
    * **Reduces Memory Fragmentation:** Leads to better allocation performance for new objects as large contiguous free blocks are available.
* **Cons:**
    * **Expensive Compaction:** The compaction phase itself is computationally expensive and adds significantly to the "stop-the-world" pause duration.

### 3. Copying Collector (Semi-Space Collector)
This algorithm uses a different approach to manage memory and avoid fragmentation.
* **How it Works:**
    1.  The heap is divided into two equal-sized regions called **semi-spaces**: a **From-space** and a **To-space**.
    2.  All new object allocations happen in the currently active From-space.
    3.  When From-space becomes full, a garbage collection cycle is triggered:
        * The GC traverses the object graph starting from the roots, copying all *live* objects from From-space to the (currently empty) To-space.
        * As objects are copied, they are naturally placed contiguously in To-space, so To-space is automatically compacted.
        * All references (pointers) to these moved objects must be updated to point to their new locations in To-space.
    4.  After all live objects are copied, the From-space (which now contains only garbage or was empty) is considered completely free.
    5.  The roles of From-space and To-space are then **swapped**. The old To-space (now containing live objects) becomes the new From-space for future allocations, and the old From-space (now clear) becomes the new To-space for the next collection.
* **Pros:**
    * **Fast Allocation:** Allocating new objects in From-space can be very fast (often just incrementing a pointer).
    * **No Fragmentation:** The active space (new From-space after a GC) is always compact.
    * GC cycle can be relatively fast if the proportion of live objects is small (as only live objects are copied).
* **Cons:**
    * **Wastes Half the Heap Space:** Only one of the two semi-spaces is usable for object allocation at any given time. This means it requires twice the memory for the same effective heap size.
    * Copying objects can be time-consuming, especially if many objects are live.

### 4. Generational Garbage Collection
This is a very widely used and effective strategy, based on the **generational hypothesis**: "most objects die young."
* **How it Works:**
    1.  The heap is divided into (at least) two **generations**:
        * **Young Generation (or Nursery):** This is where all newly created objects are initially allocated. The Young Generation is often further subdivided into an "Eden" space and two "Survivor" spaces (S0 and S1).
        * **Old Generation (or Tenured Generation):** Objects that survive multiple garbage collection cycles in the Young Generation are "promoted" (moved) to the Old Generation.
    2.  **Minor GC:** Garbage collections occur much more frequently in the Young Generation. These are called Minor GCs and are typically very fast because, according to the generational hypothesis, most objects in the Young Generation are short-lived and quickly become garbage. Young Generation collections often use a fast copying collector (e.g., between Eden and Survivor spaces).
    3.  **Major GC (or Full GC):** Garbage collections in the Old Generation are less frequent. These are called Major GCs or Full GCs and typically involve collecting the entire Old Generation (and sometimes the Young Generation too). Because objects in the Old Generation are expected to live longer, these collections can be more time-consuming and might use algorithms like Mark-Sweep-Compact.
* **Pros:**
    * **Significantly Reduces Average "Stop-the-World" Pause Times:** By focusing most GC effort on the Young Generation (where collections are quick and efficient), overall application throughput and responsiveness are improved.
    * **Efficient:** Exploits the observed lifetime behavior of objects.
* **Cons:**
    * **Increased Complexity:** The GC design is more complex.
    * **Inter-Generational Pointers:** Pointers from objects in the Old Generation to objects in the Young Generation need to be tracked carefully (e.g., using "card tables" or "remembered sets") so that Minor GCs in the Young Generation don't miss live objects referenced from the Old Generation.

### 5. Concurrent Garbage Collection
The goal of concurrent GCs is to perform most of the garbage collection work *concurrently* with the application threads, thereby minimizing or eliminating long "stop-the-world" pauses.
* **How it Works:** Certain phases of the GC cycle, particularly the time-consuming ones like marking reachable objects, are performed while the application threads are still running. This requires sophisticated synchronization mechanisms to correctly handle object graph modifications made by the application during the concurrent GC phase. Short "stop-the-world" pauses might still be necessary for specific critical operations (e.g., initial root scanning, final marking, or certain parts of compaction).
* **Examples:**
    * **CMS (Concurrent Mark Sweep):** An older concurrent collector in Java, primarily focused on low latency for the Old Generation. It had issues with fragmentation and floating garbage.
    * **G1 (Garbage-First):** A more modern collector in Java, designed to be a long-term replacement for CMS.

### 6. G1 (Garbage-First) Collector (Example of a Modern Advanced GC)
The G1 collector is a server-style garbage collector, targeted for multi-processor machines with large memory space, aiming for predictable pause times.
* **How it Works:**
    * **Regionalized Heap:** G1 divides the heap into a large number of small, equal-sized **regions**. Each region can dynamically be an Eden region, a Survivor region, or an Old Generation region. Large objects ("humongous objects") can span multiple regions.
    * **Concurrent Marking:** G1 performs a global concurrent marking phase (similar to CMS) to determine the liveness of objects across the entire heap.
    * **"Garbage-First" Collection:** After marking, G1 knows which regions contain the most garbage. It prioritizes collecting from these "garbage-rich" regions first to reclaim space most efficiently.
    * **Evacuation Pauses (Compaction):** Collection of regions is done via evacuation (copying live objects from a set of candidate regions to new regions). This is typically done during a "stop-the-world" pause, but G1 attempts to meet a user-defined pause time goal by selecting a manageable number of regions to collect in each pause.
* **Pros:**
    * Better predictability of pause times compared to older collectors like CMS.
    * Good balance of throughput and low latency for applications with large heaps.
    * Efficiently handles large heaps with less fragmentation due to compaction during evacuation.
* **Cons:** Can have slightly higher overhead than some other collectors in certain scenarios due to its complexity.

## "Everything Around It": Language Runtimes & GC

* **Which Languages Use GC?** Many popular high-level languages rely on garbage collection, including:
    * Java (via the JVM, which offers various GCs like Serial, Parallel, CMS, G1, ZGC, Shenandoah)
    * C# and other .NET languages (via the .NET CLR's GC)
    * Python (uses reference counting primarily, with a supplemental generational GC for cycles)
    * JavaScript (engines like V8 have sophisticated GCs)
    * Go (has its own concurrent, tri-color mark-and-sweep GC optimized for low latency)
    * Ruby, Lisp, Scala, Kotlin, etc.
* **Tuning GC:** For performance-critical applications, especially in Java or .NET, developers can often tune various GC parameters (heap sizes for generations, pause time goals, choice of GC algorithm) to optimize for specific workload characteristics (e.g., latency-sensitive vs. throughput-oriented).
* **Memory Leaks in GC Languages?** Yes, they are still possible! If your application unintentionally keeps references to objects that are no longer logically needed, the GC will consider them "reachable" and won't reclaim their memory. These are often called "logical memory leaks."

## Key Takeaways

* Garbage Collection is an essential automatic memory management feature in many modern programming languages, simplifying development and improving application reliability by preventing many common memory errors.
* Core GC concepts include the **heap**, **object reachability** from roots, and the challenge of minimizing **"stop-the-world" pauses**.
* Common GC algorithms include **Mark and Sweep** (simple but fragments), **Mark and Compact** (reduces fragmentation but compaction is costly), **Copying Collector** (fast allocation, no fragmentation, but wastes space), and **Generational GC** (exploits "most objects die young" for efficiency).
* Modern advanced collectors like **G1** in Java aim for predictable low pause times on large heaps using concurrent and regionalized collection techniques.

Understanding the basics of how garbage collection works can help developers write more memory-efficient code and better diagnose performance issues related to memory management in their applications.
