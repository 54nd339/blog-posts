---
title: CPU Scheduling - The OS's Art of CPU Time Management
description: Discover how Operating Systems decide which process runs when, explore the algorithms they use, and learn how performance is measured in the world of CPU scheduling.
date: 2023-04-05
draft: false
slug: /pensieve/operating-system/process-scheduling
tags:
  - OS
  - CS Basics
---

Hey everyone! We've established that **CPU Scheduling** is the OS's critical task of deciding which ready process gets the CPU next. It's essential for juggling multiple tasks and keeping the system responsive and efficient. But *how* does the OS make these decisions? It uses **scheduling algorithms**. Let's dive into the mechanics of the most common ones, exploring their evolution and trade-offs.

### The Scheduling Concept: Why We Need a CPU Traffic Cop

Quick recap: Processes run in **CPU–I/O Burst Cycles**. To maximize **CPU utilization**, the OS uses **multiprogramming**, switching the CPU to another process when the current one waits for I/O. The **Short-Term Scheduler** picks the next process from the **Ready Queue**. This can happen **nonpreemptively** (process runs until it blocks or finishes) or **preemptively** (OS can force a switch). The **Dispatcher** handles the actual context switch, incurring some **dispatch latency**.

### Performance Criteria: How Do We Judge a Scheduling Algorithm?

Before diving into algorithms, remember our key metrics:

* **CPU Utilization:** Keep the CPU busy.
* **Throughput:** Maximize completed processes per time unit.
* **Turnaround Time:** Minimize time from submission to completion.
* **Waiting Time:** Minimize time spent waiting in the ready queue.
* **Response Time:** Minimize time until the first response for interactive tasks.

Now, let's see how different algorithms stack up against these criteria.

### Scheduling Algorithms: The Evolution of CPU Allocation Strategies

Think of these algorithms as evolving strategies, each trying to improve on the limitations of the previous ones or cater to different system needs.

1. **First-Come, First-Served (FCFS):**

   * **Algorithm:** The simplest approach. Processes are added to the ready queue as they become ready. The process at the head of the queue (the one that arrived first) gets the CPU when it becomes free. It's implemented using a standard FIFO (First-In, First-Out) queue.
   * **Nature:** Strictly **Nonpreemptive**. Once a process starts its CPU burst, it runs uninterrupted until it finishes the burst, blocks for I/O, or terminates.
   * **Example:**
     * Processes arrive: P1 (Burst=24ms), P2 (Burst=3ms), P3 (Burst=3ms) in that order.
     * Gantt Chart: `| P1 (0-24) | P2 (24-27) | P3 (27-30) |`
     * Waiting times: P1=0, P2=24, P3=27. Average Waiting Time = (0+24+27)/3 = 17ms.
     * If they arrived P2, P3, P1: `| P2 (0-3) | P3 (3-6) | P1 (6-30) |`
     * Waiting times: P1=6, P2=0, P3=3. Average Waiting Time = (6+0+3)/3 = 3ms. (Big difference!)
   * **Pros:** Simple, easy to implement, inherently fair (no starvation).
   * **Cons:** Performance is highly sensitive to arrival order. Suffers from the **convoy effect**: short processes can get stuck waiting for a very long CPU-bound process to finish, leading to poor average waiting time and low utilization of I/O devices (as I/O-bound processes wait for the CPU). Unsuitable for interactive systems due to potentially long response times.
