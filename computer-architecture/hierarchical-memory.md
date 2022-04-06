---
title: The Memory Hierarchy - Locality, Caches, and Virtual Memory
description: Why a hierarchy of memories acts like one large fast memory - locality of reference, cache mapping and the three kinds of miss, write policies and replacement, the techniques that cut miss rate and miss penalty, and virtual memory with page tables and the TLB.
date: 2022-04-06
draft: false
slug: /computer-architecture/hierarchical-memory
tags:
  - Computer Architecture
  - Memory
  - Caching
---

The fastest memory is small and expensive; the big memory is slow and cheap. A processor wants both. The trick that squares this is a stack of memories — registers, cache, main memory, disk — arranged so that *almost every access* is served by a fast upper level, and the slow levels are only touched when they have to be.

That works because programs don't reach into memory at random. This post is the reason they don't (locality), the machinery that exploits it in hardware (caches — how blocks are placed, evicted, and written), the techniques that push the hit rate higher, and the same idea one level down: virtual memory, where main memory is a cache for the disk. The [memory technology](/citadel/computer-architecture/memory-system) — SRAM, DRAM, DDR — is a separate post; this one is about how the levels cooperate.

## Locality of reference

Two empirical facts about how programs touch memory:

- **Temporal locality** — a location just accessed is likely to be accessed again soon. Loop counters, a function's local variables, a hot data structure.
- **Spatial locality** — a location near one just accessed is likely to be accessed soon. Sequential instructions, walking an array, fields of a struct.

A hierarchy exploits both by pulling a whole neighbourhood (a *block*) into a fast level on first use and keeping it there while it stays warm.

Two structural properties keep the levels consistent:

- **Inclusion** — the contents of a level are a subset of the level below it: anything in L1 is also in L2, anything in L2 is in main memory. It keeps management simple. Some designs deliberately break it (**multilevel exclusion**: a block is in L1 *or* L2, never both) to use the combined capacity better, at the cost of more data movement.
- **Coherence** — when a datum is copied into several places, the copies must eventually agree. If a DMA transfer overwrites a block in main memory, any cached copy is now stale and its valid bit must be cleared; if a write-back cache holds a *dirty* copy, memory and cache have genuinely diverged until the write-back happens. In a multiprocessor, every core's cache can hold the same block, and keeping them coherent needs a protocol — covered in [multiprocessors](/citadel/computer-architecture/multiprocessor).

## How a cache works

The **cache** is a small SRAM between the processor and main memory. On each access the processor checks it first:

- **Hit** — the data is there, returned fast.
- **Miss** — a **block** (the requested word plus its neighbours, e.g. 16 words) is copied from main memory into a cache **line**, and the word is forwarded. *Load-through* (early restart) forwards the requested word the instant it arrives, before the rest of the block lands.

Each line carries a **valid bit** (cleared at power-on, set when a block is loaded) and, if the cache can hold modified data, a **dirty bit**.

### Where does a block go? Mapping functions

Take a concrete layout: a 2K-word cache organised as 128 lines of 16 words, backing a 64K-word main memory of 4K blocks of 16 words. A 16-bit address splits into a 4-bit **word offset** within a block and a 12-bit block number — and how that block number is used defines the mapping.

**Direct mapping.** Block $j$ goes only to line $j \bmod 128$. The low 7 bits of the block number are the line index; the top 5 bits are stored in the line as a **tag**. A lookup indexes the line, compares its stored tag with the address's tag, and checks the valid bit. Cheap and simple — but two hot blocks that map to the same line evict each other repeatedly (*conflict* / thrashing misses) even while other lines sit empty.

**Fully associative mapping.** A block can go in *any* line. There's no index — the whole 12-bit block number is the tag, and a lookup compares it against *every* line's tag in parallel. No conflict misses, best utilisation, but the parallel comparators are expensive, so it's only practical for small caches (like a TLB).

**Set-associative mapping.** The compromise. The cache is $v$ **sets** of $k$ lines each — a *$k$-way* cache. Block $j$ maps to set $j \bmod v$ and may occupy any of the $k$ lines in it; a lookup selects the set and compares $k$ tags. A 2-way cache with 128 lines has 64 sets of 2. Most conflict misses of direct mapping disappear at $k = 2$ or 4, for far less hardware than full associativity.

### Writes

On a **write hit**:

- **Write-through** — write the cache *and* main memory. Simple; the processor needn't stall if a **write buffer** absorbs the pending writes.
- **Write-back** — write only the cache and set the dirty bit; flush to memory when the line is evicted. Fewer memory writes when the same line is written repeatedly; more complex.

On a **write miss**:

- **Write-allocate** — fetch the block into the cache (like a read miss), then write it. Pairs naturally with write-back.
- **No-write-allocate** (write-around) — write straight to memory, don't disturb the cache. Pairs with write-through.

### The three kinds of miss

- **Compulsory** (cold) — the first-ever reference to a block. Unavoidable in principle; softened by prefetching and larger blocks.
- **Capacity** — the working set doesn't fit, so blocks are evicted and later re-fetched.
- **Conflict** — too many active blocks map to the same set, and one evicts another that's still needed. Only happens below full associativity.

## Cutting misses and miss penalty

The average time per access is captured by **AMAT** (average memory access time):

