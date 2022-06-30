---
title: Memory Management - Partitions, Paging, and Virtual Memory
description: How the OS shares RAM among processes - contiguous allocation and its fragmentation, paging with page tables and the TLB, segmentation, then virtual memory by demand paging with page-fault handling, page-replacement algorithms from FIFO to LRU approximations, frame allocation, and thrashing.
date: 2022-06-30
draft: false
slug: /operating-system/memory-management
tags:
  - Operating Systems
  - Memory Management
  - Processes
---

A program has to be in RAM to run, RAM is finite, and several [processes](/citadel/operating-system/process-thread) want it at once. The OS's **memory manager** decides who gets which bytes, translates the addresses a program uses into physical ones, and — through virtual memory — lets programs believe they have more memory than the machine physically holds.

This post traces the techniques in the order they developed: contiguous allocation and why it fragments, paging and segmentation as the fix, then virtual memory built on demand paging, with the page-replacement and allocation problems it creates.

## Contiguous allocation

Early schemes kept each process in one unbroken block of memory.

- **Fixed partitions.** Divide memory into fixed-size regions, one process per region. Simple, but a process smaller than its partition wastes the rest — **internal fragmentation** — and the partition sizes rarely match program needs.
- **Variable partitions.** Carve out a block exactly the size each process needs. When a process exits, its block becomes a **hole**. Allocating a new process to a hole:
  - **First-fit** — the first hole big enough. Fast.
  - **Best-fit** — the smallest hole big enough. Leaves tiny unusable holes; needs a full scan.
  - **Worst-fit** — the largest hole. Leaves a big remainder; also a full scan. Generally the worst on both time and fragmentation.

Over time, free memory scatters into many small holes — **external fragmentation** — where the total free space suffices but no single hole does. **Compaction** shuffles processes together to merge free space, but it is slow and needs relocatable processes.

**Base and limit registers** make relocation and protection cheap: the **MMU** adds the base register to every logical address to get the physical one, and traps any address that exceeds the limit. A process can then be loaded anywhere, and it cannot touch memory outside its bounds.

## Paging

Paging drops the requirement that a process's physical memory be contiguous. Split physical memory into fixed-size **frames** and each process's logical address space into **pages** of the same size; any page can go in any free frame.

A logical address splits into a **page number** $p$ and a **page offset** $d$. The page number indexes the process's **page table**, whose entry gives the **frame number**; combine that with the offset for the physical address. Hardware provides a **page-table base register** pointing at the table and a **page-table length register** for its size.

Paging kills external fragmentation entirely — any $N$ free frames hold an $N$-page process — and lets processes share code pages. It still has **internal fragmentation** in the last partial page, and every memory reference now needs an extra memory access to read the page table.

### The TLB

The extra access is removed by a **translation look-aside buffer** — a small, fast associative cache of recent page-to-frame translations. On a reference the MMU checks the TLB first; a **hit** returns the frame immediately, a **miss** falls back to the page table and caches the result. With a 99% hit ratio the average access time is close to a single memory access.

### Big address spaces

A flat page table for a 32- or 64-bit space is enormous, so:

- **Hierarchical paging** — page the page table itself (two- or three-level).
- **Hashed page tables** — hash the virtual page number into a table with chained entries. Good for sparse spaces.
- **Inverted page tables** — one entry per *physical frame*, recording which process and virtual page occupies it. Small table, slower lookup (usually helped by hashing).

## Segmentation

Segmentation matches the programmer's view: a program is a set of logical **segments** — code, stack, heap, a symbol table — of varying size. A logical address is `<segment-number, offset>`; the segment number indexes a **segment table** giving the segment's **base** and **limit**, the offset is checked against the limit and added to the base.

Segmentation makes per-segment protection and sharing natural (share a code segment, protect a data segment) and lets segments grow. But variable-size segments bring back **external fragmentation**. Some architectures (Intel IA-32) combine both, paging each segment.

## Virtual memory

**Virtual memory** separates the logical address space a program sees from physical RAM, and lets the former be much larger. The insight: a program does not need all of itself in memory at once, only the parts it is currently touching. The rest lives on a **backing store** (disk).

This means programs are not capped by RAM, more of them run at once (each using less physical memory), and less I/O is spent loading whole processes. It is almost always implemented as **demand paging**.

### Demand paging

A **lazy pager** brings a page into a frame only when it is first accessed. Each page-table entry has a **valid-invalid bit**: `valid` means the page is in a frame, `invalid` means it is outside the address space *or* it is on the backing store.

Accessing an `invalid` page that is legitimately on disk raises a **page fault**:

1. The MMU traps to the OS.
2. The OS checks whether the address is legal (terminate the process if not) or just not resident.
3. It locates the page on the backing store.
4. It finds a **free frame** — running **page replacement** if none is free.
5. It schedules the disk read into that frame.
6. While the (slow) I/O runs, the CPU switches to another process.
7. On the completion interrupt, the OS updates the page-table entry to `valid` with the frame number.
8. It restarts the faulting instruction, which now proceeds normally.

Performance hinges on the fault rate $p$:

$$\text{EAT} = (1 - p)\cdot t_{mem} + p \cdot t_{fault}.$$

Because $t_{fault}$ (a disk I/O, milliseconds) dwarfs $t_{mem}$ (nanoseconds), even a tiny $p$ hurts, so keeping $p$ minuscule is the whole game.

**Copy-on-write** optimises `fork()`: parent and child share all pages read-only, and only a page that one of them *writes* is copied.

### Swap space

The backing store for demand paging is **swap space** — either a dedicated disk partition (faster) or a file in the regular file system. The OS tracks which swap blocks are free and allocates/frees them as pages move out and back. Because swap traffic is on the demand-paging critical path, its layout is tuned for throughput.

## Page replacement

When a fault needs a frame and none is free, the OS picks a **victim**, writes it back if its **dirty bit** is set, and loads the new page in its place. The replacement algorithm tries to evict the page least likely to be needed soon.

- **FIFO** — evict the oldest-loaded page. Simple (a queue), often poor, and prone to **Belady's anomaly**: adding frames can *raise* the fault rate.
- **Optimal (OPT)** — evict the page not used for the longest time *in the future*. Lowest possible fault rate, no Belady's anomaly, and impossible to implement — it needs the future. It is the benchmark other algorithms are measured against.
- **LRU** — evict the page unused for the longest time *in the past*, using recent history as a proxy for the near future (the **locality of reference** principle). Performs well, no Belady's anomaly. Implemented with per-entry timestamps (search for the oldest) or a stack (move referenced page to top). Both add overhead.
- **LRU approximations** — because true LRU is expensive:
  - **Reference-bit** — hardware sets a bit on access; the OS periodically clears them and evicts a page whose bit is 0. Knows *whether*, not *when*.
  - **Second-chance (clock)** — FIFO around a circular queue with a pointer; if the pointed page's reference bit is 1, clear it and advance (a second chance), else evict.
  - **Enhanced second-chance** — order by the pair `(reference, modify)`, preferring to evict `(0,0)` before `(0,1)` before `(1,0)` before `(1,1)`, so a clean page goes before a dirty one.
- **Counting** — **LFU** evicts the least-referenced page; **MFU** evicts the most-referenced, on the theory that a low count means a page just arrived. Both uncommon.
- **Page buffering** — keep a pool of free frames so a fault is served immediately, and write out the victim later. Cuts fault service time.

## Frame allocation

How many frames does each process get?

- **Minimum** — enough that any single instruction can complete (an instruction may reference operands spanning page boundaries).
- **Fixed allocation** — **equal** ($m/n$ frames each) or **proportional** to process size or priority.
- **Priority allocation** — proportional to priority; a high-priority process may take a frame from a lower-priority one.

**Global replacement** lets a process take a frame from any process — higher throughput, but a process's performance now depends on unrelated ones. **Local replacement** confines a process to its own frames — consistent per-process behaviour, but frames can sit idle in one process while another faults.

## Thrashing

If a process has fewer frames than its active page set, it faults on a page, evicts one it needs immediately, faults again, and so on — spending more time paging than computing. This is **thrashing**, and it drags CPU utilisation down. Worse, an OS seeing low utilisation may admit *more* processes, cutting frames per process further and deepening the thrash.

Three ways to keep a process's frames matched to its needs:

- **Locality model** — a process runs in a **locality**, a set of pages it actively uses; thrashing happens when the summed locality sizes exceed physical frames.
- **Working-set model** — approximate the locality as the distinct pages referenced in the last $\Delta$ references (the **working-set window**). If total demand $D = \sum \text{WSS}_i$ exceeds available frames, suspend a process.
- **Page-fault frequency** — monitor each process's fault rate against an upper and lower bound: above the upper bound, give it more frames; below the lower bound, take some away; high rate with no free frames, suspend a process.

## The one idea to keep

Every layer here is the same move — break a resource into fixed-size units and use a lookup table to place them anywhere — applied first to RAM (pages and frames) and then to the RAM/disk boundary (demand paging). Virtual memory works because programs have locality: they touch a small working set at a time, so most of a process can sit on disk with almost no penalty — right up until the working set no longer fits, when the penalty becomes everything.
