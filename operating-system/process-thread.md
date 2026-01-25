---
title: The Process Polka & Concurrency Cha-Cha - Keeping Your OS in Rhythm!
description: Unravel the concepts of processes, concurrency, and the challenges of keeping them synchronized within your Operating System.
date: 2023-04-04
draft: false
slug: /pensieve/operating-system/process-thread
tags:
  - OS
  - CS Basics
---

Hey OS aficionados! We've explored the overall structure and services of the Operating System. Now, let's zoom in on one of its most crucial roles: managing **processes** and the complex dance of **concurrency**. How does the OS juggle multiple tasks at once? What happens when these tasks need to cooperate or compete? Let's get into the rhythm!

### What is a Process? The Lifeblood of Computation

At its heart, a **process** is simply a **program in execution**. While a program sitting on your hard drive is a passive set of instructions (like sheet music), a process is that program brought to life, actively using system resources to get things done (like musicians playing the music). It's the fundamental unit of work in modern operating systems.

What makes up a process? It's more than just the program code (often called the **text section**). A process also includes:

* **Current Activity:** Represented by the **program counter** (which points to the next instruction to execute) and the contents of the CPU's **registers**.
* **Stack:** A dedicated memory area holding temporary data, such as function parameters, return addresses, and local variables. It grows and shrinks as functions are called and return.
* **Data Section:** Contains global variables accessible throughout the program.
* **Heap:** An area of memory dynamically allocated during the process's runtime (e.g., when you use `malloc` in C or `new` in Java).

**Process States:** As a process executes, it transitions through various states:

1. **New:** The process is being created.
2. **Ready:** The process has everything it needs to run but is waiting for its turn on the CPU.
3. **Running:** The process's instructions are currently being executed by the CPU.
4. **Waiting:** The process is paused, waiting for some event to occur (like I/O completion or receiving a signal).
5. **Terminated:** The process has finished execution.