2. **Shortest-Job-First (SJF):**

   * **Algorithm:** Aims to minimize waiting time. Associates each process with the length of its *next* CPU burst. When the CPU is free, it's allocated to the process predicted to have the shortest *upcoming* CPU burst. Ties are broken using FCFS.
   * **Nature:** Can be **Nonpreemptive** or **Preemptive**.
     * *Nonpreemptive SJF:* Once the CPU is given to a process, it cannot be preempted until it completes its *current* CPU burst.
     * *Preemptive SJF (Shortest-Remaining-Time-First - SRTF):* If a new process arrives in the ready queue with a predicted CPU burst length *shorter* than the *remaining* time of the currently executing process's burst, the scheduler preempts the current process and allocates the CPU to the new, shorter process.
   * **Prediction:** Since we can't know the future burst length, we *estimate* it. The most common method is **exponential averaging** of previous burst lengths:
     $ \tau_{n+1} = \alpha t_n + (1-\alpha) \tau_n $
     Where:
     * $t_n$ = actual length of the $n^{th}$ CPU burst.
     * $\tau_n$ = predicted length of the $n^{th}$ CPU burst.
     * $\tau_{n+1}$ = predicted length of the next ($n+1^{th}$) CPU burst.
     * $\alpha$ (alpha) is a weighting factor ($0 \le \alpha \le 1$). If $\alpha=0$, past predictions ($\tau_n$) dominate. If $\alpha=1$, only the last actual burst ($t_n$) matters. Often, $\alpha=0.5$ is used, giving equal weight to recent history and past prediction.
   * **Example (SRTF):**
     * Processes: P1(Arrival=0, Burst=7), P2(Arrival=2, Burst=4), P3(Arrival=4, Burst=1), P4(Arrival=5, Burst=4)
     * Gantt Chart (SRTF): `| P1(0-2) | P2(2-4) | P3(4-5) | P2(5-7) | P4(7-11) | P1(11-16) |`
     * (P1 starts -> P2 arrives, P1 remaining=5 > P2 burst=4, preempt P1 -> P2 starts -> P3 arrives, P2 remaining=2 > P3 burst=1, preempt P2 -> P3 runs & finishes -> P4 arrives, P2 remaining=2 < P4 burst=4, resume P2 -> P2 finishes -> P4 runs -> P4 finishes -> P1 runs & finishes)
     * Average waiting time calculation is complex but generally lower than non-preemptive SJF for this input.
   * **Pros:** Provably optimal for minimizing *average* waiting time. SRTF is optimal among preemptive algorithms.
   * **Cons:** Requires predicting the next CPU burst length, which is impossible to do perfectly. Relies on estimation, reducing practical optimality. Prone to **starvation** for processes with long CPU bursts if there's a continuous supply of short-burst processes. SRTF involves more context switches than non-preemptive SJF.
3. **Priority Scheduling:**

   * **Algorithm:** Assigns a priority level (typically an integer) to each process. The CPU is allocated to the process with the highest priority. Processes with equal priority are usually scheduled FCFS. Priorities can be assigned internally (based on process characteristics like memory requirements, burst times) or externally (based on user importance, payment, etc.).
   * **Nature:** Can be **Preemptive** or **Nonpreemptive**.
     * *Preemptive:* If a new process arrives with a higher priority than the currently running process, the running process is preempted.
     * *Nonpreemptive:* The new process simply goes to the head of the ready queue.
   * **Example:**
     * Processes: P1(Burst=10, Prio=3), P2(Burst=1, Prio=1), P3(Burst=2, Prio=4), P4(Burst=1, Prio=5), P5(Burst=5, Prio=2). (Assume lower number = higher priority).
     * Gantt Chart (Nonpreemptive): `| P2(0-1) | P5(1-6) | P1(6-16) | P3(16-18) | P4(18-19) |`
     * (Preemptive could differ if arrivals were staggered).
   * **Pros:** Allows for explicit control over process importance. Can implement policy decisions effectively.
   * **Cons:** Major issue is **starvation** – low-priority processes might never get the CPU if there's a constant supply of higher-priority ones. Solution: **Aging**, where the priority of a process increases gradually the longer it waits.
4. **Round Robin (RR):**

   * **Algorithm:** Designed for time-sharing. Uses a fixed time unit called a **time quantum** (`q`). The ready queue is treated as a circular FIFO queue. The scheduler allocates the CPU to the process at the head of the queue for *at most* one time quantum.
     * If the process finishes its CPU burst before `q` expires, it releases the CPU voluntarily.
     * If the burst is longer than `q`, a timer interrupt occurs, the OS performs a context switch, and the process is moved to the *tail* of the ready queue.
     * If the process blocks for I/O before `q` expires, it releases the CPU and goes to the relevant device queue.
   * **Nature:** Always Preemptive.
   * **Example:**
     * Processes: P1(Burst=24), P2(Burst=3), P3(Burst=3). Time Quantum `q=4`.
     * Gantt Chart: `| P1(0-4) | P2(4-7) | P3(7-10) | P1(10-14) | P1(14-18) | P1(18-22) | P1(22-26) | P1(26-30) |`
     * (P1 runs 4ms -> P2 runs 3ms -> P3 runs 3ms -> P1 runs 4ms -> P1 runs 4ms -> P1 runs 4ms -> P1 runs 4ms -> P1 runs remaining 2ms)
   * **Pros:** Fair allocation of CPU time. Good average response time, suitable for interactive systems. Prevents starvation (each process gets $1/n$ of the CPU time in chunks of at most $q$).
   * **Cons:** Average turnaround time is often worse than SJF. Performance is very sensitive to the time quantum `q`:
     * Large `q`: Approaches FCFS behavior.
     * Small `q`: Improves response time but increases context switching overhead, potentially lowering throughput if `q` is too close to the context switch time. `q` must be significantly larger than context switch time (e.g., q=10-100ms, switch < 10µs).
