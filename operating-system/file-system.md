---
title: File Systems - Your OS's Digital Filing Cabinet!
description: Dive into how Operating Systems manage files, organize directories, provide access methods, enable sharing, and handle the complexities of implementation.
date: 2023-04-09
draft: false
slug: /pensieve/operating-system/file-system
tags:
  - OS
  - CS Basics
---

Hey data organizers! Ever wonder how your computer keeps track of thousands of documents, pictures, applications, and system data, ensuring you can find them again reliably? It's not magic; it's the **File System**, a crucial part of the Operating System that acts like a highly sophisticated digital filing cabinet for all the information stored on devices like hard drives, SSDs, and USB sticks. Let's open the drawers and explore how files are defined, organized, accessed, shared, and how the whole system is implemented.

### The File Concept: What *Is* a File?

From the OS perspective, a file is the fundamental **logical storage unit**. It's a named collection of related information recorded on secondary storage (non-volatile media like disks). To you, it might be a text document, a spreadsheet, a photo, a song, or an application. To the OS, it's typically seen as a sequence of bytes, words, or blocks.

**File Attributes:** Besides its **Name** (the human-readable identifier), the OS stores metadata about each file, often called **attributes**, typically within a directory structure:

* **Identifier:** A unique tag or number identifying the file within the file system.
* **Type:** Information about the file's nature (e.g., executable, text, image), often indicated by an extension.
* **Location:** Pointer(s) to where the file's contents are stored on the physical device.
* **Size:** The current size of the file.
* **Protection:** Access control information (who can read, write, execute).
* **Timestamps:** Dates and times of creation, last modification, last access.
* **User/Group ID:** Information about the file's owner and group for protection purposes.

**File Structure:** While the OS often treats files as simple byte streams, files can have internal structures:

* **None:** Sequence of bytes/words (common in UNIX/Linux). Interpretation is up to the application.
* **Simple Record Structure:** Divided into lines or fixed/variable-length records.
* **Complex Structures:** Formatted documents (like word processor files) or executable files with specific internal layouts. The OS might understand some complex structures, or interpretation might be left entirely to the application.

**File Operations:** Files are an abstract data type, meaning the OS provides standard operations to manipulate them:

* **Create:** Make a new file.
* **Write:** Add data to the file (usually at the current write pointer position).
* **Read:** Retrieve data from the file (usually from the current read pointer position).
* **Reposition (Seek):** Move the current file pointer (read/write position) within the file.
* **Delete:** Remove the file and free its storage space.
* **Truncate:** Erase the file's contents but keep its attributes.
* **Open:** Prepare a file for access. The OS finds the file in the directory, loads its metadata into memory (often in a system-wide **Open File Table**), and returns a handle or file descriptor to the process. The system tracks open counts, per-process read/write pointers, and access rights.
* **Close:** Indicate the process is finished with the file. The OS updates metadata on disk (if needed) and releases the entry from the per-process open file table.

### File Organization and Access Mechanisms: Getting to the Data

How can processes actually read or write the data within a file? The OS provides different **access methods**:

1. **Sequential Access:** The simplest method. Information is processed in order, one byte or record after another, like reading a book or magnetic tape. A read operation advances the file pointer to the next piece of data; a write appends data or overwrites subsequent data. You can typically reset the pointer to the beginning. Most common access method (used by editors, compilers).
2. **Direct Access (or Relative Access):** Views the file as a numbered sequence of fixed-size logical blocks or records. Processes can read or write blocks directly by specifying the block number, without processing sequentially. Think of accessing tracks on a CD or pages in memory. This is crucial for applications like databases that need fast access to specific records. Operations include `read n`, `write n`, `position to n`.
3. **Indexed Sequential Access:** Builds upon sequential access by adding an **index** to the file. The index contains pointers to different blocks or key points within the file. To find a specific record, you first search the index (which might be sequential or structured) to find the block containing the record, and then access that block directly. This combines the ability to process sequentially with faster access to specific records based on a key. (Example: IBM's ISAM).

### File Directories: Organizing the Chaos

Imagine trying to find a specific piece of paper on a desk covered with thousands of unsorted documents! Directories are the OS's solution for organizing files, making them manageable for users and the system. A **directory** is itself a special type of file containing information about *other* files and possibly other directories.

**Directory Operations:** Common operations include:

* Searching for a specific file.
* Creating a new file within the directory.
* Deleting a file from the directory.
* Listing the contents of the directory.
* Renaming a file within the directory.
* Traversing the file system hierarchy.

**Directory Structures (Evolution):**

* **Single-Level Directory:** Simplest form. One directory for *all* files on the volume.
  * *Problem:* Naming collisions (two users can't easily have files with the same name). No logical grouping of files. Impractical for more than a few files.
* **Two-Level Directory:** Each user gets their own private directory (a User File Directory, UFD). Files are accessed via a path like `/User/filename`.
  * *Pros:* Solves the naming collision problem between users.
  * *Cons:* Still no way for a user to group their own files logically. Sharing files between users is cumbersome.
* **Tree-Structured Directory:** The most common structure. Directories can contain files *and* other subdirectories, forming a hierarchical tree.
  * *Pros:* Allows arbitrary logical grouping. Efficient searching. Defines concepts like the **current directory** (working directory) and **pathnames** (absolute, starting from the root `/`, or relative, starting from the current directory).
  * *Cons:* Sharing files or directories between different branches of the tree can be difficult. Deleting a directory might require deleting everything underneath it.
* **Acyclic-Graph Directory:** An extension of the tree structure that allows directories to contain **links** or pointers to other files *or directories*. This enables **sharing** – a file or subdirectory can appear in multiple locations in the directory structure.
  * *Pros:* Flexible sharing.
  * *Problem:* **Aliasing** (multiple names for the same file/directory). If the original file/directory is deleted, links become **dangling pointers**. Requires mechanisms like reference counts or backpointers to manage deletion correctly. Need to avoid cycles.
* **General Graph Directory:** Allows links that can create **cycles** (e.g., directory A contains a link to B, and B contains a link back to A).
  * *Problem:* Traversal algorithms (like listing or searching) can get stuck in infinite loops. Requires cycle detection algorithms during link creation or garbage collection to manage. Most systems prevent cycles by only allowing links to files, not directories, or using specific link types (like symbolic links in Unix).

### File Sharing: Working Together

Modern systems need to allow multiple users or processes to share files.

* **Directory Structure Support:** Acyclic-graph or general graph directory structures inherently support sharing through linking.
* **Mounting:** File systems residing on different devices or partitions need to be integrated into a single logical namespace. **Mounting** attaches one file system (e.g., from a USB drive) onto a specific directory (**mount point**) within another file system. This makes the mounted file system appear as a subdirectory of the existing structure.
* **Access Control:** Protection mechanisms (discussed later) determine *who* can share *what* and in what way (read, write, etc.).
* **Concurrency Control (Locking):** When multiple processes access a *shared* file simultaneously, especially for writing, potential conflicts arise. File systems or the OS may provide **file locking** mechanisms:
  * **Shared Lock:** Multiple processes can hold a shared lock concurrently (like readers).
  * **Exclusive Lock:** Only one process can hold an exclusive lock at a time (like a writer).
  * **Mandatory Locking:** The OS enforces the lock rules, denying access if a conflicting lock is held.
  * **Advisory Locking:** The OS provides lock status information, but it's up to the cooperating processes to check locks and behave correctly.

### Implementation Issues: Under the Hood

Making file systems work efficiently and reliably involves many implementation details:

* **Layered Structure:** File systems are typically implemented in layers. User applications interact with the **Logical File System** (handling metadata, directory structure, protection). This layer translates requests to the **File Organization Module** (managing logical-to-physical block mapping, free space). This module interacts with the **Basic File System** (issuing generic commands to device drivers). The **I/O Control** layer contains the **Device Drivers** that communicate with the hardware controllers.
* **On-Disk Structures:** Essential information stored persistently on the storage volume:
  * **Boot Control Block:** Information needed to boot an OS from this volume (if present).
  * **Volume Control Block (Superblock):** Contains volume details like block size, total blocks, free block count, pointers to free blocks, etc.
  * **Directory Structure:** Stores directory and file names and pointers to their metadata.
  * **File Control Block (FCB) or inode:** Contains metadata for one file (permissions, size, timestamps, location of data blocks).
* **In-Memory Structures:** Caches and tables held in RAM for performance: Mount table, directory cache, system-wide open file table, per-process open file table, buffer cache for file data.
* **Virtual File System (VFS):** An abstraction layer within the kernel allowing the OS to support multiple different concrete file system types (e.g., UFS, NTFS, FAT32, network file systems) simultaneously through a single, clean API. It uses concepts like `vnodes` to represent files generically.
* **Directory Implementation:** Common methods include:
  * **Linear List:** Simple list of names and pointers. Slow search.
  * **Hash Table:** Uses a hash function on the file name to find the entry. Faster search but needs collision handling and managing table size.
* **Allocation Methods:** How disk blocks are allocated to files:
  * **Contiguous:** Allocates adjacent blocks. Fast sequential/random access but suffers from external fragmentation.
  * **Linked:** Each block points to the next. No external fragmentation, flexible size, but slow random access and overhead for pointers. FAT (File Allocation Table) is a variation where links are stored centrally.
  * **Indexed:** Uses an index block (per file) containing pointers to all data blocks. Supports direct access, no external fragmentation, but overhead for index block(s). Large files might need linked index blocks, multilevel index blocks, or a combined scheme (like UNIX inodes with direct, single, double, triple indirect pointers).
* **Free-Space Management:** Tracking available disk blocks:
  * **Bit Vector (Bitmap):** One bit per block (0=used, 1=free). Simple, allows easy finding of contiguous blocks, but can consume significant space for large disks.
  * **Linked List:** Free blocks are linked together. No space wasted on the map, but finding contiguous blocks is hard.
  * **Grouping:** Stores addresses of multiple free blocks within a free block.
  * **Counting:** Stores address of first block and count of contiguous free blocks. Efficient if allocation/freeing often happens contiguously.
* **Efficiency & Performance:** Influenced by allocation/directory methods. Caching (buffer cache, page cache), read-ahead, and free-behind techniques optimize performance. Keeping metadata and data physically close on disk helps.
* **Recovery:** Protecting against data loss or inconsistency due to crashes:
  * **Consistency Checking:** Utilities (like `fsck` in Unix) scan metadata for inconsistencies and attempt repairs. Can be slow.
  * **Backups:** Regularly copying data to separate media.
  * **Log-Structured / Journaling File Systems:** Record pending metadata changes to a log (journal) *before* applying them to the main file system structures. After a crash, the log is replayed to ensure consistency quickly, avoiding lengthy checks.

### Advanced Storage: RAID and Swap Space

Managing disk storage effectively involves more than just file allocation.

* **Swap-Space Management:** When using virtual memory, especially demand paging, the OS needs disk space (the **backing store** or **swap space**) to hold pages that are not currently in physical memory. The OS needs to manage this swap space efficiently. It might be a dedicated disk partition or a file within the regular file system. Managing swap space involves tracking which parts are free and allocating/deallocating space for swapped-out pages. Performance is key, so dedicated partitions are often faster.
* **RAID (Redundant Array of Independent Disks):** A set of techniques to combine multiple physical disk drives into a single logical unit to improve performance, reliability, or both. Common **RAID levels** include:
    * **RAID 0 (Striping):** Data is split across multiple disks. Offers higher performance (parallel reads/writes) but *no* redundancy (if one disk fails, all data is lost).
    * **RAID 1 (Mirroring):** Data is duplicated exactly onto two or more disks. Provides excellent redundancy (system keeps working if one disk fails) but is expensive in terms of disk space (50% capacity utilization for two disks).
    * **RAID 5 (Striping with Parity):** Data is striped across disks, and parity information (a checksum allowing data reconstruction) is also distributed across the disks. Offers a balance of performance and redundancy (can survive one disk failure) with better capacity utilization than mirroring. Requires at least 3 disks.
    * **RAID 6 (Striping with Double Parity):** Like RAID 5 but uses two independent parity blocks, allowing it to survive *two* simultaneous disk failures. Offers higher reliability than RAID 5 at the cost of more complex calculations and slightly lower write performance. Requires at least 4 disks.
    * **RAID 1+0 (or 10):** Combines mirroring and striping. Data is striped across mirrored pairs. Offers good performance and good redundancy.

RAID can be implemented in hardware (dedicated controller card) or software (by the OS).

### Memory-Mapped Files: Files as Memory

This technique maps a file (or portion of a file) directly into a process's virtual address space. Instead of using standard `read()` and `write()` system calls, the process accesses the file contents directly through memory pointers as if it were an array in RAM.

* **How it works:** Uses the demand paging system. Accessing a memory address corresponding to the mapped file triggers page faults if the data isn't already in a physical frame. The OS loads the required file block (page) from disk into a frame. Writes to the memory region are eventually written back to the file on disk by the OS.
* **Benefits:** Simplifies file access logic for the programmer. Potentially faster than standard I/O due to reduced system call overhead and direct use of the page cache (avoiding double buffering). Enables easy sharing of file data between multiple processes (they just map the same file into their respective address spaces).

### Conclusion: The OS's Master Organizer

The file system is far more than just a place to dump files. It's a complex OS component that provides the structure, access methods, sharing capabilities, and reliability we need to manage persistent data effectively. From the basic concept of a file and its attributes, through the various ways directories and disk blocks can be organized, to the crucial implementation details ensuring performance and recovery, the file system is the OS's master organizer, bringing order to the vast landscape of digital information.