![Process State](https://upload.wikimedia.org/wikipedia/commons/thumb/8/83/Process_states.svg/1280px-Process_states.svg.png)

**The Process Control Block (PCB):** The OS keeps track of each process using a data structure called the **Process Control Block** (or Task Control Block). Think of it as the process's ID card and file folder, containing vital information like:

* Process State (Running, Waiting, etc.)
* Process ID (PID) - a unique identifier.
* Program Counter
* CPU Registers
* CPU Scheduling Information (priority, queue pointers)
* Memory Management Information (base/limit registers, page tables)
* Accounting Information (CPU time used, time limits)
* I/O Status Information (devices allocated, list of open files)
* Parent/Child Process Information

When the OS switches the CPU from one process to another (a **context switch**), it saves the state of the old process (from its PCB) and loads the saved state of the new process (into its PCB).

### Threads: Lightweight Concurrency Within a Process

**Threads** are lightweight paths of execution *within* a single process. They share the process's code, data, and resources but have their own program counter, registers, and stack.

**User Threads vs. Kernel Threads:**
* **User Threads:** Managed entirely by a user-level library without kernel awareness (e.g., older Green Threads). Fast to create/switch but if one thread blocks on a system call, the entire process blocks. Cannot easily take advantage of multiple cores.
* **Kernel Threads:** Managed directly by the OS kernel. Slower to create/switch than user threads but allow true parallelism on multi-core systems, and one thread blocking doesn't stop others in the process. Virtually all modern general-purpose OSs (Windows, Linux, macOS, Solaris) support kernel threads.

**Multithreading Models:** These define the relationship between user and kernel threads:
* **Many-to-One:** Maps many user threads to one kernel thread. Simple but suffers from the blocking issue and lack of parallelism. (Rare now).
* **One-to-One:** Maps each user thread to a dedicated kernel thread. Provides true parallelism and avoids blocking issues but can be resource-intensive if many threads are created. (Common in Linux, Windows, newer Solaris).
* **Many-to-Many:** Multiplexes many user threads onto a smaller or equal number of kernel threads. Offers flexibility and parallelism without excessive kernel resource use. Some variations allow binding a user thread to a specific kernel thread (**Two-Level Model**). (Used in older Solaris, IRIX, HP-UX).

**Thread Libraries:** Provide programmers with an API for creating and managing threads. Three main libraries:
    * **POSIX Pthreads:** A standard API (IEEE 1003.1c) common on UNIX-like systems (Linux, macOS, Solaris). Can be implemented as user-level or kernel-level. Specifies behavior, not implementation.
    * **Windows Threads:** Kernel-level library specific to Windows.
    * **Java Threads:** Managed by the Java Virtual Machine (JVM); implementation often uses the underlying OS's thread library (e.g., Pthreads or Windows threads).

**Implicit Threading:** Shifts thread creation and management burden from programmers to compilers and run-time libraries. Approaches include:
    * **Thread Pools:** Create a pool of threads at startup. When work arrives, assign it to an available thread from the pool. More efficient than creating/destroying threads for short tasks.
    * **OpenMP:** Compiler directives and API for C/C++/Fortran that automatically parallelize code sections (often loops) using threads. Programmers add `#pragma omp ...` directives.

**Common Threading Issues:**
    * **`fork()` and `exec()` Semantics:** When a thread calls `fork()`, does the new process duplicate *all* threads or just the calling thread? Some systems offer both options. `exec()` normally replaces the entire process (all threads).
    * **Signal Handling:** In UNIX, where should a signal (asynchronous notification) be delivered? To a specific thread? To all threads? To the first thread not blocking the signal? Varies by signal type and system.
    * **Thread Cancellation:** Terminating a thread before it completes.
        * *Asynchronous Cancellation:* Terminates the target thread immediately. Risky if thread holds resources or is in the middle of an update.
        * *Deferred Cancellation:* Target thread periodically checks if it should be cancelled, allowing it to terminate gracefully.
    * **Thread-Local Storage (TLS):** Sometimes threads need their own private copies of certain data (not shared with other threads in the process). TLS provides storage specific to each thread.

**OS Thread Implementation Examples:**
    * **Linux:** Refers to threads as *tasks*. Thread creation uses the `clone()` system call, which allows finer control over resource sharing (e.g., address space) between parent and child compared to `fork()`. Supports the one-to-one model.
    * **Windows:** Uses a one-to-one model. Provides a comprehensive API for thread creation and management.

### The Principle of Concurrency: Juggling Multiple Tasks

Modern operating systems rarely run just one process at a time. **Concurrency** is the concept of having multiple processes (or threads within processes) making progress over the same period. On a single-CPU system, this is an illusion created by rapidly switching the CPU between processes (**multitasking** or **time-sharing**). On multi-core systems, processes can run in **parallel**, with multiple processes truly executing simultaneously on different cores.

Concurrency is fantastic for responsiveness and throughput, but it introduces a significant challenge: what happens when multiple processes need to access the *same* data or resource?

### The Critical Section Problem: Avoiding Data Chaos

Imagine two processes trying to update the same bank account balance simultaneously. Process A reads the balance ($100), calculates the new balance ($100 + $50 = $150), but before it can write it back, the OS switches to Process B. Process B reads the *original* balance ($100), calculates *its* new balance ($100 - $20 = $80), and writes $80 back. Then, Process A gets the CPU back and writes *its* calculated value, $150. The $20 withdrawal is lost! This is a **race condition**.

The part of a program where it accesses shared resources (like shared variables, files, or data structures) is called its **critical section**. The **Critical Section Problem** is designing a protocol that processes can use to cooperate, ensuring that when one process is executing in its critical section, no other process is allowed to execute in *its* critical section related to the same shared resource.

A valid solution must satisfy three requirements:

1. **Mutual Exclusion:** If one process is in its critical section, no other processes can be executing in *their* critical sections (for the same resource).
2. **Progress:** If no process is in its critical section, and some processes wish to enter theirs, then only those processes *not* in their remainder sections can participate in deciding which will enter next, and this decision cannot be postponed indefinitely.
3. **Bounded Waiting:** There must be a limit on the number of times other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted. This prevents **starvation**, where a process waits indefinitely.

Early solutions involved complex software algorithms (like Peterson's Solution), but modern systems often rely on hardware support (like atomic test-and-set or compare-and-swap instructions) or OS-provided tools.

### Semaphores: The Signaling System for Processes

Operating systems provide synchronization tools to help programmers solve the critical section problem. One fundamental tool is the **Semaphore**. Invented by Edsger Dijkstra, a semaphore is essentially an integer variable that, apart from initialization, is accessed only through two standard, **atomic** (indivisible) operations:

* `wait(S)` (originally `P`): Decrements the semaphore value. If the value becomes negative, the process calling `wait()` blocks (is put to sleep) until the semaphore value is positive again.
  ```
  // Conceptual implementation (blocking, not busy-waiting)
  wait(semaphore *S) {
      S->value--;
      if (S->value < 0) {
          // Add this process to S's waiting list
          block(); // Put the process to sleep
      }
  }
  ```
* `signal(S)` (originally `V`): Increments the semaphore value. If there are processes blocked waiting on this semaphore, one of them is woken up.
  ```
  // Conceptual implementation
  signal(semaphore *S) {
      S->value++;
      if (S->value <= 0) {
          // Remove a process P from S's waiting list
          wakeup(P); // Move P to the ready queue
      }
  }
  ```

**Types of Semaphores:**

* **Counting Semaphore:** The integer value can range over an unrestricted domain. Used to control access to a resource with multiple instances (e.g., the semaphore count represents the number of available resources).
* **Binary Semaphore:** The integer value can only be 0 or 1. Essentially acts as a **mutex lock**, providing mutual exclusion for critical sections. A process waits if the value is 0, sets it to 0 upon entry, and signals (sets to 1) upon exit.

**Implementation:** The `wait()` and `signal()` operations themselves must be atomic to avoid race conditions within the semaphore logic. Early implementations used **busy waiting** (spinning in a loop checking the value), which wastes CPU cycles. Modern implementations associate a waiting queue with each semaphore. Processes that must wait are placed on the queue (**block** operation), and the `signal` operation removes a process from the queue and places it in the ready queue (**wakeup** operation), avoiding busy waiting.

**Potential Problems:** While powerful, semaphores can be tricky:

* Incorrect usage (e.g., signaling before waiting, forgetting a wait or signal) can lead to violations of mutual exclusion or deadlock.
* **Deadlock:** Two or more processes waiting indefinitely for an event that can only be caused by one of the waiting processes (e.g., Process A holds Lock1 and waits for Lock2, while Process B holds Lock2 and waits for Lock1).
* **Starvation:** A process may wait indefinitely in the semaphore's queue and never get woken up.

### Classical Concurrency Conundrums

To test the effectiveness of synchronization mechanisms, several classical problems are often used:

1. **The Bounded-Buffer (Producer-Consumer) Problem:**

   * **Scenario:** A producer process generates items and puts them into a fixed-size buffer; a consumer process takes items out. Prevent adding to a full buffer or removing from an empty one. Ensure only one process manipulates the buffer at a time.
   * **Shared Data:**
     * `item buffer[BUFFER_SIZE];`
     * `int in = 0;` (index for next produced item)
     * `int out = 0;` (index for next consumed item)
   * **Semaphores:**
     * `semaphore mutex = 1;` (For mutual exclusion on buffer access)
     * `semaphore empty = BUFFER_SIZE;` (Counts empty buffer slots)
     * `semaphore full = 0;` (Counts full buffer slots)
   * **Producer Process Algorithm:**
     ```
     do {
         // ... produce an item in next_produced ...
         wait(empty);   // Wait if buffer is full
         wait(mutex);   // Acquire lock for buffer access
         // ... add next_produced to buffer[in] ...
         in = (in + 1) % BUFFER_SIZE;
         signal(mutex); // Release lock
         signal(full);  // Signal that a slot is now full
     } while (true);
     ```
   * **Consumer Process Algorithm:**
     ```
     do {
         wait(full);    // Wait if buffer is empty
         wait(mutex);   // Acquire lock for buffer access
         // ... remove an item from buffer[out] to next_consumed ...
         out = (out + 1) % BUFFER_SIZE;
         signal(mutex); // Release lock
         signal(empty); // Signal that a slot is now empty
         // ... consume the item in next_consumed ...
     } while (true);
     ```
2. **The Readers-Writers Problem:**

   * **Scenario:** Shared data accessed by Readers (read-only) and Writers (read/write). Allow multiple Readers concurrently, but only one Writer at a time (with no concurrent Readers).
   * **Shared Data:**
     * `Data set` (the shared resource)
     * `int read_count = 0;` (Tracks number of active readers)
   * **Semaphores:**
     * `semaphore mutex = 1;` (For mutual exclusion on read_count)
     * `semaphore rw_mutex = 1;` (For mutual exclusion for writers, and first/last reader)
   * **Writer Process Algorithm:**
     ```
     do {
         wait(rw_mutex); // Acquire exclusive access
         // ... writing is performed ...
         signal(rw_mutex); // Release exclusive access
     } while (true);
     ```
   * **Reader Process Algorithm (Reader Priority):**
     ```
     do {
         wait(mutex);        // Lock to update read_count
         read_count++;
         if (read_count == 1) {
             wait(rw_mutex); // First reader locks out writers
         }
         signal(mutex);      // Unlock read_count

         // ... reading is performed ...

         wait(mutex);       // Lock to update read_count
         read_count--;
         if (read_count == 0) {
             signal(rw_mutex); // Last reader unlocks for writers
         }
         signal(mutex);     // Unlock read_count
     } while (true);
     ```
   * **Note:** This version prioritizes readers; writers might starve if readers are constantly active. Other variations exist to balance priorities.
3. **The Dining Philosophers Problem:**

   * **Scenario:** Five philosophers around a table, alternating thinking and eating. Need two chopsticks (left and right) to eat. Pick up one at a time. Goal: Avoid deadlock and starvation.
   * **Shared Data:**
     * `semaphore chopstick[5];` (One semaphore per chopstick, initialized to 1)
   * **Philosopher `i` Algorithm (Naive - Leads to Deadlock):**
     ```
     do {
         // THINK
         wait(chopstick[i]);              // Pick up left chopstick
         wait(chopstick[(i + 1) % 5]);    // Pick up right chopstick

         // EAT

         signal(chopstick[(i + 1) % 5]);  // Put down right chopstick
         signal(chopstick[i]);            // Put down left chopstick
     } while (TRUE);
     ```
   * **Deadlock:** If all philosophers pick up their left chopstick simultaneously, none can pick up their right, and they all wait forever.
   * **Deadlock-Free Solutions (Ideas):** Limit the number of philosophers at the table to four; allow pickup only if *both* chopsticks are available (requires an atomic check-and-grab, often within a mutex or monitor); use an asymmetric approach where odd philosophers pick left-then-right, and even philosophers pick right-then-left.

### Peterson's Solution (Algorithm Detail)

A classic software-based solution for *two* processes (Pi and Pj). Assumes atomic load/store operations.

* Shared variables: `int turn; boolean flag[2];` (initially `flag[0]=flag[1]=false`, `turn` can be 0 or 1).
* Algorithm for Process Pi:
    ```
    do {
        flag[i] = true;             // Indicate Pi wants to enter
        turn = j;                   // Politely say "your turn"
        while (flag[j] && turn == j) {
            ; // Busy wait if Pj wants in AND it's Pj's turn
        }

        // --- Critical Section ---

        flag[i] = false;            // Pi is done

        // --- Remainder Section ---
    } while (true);
    ```
    This solution correctly provides mutual exclusion, progress, and bounded waiting for two processes but doesn't generalize easily and requires busy waiting.

### Hardware Synchronization Support

Modern CPUs provide atomic instructions to help build synchronization locks efficiently:

* **`test_and_set(boolean *target)`:** Atomically returns the original value of `*target` and sets `*target` to `true`.
    ```
    // Atomic Definition
    boolean test_and_set (boolean *target) {
        boolean rv = *target;
        *target = TRUE;
        return rv;
    }
    // Simple Spinlock using test_and_set
    // Shared: boolean lock = false;
    do {
        while (test_and_set(&lock)) {
           ; // Spin wait
        }
        // --- Critical Section ---
        lock = false;
        // --- Remainder Section ---
    } while (true);
    ```
* **`compare_and_swap(int *value, int expected, int new_value)`:** Atomically compares `*value` with `expected`. If they match, sets `*value` to `new_value`. Returns the *original* value of `*value`.
    ```
    // Atomic Definition
    int compare_and_swap(int *value, int expected, int new_value) {
        int temp = *value;
        if (*value == expected) {
            *value = new_value;
        }
        return temp;
    }
    // Simple Spinlock using compare_and_swap
    // Shared: int lock = 0; // 0 = unlocked, 1 = locked
    do {
        while (compare_and_swap(&lock, 0, 1) != 0) {
            ; // Spin wait (keep trying until lock was 0 and we set it to 1)
        }
        // --- Critical Section ---
        lock = 0;
        // --- Remainder Section ---
    } while (true);
    ```
* **Bounded-Waiting Mutex with `test_and_set`:** The simple spinlocks above don't guarantee bounded waiting. A more complex algorithm using `test_and_set` and an additional `waiting[]` array can achieve this:
    ```
    // Shared: boolean waiting[n]; boolean lock; (all init false)
    // Algorithm for process Pi
    do {
        waiting[i] = true;
        boolean key = true;
        while (waiting[i] && key) {
            key = test_and_set(&lock); // Try to get lock
        }
        waiting[i] = false; // Got the lock or waiting ended

        // --- Critical Section ---

        int j = (i + 1) % n; // Find next waiting process
        while ((j != i) && !waiting[j]) {
            j = (j + 1) % n;
        }

        if (j == i) { // No one else waiting
            lock = false; // Release lock
        } else { // Someone else is waiting
            waiting[j] = false; // Pass lock directly (avoids busy wait for j)
                                // Note: lock remains true here!
        }

        // --- Remainder Section ---
    } while (true);
    ```

### Inter-Process Communication (IPC): Processes Talking to Each Other

Processes often need to cooperate and exchange information. **Inter-Process Communication (IPC)** provides the mechanisms for this. Cooperating processes can speed up computation, allow for modular design, share information conveniently, and build complex systems.

Two primary models exist:

1. **Shared Memory:** The OS establishes a region of memory shared between cooperating processes. Processes can then read and write to this shared region directly. It's very fast (like accessing normal memory) but requires careful synchronization (using tools like semaphores) to control access and prevent race conditions. The OS is involved only in setting up the shared space; subsequent communication doesn't directly involve the kernel. The Producer-Consumer problem can be implemented by having the shared buffer in this memory region.
2. **Message Passing:** Processes communicate by exchanging messages without sharing memory space. The OS provides `send(message)` and `receive(message)` operations. This is often easier to implement correctly, especially across a network, as synchronization is inherent in message passing (a process typically waits if trying to receive when no message is available). The OS is actively involved in managing the message transfer. A communication **link** (logical or physical) must be established between the processes.

### Monitors: A Higher-Level Synchronization Tool

Because semaphores can be difficult to use correctly, **Monitors** were developed as a higher-level synchronization construct, often built into programming languages. A monitor is an **abstract data type** that encapsulates shared data variables and the procedures that operate on them.

Key features:

* **Mutual Exclusion:** Only *one* process can be actively executing *inside* the monitor's procedures at any given time. This is guaranteed implicitly by the monitor construct itself, simplifying programming.
* **Condition Variables:** Monitors use **condition variables** to manage synchronization *within* the monitor. A process might need to wait for a specific condition to become true before proceeding.
  * `x.wait()`: Suspends the calling process until another process signals condition `x`. The process releases the monitor lock while waiting.
  * `x.signal()`: Resumes *one* process (if any) that is waiting on condition `x`. If no process is waiting, the signal has no effect.

**Signaling Semantics:** A subtle point is what happens after a `signal`. If process P signals and process Q was waiting, Q is now ready to run. But P is also still inside the monitor. Since only one can be active, what happens?

* **Signal and Wait:** P waits (either outside the monitor or on another condition) until Q leaves the monitor or waits again.
* **Signal and Continue:** Q waits until P leaves the monitor or waits.

Monitors provide a structured way to manage shared data and synchronization, reducing the likelihood of errors compared to raw semaphore usage. The Dining Philosophers problem, for instance, can be solved more elegantly using a monitor to manage chopstick state and philosopher conditions (Hungry, Eating, Thinking).

### Process Generation and Termination: The Circle of Life

Processes aren't static; they are created and eventually terminate.

**Process Creation (Generation):**
Operating systems provide mechanisms for one process (a **parent**) to create another process (a **child**). This typically forms a tree structure. Key aspects include:

* **Identification:** Each process gets a unique Process ID (PID).
* **Resource Sharing:** Options vary:
  * Parent and child share all resources.
  * Child shares a subset of parent's resources.
  * Parent and child share no resources.
* **Execution:**
  * Parent and child execute concurrently.
  * Parent waits until some or all children terminate.
* **Address Space:**
  * The child might be a duplicate of the parent (sharing the same program and data initially, often using Copy-on-Write).
  * The child might have a new program loaded into its address space.
  * In UNIX-like systems, `fork()` creates a duplicate child process, and `exec()` (often called after `fork()`) replaces the child's memory space with a new program.

**Process Termination:**
Processes end execution in several ways:

* **Normal Exit:** The process finishes its task and calls `exit()`.
* **Error Exit:** The process encounters an error and voluntarily terminates (e.g., `abort()`).
* **Fatal Error:** An unrecoverable error occurs (e.g., division by zero, illegal memory access), causing the OS to terminate the process.
* **Termination by Parent:** A parent process may terminate a child using a system call (`abort()`), perhaps because the child exceeded resources, its task is no longer needed, or the parent itself is exiting.
* **Cascading Termination:** In some systems, if a parent terminates, the OS automatically terminates all its children (and grandchildren, etc.).

When a child terminates, it might return a status value to its parent. The parent uses a call like `wait()` to wait for a child to terminate and collect this status.

* **Zombie Process:** A process that has terminated, but whose parent hasn't called `wait()` yet. Its PCB is kept around to hold the exit status.
* **Orphan Process:** A process whose parent terminated without waiting. These are often "adopted" by a system process like `init` which periodically calls `wait()` to clean them up.

### Process Scheduling: Deciding Who Runs When

In a multiprogramming/time-sharing system, there are often multiple processes ready to run. The **Process Scheduler** (specifically the **short-term** or **CPU scheduler**) decides which ready process gets the CPU next.

**Why Schedule?** Processes alternate between CPU execution (**CPU bursts**) and I/O waiting (**I/O bursts**). Scheduling aims to keep the CPU busy during the I/O waits of other processes, maximizing **CPU utilization** and system **throughput**. In interactive systems, it also aims to minimize **response time**.

**Scheduling Queues:** The OS maintains several queues:

* **Job Queue:** Contains all processes in the system.
* **Ready Queue:** Contains processes residing in main memory that are ready and waiting to execute.
* **Device Queues:** Contain processes waiting for a particular I/O device.

Processes migrate between these queues throughout their lifetime.

**Schedulers:**

* **Long-Term Scheduler (Job Scheduler):** Selects processes from the job pool (on disk) and loads them into memory (ready queue). Controls the **degree of multiprogramming** (number of processes in memory). Runs infrequently. Strives for a good mix of **I/O-bound** (many short CPU bursts) and **CPU-bound** (few long CPU bursts) processes.
* **Short-Term Scheduler (CPU Scheduler):** Selects the next process from the ready queue and allocates the CPU. Runs *very* frequently (milliseconds), so must be fast.
* **Medium-Term Scheduler:** Sometimes used to temporarily remove processes from memory (and active contention for the CPU) to reduce the degree of multiprogramming (e.g., via **swapping**).

**Context Switching:** The mechanism to switch the CPU from one process to another. It involves saving the state (context) of the current process and loading the saved state of the next process. This represents overhead, as no useful work is done during the switch itself.

### That's the Tempo!

Managing processes and their concurrent execution is a fundamental, complex, and fascinating aspect of operating systems. From defining what a process *is*, to ensuring concurrent processes don't corrupt shared data using semaphores and monitors, to handling classic synchronization challenges, and finally deciding which process runs when – the OS truly keeps the entire system in rhythm!