5. **Multilevel Queue Scheduling:**

   * **Algorithm:** Classifies processes into different groups and assigns them to different ready queues. Common divisions include foreground (interactive) vs. background (batch), or system vs. user processes. Processes are *permanently* assigned to a queue upon entry. Each queue has its own scheduling algorithm (e.g., RR for foreground, FCFS for background).
   * **Inter-Queue Scheduling:** A strategy is needed to schedule between the queues themselves:
     * *Fixed Priority Preemptive:* Queues have absolute priority (e.g., serve all from foreground before any background). High-priority processes preempt low-priority ones. Risk of **starvation** for lower-priority queues.
     * *Time Slice:* Each queue gets a specific percentage of CPU time, which it can distribute among its processes using its own algorithm (e.g., 80% for foreground RR, 20% for background FCFS). Fairer but more complex.
   * **Pros:** Can cater to different process needs by using appropriate algorithms for each class. Low scheduling overhead within a queue.
   * **Cons:** Inflexible, as processes cannot change queues. Fixed priority scheduling can lead to starvation.
6. **Multilevel Feedback Queue Scheduling:**

   * **Algorithm:** The most general (and complex) approach. Like Multilevel Queue, but it allows processes to **migrate between queues**. If a process uses too much CPU time (behaves like a CPU-bound process), it can be moved to a lower-priority queue. If a process waits too long in a low-priority queue (perhaps it became I/O-bound), it can be moved to a higher-priority queue (a form of aging to prevent starvation).
   * **Configuration:** Defined by several parameters:
     * Number of queues.
     * Scheduling algorithm for each queue.
     * Method to determine when to *upgrade* a process to a higher queue.
     * Method to determine when to *demote* a process to a lower queue.
     * Method to determine which queue a process enters initially.
   * **Example Configuration:**
     * Queue 0: RR with quantum 8ms.
     * Queue 1: RR with quantum 16ms.
     * Queue 2: FCFS.
     * Scheduling: New jobs enter Q0. If they don't finish in 8ms, move to Q1. If they don't finish in 16ms in Q1, move to Q2. Serve Q0 completely, then Q1 completely, then Q2. A process arriving for Q0 preempts Q1/Q2; arrival for Q1 preempts Q2. (Aging could be added to promote from Q2/Q1 if waiting too long).
   * **Pros:** Very flexible, can be tuned to specific needs. Can implement aging and prevent starvation effectively. Can adapt to process behavior changes.
   * **Cons:** Most complex algorithm to design, implement, and tune. Parameter selection is critical and non-trivial. Over-parameterization can lead to high overhead.

### Thread Scheduling: Processes vs. Threads

When an OS supports **kernel-level threads** (as most modern OSs do), it's the *threads*, not the processes, that are being scheduled onto the CPU.

* **Distinction:** While a process groups resources, a thread is the unit of CPU execution. If multiple threads belong to the same process, they share resources but compete individually for CPU time.
* **User-Level vs. Kernel-Level Threads:**
    * Systems using **user-level threads** (managed by a thread library without kernel awareness, mapped Many-to-One or Many-to-Many to kernel threads) schedule *processes*. The thread library within the process then decides which user thread runs when that process gets the CPU. This is often called **Process Contention Scope (PCS)**, as threads within the same process compete for CPU time allocated to that process.
    * Systems using **kernel-level threads** (where the kernel sees and manages each thread, typically One-to-One mapping) schedule the *kernel threads* directly onto the CPU. This is **System Contention Scope (SCS)**, as all threads in the system compete for CPU time on a system-wide basis.
* **Implications:** SCS allows kernel threads from the same process to run truly in parallel on different CPUs and prevents the entire process from blocking if one thread makes a blocking system call. Most modern systems (Linux, Windows, Solaris) primarily use SCS.

### Multiprocessor Scheduling: Handling Multiple Cores

Scheduling becomes significantly more complex with multiple CPUs (cores). The scheduler must now decide not only *which* thread/process to run but *which CPU* to run it on.

* **Approaches:**
    * **Asymmetric Multiprocessing:** One processor (the master) handles all scheduling, I/O, and system tasks; others just execute user code. Simple but creates a bottleneck.
    * **Symmetric Multiprocessing (SMP):** Each processor is self-scheduling. They might share a common ready queue or have separate private queues. This is the standard modern approach, offering better performance and reliability.