$$\text{AMAT} = \text{HitTime} + \text{MissRate} \times \text{MissPenalty}$$

and for two levels of cache:

$$\text{AMAT} = \text{HitTime}_{L1} + \text{MissRate}_{L1} \times (\text{HitTime}_{L2} + \text{MissRate}_{L2} \times \text{MissPenalty}_{L2})$$

Each term is a lever:

**Lower miss rate.**

- *Larger block size* — fewer compulsory misses, better use of spatial locality; but past a point the miss penalty grows (more to transfer), unused words pollute the cache, and conflict misses rise.
- *Larger cache* — fewer capacity misses; costs money, area, power, and a little hit time.
- *Higher associativity* — fewer conflict misses; more tag comparisons, so more hit time.
- *Prefetching* — bring blocks in before they're asked for. *Hardware* prefetchers detect sequential or fixed-stride patterns; *software* prefetch instructions are inserted by the compiler or programmer where the access pattern is known.
- *Compiler layout* — loop interchange, blocking, and struct reordering to make the access pattern cache-friendly (see [code optimisation](/citadel/compiler-design/code-optimisation)).

**Lower miss penalty.**

- *Multilevel caches* — a fast small L1 for hit time, a larger L2 to catch most L1 misses so L1's *effective* penalty is an L2 hit, an L3 behind that.
- *Critical word first / early restart* — deliver the requested word immediately, fill the rest of the line after.
- *Victim cache* — a small fully-associative buffer between L1 and L2 holding recently evicted lines; a hit there is much cheaper than going to L2, and it specifically recovers conflict-miss victims.
- *Write buffers* — hide the latency of write-through stores and write-back flushes.
- *Lockup-free (non-blocking) cache* — keeps serving hits, and other misses, while one miss is outstanding.

**Lower hit time.** Keep L1 small and simple — direct-mapped or low-associativity — so tag comparison and indexing are quick.

## Replacement policies

When a set is full and a new block arrives, one line must go. Direct mapping has no choice; associative caches (and, one level down, page frames) pick a victim by policy:

- **LRU** (least recently used) — evict the line unused for the longest. Follows temporal locality and performs well, but tracking exact recency is costly at high associativity, so hardware approximates it.
- **FIFO** — evict the oldest-loaded line. Simpler; can throw out a still-hot block loaded long ago.
- **Random** — pick one at random. Trivial, and often within a few percent of LRU.
- **LFU** (least frequently used) — evict the least-accessed line; needs a counter per line.
- **Clock / second chance** — an LRU approximation for pages. Each frame has a use bit; a circular scan clears a set bit (granting a "second chance") and evicts the first frame it finds with the bit already clear.

Evicting a dirty line or page means writing it back first, which adds to the penalty.

## Virtual memory

The same idea, one level down: **virtual memory** lets a program use an address space larger than physical RAM, keeping only the active parts in memory and the rest on disk.

The CPU issues **virtual addresses**; a hardware **memory management unit (MMU)**, with the OS, translates them to **physical addresses**. Both spaces are divided into fixed-size **pages** / **page frames** (commonly 4 KB or 8 KB), and data moves between disk and memory a page at a time.

A **page table** — in main memory, pointed to by the **page table base register (PTBR)** — maps virtual pages to frames. A virtual address splits into a **virtual page number (VPN)** and an **offset**; the VPN indexes the table to a **page table entry (PTE)** holding:

- the **physical frame number**, if the page is resident;
- a **valid / present bit** — is it in memory at all?
- a **modified / dirty bit** — must it be written back before its frame is reused?
- **protection bits** — read / write / execute permissions.

Consulting a table in memory on *every* access would double memory traffic, so the MMU caches recent PTEs in a small fully-associative **translation lookaside buffer (TLB)**. Translation goes:

1. CPU issues a virtual address; MMU splits it into VPN and offset.
2. MMU checks the TLB for the VPN. **Hit** → frame number in hand. **Miss** → use PTBR + VPN to fetch the PTE from the page table in memory, load it into the TLB.
3. If the PTE's valid bit is 0, raise a **page fault**.
4. Otherwise combine frame number and offset into the physical address and access memory.

The TLB must stay consistent with the page table: when the OS changes a PTE, it invalidates the matching TLB entry.

### Page fault handling

A page fault is an exception the OS services:

1. Suspend the faulting process; find the virtual address that faulted.
2. Check it's a legal address and the access respects the protection bits — otherwise it's a protection fault (segmentation fault).
3. Locate the page on disk.
4. Find a free frame; if none, pick a **victim** page by a replacement policy (LRU approximation, clock).
5. If the victim is dirty, write it to disk.
6. Read the required page into the frame.
7. Update the PTE (valid bit, frame number, clear dirty); invalidate any stale TLB entry for the victim.
8. Restart the instruction that faulted.

The OS owns frame allocation, fault handling, page-table maintenance, and protection; the [operating-system view](/citadel/operating-system/memory-management) covers page replacement algorithms and working-set management in depth.

## The one idea to keep

A hierarchy turns "small and fast" plus "big and slow" into "big and fast on average", and it works only because programs have locality. Every design knob — block size, associativity, prefetch, multilevel, victim caches — is aimed at one of the two factors in AMAT: how often you miss, and how much a miss costs. Virtual memory is the same bet made against the disk, with the page fault as its (very expensive) miss.
