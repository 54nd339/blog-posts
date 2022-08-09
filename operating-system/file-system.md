---
title: File Systems - Naming, Organizing, and Storing Files
description: "How the OS turns a disk of blocks into named files - file attributes and operations, access methods, the directory structures from single-level to acyclic graphs, then implementation: on-disk and in-memory structures, the VFS layer, block allocation methods, free-space management, journaling, RAID, and memory-mapped files."
date: 2022-08-09
draft: false
slug: /operating-system/file-system
tags:
  - Operating Systems
  - File Systems
  - Memory Management
---

A disk is an array of numbered blocks. A **file system** is the [OS](/citadel/operating-system/os) layer that turns it into something usable: named files, organised in directories, with permissions, that survive a crash. This post covers the file abstraction it presents, how directories are structured, and how all of it is implemented on top of raw blocks.

## What a file is

To the OS a file is the basic **logical storage unit** — a named sequence of bytes on non-volatile media. Beyond the **name**, the file system stores **attributes** (metadata), typically in a directory entry:

- **Identifier** — a unique internal tag.
- **Type** — executable, text, image (often from an extension).
- **Location** — pointers to the data blocks on the device.
- **Size**, **protection** (read/write/execute permissions), **timestamps** (created, modified, accessed), **owner and group**.

The internal **structure** may be none (a byte stream, as in UNIX — the application interprets it), a simple record structure (fixed or variable-length lines), or a complex format the OS partly understands (executables).

Files are an abstract data type with standard **operations**: create, write (at the write pointer), read (at the read pointer), reposition/seek, delete, truncate (clear contents, keep attributes), open, close. **Open** finds the file, loads its metadata into a system-wide **open-file table**, and returns a handle; the system tracks the open count, per-process read/write pointers, and access rights. **Close** flushes metadata and releases the per-process entry.

## Access methods