* **Key Challenges in SMP:**
    * **Processor Affinity:** Threads often benefit from staying on the same processor (**cache affinity**) because data they recently accessed might still be in that processor's cache memory. Moving a thread might require cache invalidation and reloading, hurting performance.
        * *Soft Affinity:* The OS *tries* to keep a thread on the same processor but doesn't guarantee it (e.g., if load balancing requires moving it).
        * *Hard Affinity:* Processes/threads can sometimes specify a subset of processors they are allowed to run on.
    * **Load Balancing:** We need to keep the workload distributed evenly across all CPUs to prevent some from being idle while others are overloaded. This is crucial for throughput. Load balancing algorithms might periodically redistribute threads from busy queues to less busy ones. This naturally conflicts with processor affinity. Strategies include:
        * *Push Migration:* A specific task periodically checks loads and pushes threads from overloaded CPUs to others.
        * *Pull Migration:* An idle processor pulls a waiting task from a busy processor's queue.
    * **Synchronization:** With multiple schedulers potentially accessing shared ready queues or other structures, careful synchronization is needed to prevent race conditions within the OS itself.

Modern SMP schedulers often use per-CPU ready queues to minimize contention and improve affinity, employing load balancing only when significant imbalances occur.

### A Glimpse into Real-Time Scheduling

Some systems have stringent timing requirements where completing a task *on time* is as important as getting the correct result. These **Real-Time Operating Systems (RTOS)** require specialized scheduling.

* **Hard vs. Soft Real-Time:**
    * **Hard Real-Time:** Guarantees that critical tasks complete within their deadlines. Missing a deadline is considered a system failure (e.g., flight control, industrial automation).
    * **Soft Real-Time:** Aims to meet deadlines for critical processes but allows occasional misses (e.g., multimedia streaming, where a dropped frame is undesirable but not catastrophic).
* **Focus on Latency:** Real-time scheduling prioritizes minimizing **latency**. Key latencies include:
    * *Interrupt Latency:* Time from interrupt arrival to start of the interrupt service routine.
    * *Dispatch Latency:* Time taken by the dispatcher to stop one process/thread and start another. OSs need to minimize intervals where interrupts are disabled and make dispatchers very fast. Preemptible kernels are crucial here.
* **Real-Time Scheduling Algorithms:** General-purpose algorithms like RR or Priority aren't sufficient. Real-time algorithms prioritize tasks based on their timing requirements. Common examples include:
    * **Rate-Monotonic Scheduling:** A static priority algorithm assigning higher priority to tasks that need to run more frequently (shorter period). Optimal static-priority algorithm if tasks meet certain criteria.
    * **Earliest-Deadline-First (EDF) Scheduling:** Dynamically assigns priorities based on upcoming deadlines – the task with the earliest deadline gets the highest priority. Theoretically optimal (if the system isn't overloaded, EDF can schedule tasks that any other algorithm can).
    * **Proportional Share Scheduling:** Allocates CPU time in specific proportions (shares) among different tasks over time.

Implementing and analyzing real-time scheduling is complex and involves deep understanding of task execution times, periods, and deadlines.

### Evaluating the Algorithms: How to Choose?

With so many algorithms, how do OS designers or system administrators pick the best one for a specific system and workload? They use various evaluation techniques:

1.  **Deterministic Modeling:** Takes a predetermined workload (a specific set of processes with known burst times, arrival times, priorities) and calculates the performance of each algorithm (e.g., average waiting time, throughput) for *that specific workload*. Simple and fast, gives exact numbers for a given input, but results only apply to that specific input and not necessarily to the real system behavior.
2.  **Queuing Models (Analytic Evaluation):** Uses mathematical formulas and queuing network theory. The system workload (arrival rates, service rates/burst lengths) is described using probability distributions. Formulas calculate average throughput, utilization, waiting time, etc. Useful for getting approximate performance characteristics but relies on potentially unrealistic assumptions about distributions.
3.  **Simulations:** Involves programming a model of the computer system. Gather statistics on system behavior by running the simulation with trace data (recorded sequences of real events from an actual system) or randomly generated data based on distributions. More flexible and accurate than queuing models but requires careful programming and potentially long execution times for the simulator. Trace tapes provide accurate input but are specific to the system traced; random generation allows broader exploration.

Often, a combination of simulation and real-world testing (implementing the algorithm and measuring its performance on the actual system) is used to select and tune scheduling algorithms.

### The Final Beat

Choosing a CPU scheduling algorithm involves understanding trade-offs. There's no single "best" algorithm; the choice depends heavily on the system's goals (e.g., maximizing batch throughput vs. minimizing interactive response time) and the nature of the workload. From the simplicity of FCFS, through the optimality-seeking SJF, the interactive fairness of RR, to the adaptable complexity of Multilevel Feedback Queues, each algorithm provides a different way for the OS to conduct the CPU orchestra. Understanding these mechanisms helps appreciate the intricate dance happening inside our computers every millisecond!