- **Sequential access** — process the file in order, byte after byte or record after record; a read advances the pointer, a write appends. The default, used by editors and compilers.
- **Direct (relative) access** — treat the file as numbered fixed-size blocks and read or write block $n$ directly. Essential for databases. Operations are `read n`, `write n`, `position to n`.
- **Indexed sequential access** — keep an **index** of key values pointing at blocks; search the index for the block holding a record, then access it directly. Combines sequential processing with keyed lookup (IBM's ISAM).

## Directories

A **directory** is a special file listing other files (and directories). Operations: search, create a file, delete a file, list, rename, traverse.

Structures, in order of capability:

- **Single-level** — one directory for the whole volume. Name collisions and no grouping; impractical past a few files.
- **Two-level** — a private directory per user, addressed as `/user/file`. Fixes cross-user name collisions, but no per-user grouping and awkward sharing.
- **Tree-structured** — directories contain subdirectories, forming a hierarchy. Arbitrary grouping, efficient search, a **current working directory**, and **absolute** (from `/`) or **relative** pathnames. This is the standard. Sharing across branches is still awkward, and deleting a directory means deleting its subtree.
- **Acyclic-graph** — allow **links** so one file or subdirectory appears in several places. Enables sharing, but brings **aliasing** (multiple names for one object): deleting the original leaves **dangling** links, so the system needs **reference counts** or backpointers to delete safely, and must avoid creating cycles.
- **General-graph** — links may form **cycles**. Traversal can loop forever, so it needs cycle detection at link time or garbage collection. Most systems dodge this by only allowing links to files, or by using symbolic links that resolve lazily.

## File sharing

Multiple users or processes sharing files need:

- **Directory support** — acyclic- or general-graph structures via links.
- **Mounting** — attach a file system from another device onto a **mount point** directory, so it appears as a subtree of the existing hierarchy.
- **Access control** — [protection](/citadel/operating-system/protection) mechanisms decide who shares what and how.
- **Locking** — a **shared lock** (many readers) or an **exclusive lock** (one writer). **Mandatory** locking is enforced by the OS; **advisory** locking only reports status, and cooperating processes must check it.

## Implementation

### Layered structure

The file system is built in layers, top to bottom: the **logical file system** (metadata, directory structure, protection) calls the **file-organization module** (logical-to-physical block mapping, free space), which calls the **basic file system** (generic block read/write commands), which calls **I/O control** — the [device drivers](/citadel/operating-system/io-management) that talk to the hardware.

### On-disk structures

- **Boot control block** — code to boot an OS from this volume, if any.
- **Volume control block (superblock)** — block size, block count, free-block count and pointers.
- **Directory structure** — names and pointers to file metadata.
- **File control block (FCB) / inode** — one file's metadata: permissions, size, timestamps, and the locations of its data blocks.

### In-memory structures

Held in RAM for speed: the mount table, a directory cache, the system-wide open-file table, a per-process open-file table, and a **buffer cache** for file data.

### Virtual File System

The **VFS** is a kernel abstraction layer that lets one OS support many concrete file-system types (ext4, NTFS, FAT, network file systems) behind one API, using generic objects like `vnodes` to represent files.

### Directory implementation

- **Linear list** of name/pointer pairs — simple, slow search.
- **Hash table** on the file name — fast, needs collision handling and resizing. Large directories often use a [B-tree](/citadel/data-structures/b-tree) instead.

### Allocation methods

How data blocks are assigned to a file:

- **Contiguous** — adjacent blocks. Fast sequential and random access; external fragmentation; hard to grow a file.
- **Linked** — each block points to the next. No external fragmentation, files grow freely, but random access is slow and pointers cost space. **FAT** is a variant keeping all the links in one central table.
- **Indexed** — a per-file **index block** holds pointers to all data blocks. Direct access, no external fragmentation, at the cost of the index block. Large files need multilevel index blocks or a combined scheme — the **UNIX inode** has a few **direct** pointers plus **single**, **double**, and **triple indirect** pointers, so small files are cheap and huge files are still addressable.

### Free-space management

- **Bit vector** — one bit per block (free/used). Simple, finds contiguous runs easily, but large for big disks.
- **Linked list** of free blocks — no map overhead, but finding contiguous space is hard.
- **Grouping** — a free block stores the addresses of several other free blocks.
- **Counting** — store the address of a run's first block plus its length. Efficient when allocation and freeing happen in runs.

### Efficiency, performance, recovery

Performance comes from caching (buffer cache, page cache), **read-ahead** (prefetch sequential blocks), and **free-behind** (drop blocks already read in a sequential scan), plus keeping metadata physically near its data. Against crashes:

- **Consistency checking** — a tool like `fsck` scans metadata for inconsistencies and repairs them. Thorough but slow.
- **Backups** — periodic copies to other media.
- **Journaling** — record pending metadata changes to a log *before* applying them; after a crash, replay the log for a fast, consistent recovery instead of a full scan.

## RAID

**Redundant Array of Independent Disks** combines physical drives into one logical unit for performance, reliability, or both:

| Level | Layout | Redundancy | Notes |
| --- | --- | --- | --- |
| RAID 0 | striping | none | parallel throughput; one failure loses everything |
| RAID 1 | mirroring | 1 disk | survives a failure; 50% capacity |
| RAID 5 | striping + distributed parity | 1 disk | good balance; needs ≥ 3 disks |
| RAID 6 | striping + double parity | 2 disks | survives two failures; needs ≥ 4 disks |
| RAID 1+0 | striped mirrors | 1 per mirror | good performance and redundancy |

RAID can be a hardware controller card or an OS software layer.

## Memory-mapped files

Mapping a file into a process's virtual address space lets it access the contents through pointers, as if the file were an array, instead of `read()` and `write()`. It runs on the [demand-paging](/citadel/operating-system/memory-management) machinery: touching a mapped address faults the corresponding file block into a frame, and writes are flushed back to disk by the OS. It cuts system-call overhead, uses the page cache directly (no double buffering), and lets several processes share file data by mapping the same file.

## The one idea to keep

A file system is a naming layer and an allocation layer stacked on a block device. The naming layer — directories, paths, links — is about how humans and programs find data; the allocation layer — inodes, indirect blocks, free-space maps — is about turning "file X, byte 40000" into "block 913". Journaling exists because those two layers must stay consistent with each other across a power failure that can strike between any two block writes.
