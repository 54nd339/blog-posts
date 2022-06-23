---
title: CPU Scheduling - Choosing Which Process Runs Next
description: How the short-term scheduler picks from the ready queue - FCFS and the convoy effect, shortest-job-first and burst prediction, round robin and quantum sizing, multilevel feedback queues, plus thread and multiprocessor scheduling, real-time algorithms, and how you evaluate a policy.
date: 2022-06-23
draft: false
slug: /operating-system/process-scheduling
tags:
  - Operating Systems
  - Scheduling
  - Processes
---

Processes alternate between bursts of CPU work and stretches of waiting for I/O. While one process waits, another could be running — so the moment a process blocks or its time runs out, the **short-term scheduler** picks the next one from the [ready queue](/citadel/operating-system/process-thread). Which one it picks, by which rule, is CPU scheduling, and the choice trades throughput against responsiveness against fairness.

This post walks the scheduling algorithms in the order they were invented, each fixing a flaw in the last, then covers threads, multiple cores, real-time deadlines, and how you tell which policy is right.

## The criteria

Every algorithm is judged on the same metrics:

- **CPU utilisation** — keep the CPU busy.
- **Throughput** — processes completed per unit time.
- **Turnaround time** — submission to completion.
- **Waiting time** — total time in the ready queue.
- **Response time** — submission to first output, the one that matters for interactive use.

Scheduling is **nonpreemptive** if a process keeps the CPU until it blocks or exits, **preemptive** if the OS can take it away. The **dispatcher** performs the actual switch, at a small cost called **dispatch latency**.

## The algorithms

### First-Come, First-Served

The ready queue is a plain FIFO; the process that arrived first runs first, to completion. Nonpreemptive.

With bursts P1=24, P2=3, P3=3 arriving in that order, the Gantt chart is `P1(0-24) P2(24-27) P3(27-30)`, waiting times 0, 24, 27, average **17 ms**. Reorder arrivals to P2, P3, P1 and the average drops to **3 ms**. FCFS is simple and starvation-free, but its performance swings wildly with arrival order, and it suffers the **convoy effect**: short jobs queue behind one long CPU-bound job, and I/O devices sit idle waiting for those short jobs. Unusable for interactive systems.

### Shortest-Job-First

Give the CPU to the process with the shortest *next* CPU burst; break ties by FCFS. The preemptive version, **Shortest-Remaining-Time-First** (SRTF), preempts the running process when an arrival has a shorter burst than its remaining time.

SJF provably minimises average waiting time, and SRTF is optimal among preemptive policies. The problem is you cannot know the next burst length, so you **estimate** it by exponential averaging of past bursts:

$$\tau_{n+1} = \alpha\, t_n + (1 - \alpha)\, \tau_n$$

with $t_n$ the last actual burst, $\tau_n$ the last prediction, and $\alpha \in [0,1]$ setting how fast the estimate follows recent history ($\alpha = 0.5$ is common). SJF also **starves** long jobs when short ones keep arriving.

For P1(arrival 0, burst 7), P2(2, 4), P3(4, 1), P4(5, 4), SRTF produces `P1(0-2) P2(2-4) P3(4-5) P2(5-7) P4(7-11) P1(11-16)` — P1 is preempted twice — for an average waiting time of 3 ms, below what nonpreemptive SJF gives on the same input, at the cost of more context switches.

### Priority scheduling

Each process gets a priority; the highest-priority ready process runs. Equal priorities go FCFS. Priorities can be internal (from burst time, memory use) or external (importance, payment). Preemptive or nonpreemptive.

For bursts/priorities P1(10, 3), P2(1, 1), P3(2, 4), P4(1, 5), P5(5, 2) with lower number = higher priority, nonpreemptive order is `P2(0-1) P5(1-6) P1(6-16) P3(16-18) P4(18-19)`.

The failure mode is **starvation** of low-priority processes. The fix is **aging** — raise a process's priority the longer it waits. (SJF is priority scheduling with priority = inverse predicted burst.)

### Round Robin

Built for time-sharing. A fixed **time quantum** $q$; the ready queue is circular; each process runs at most $q$ before a timer interrupt moves it to the tail. Always preemptive.

With P1=24, P2=3, P3=3 and $q = 4$: `P1 P2 P3 P1 P1 P1 P1 P1`, and P1 finishes with a 2 ms final slice. RR is fair, prevents starvation (each process gets $1/n$ of the CPU in $q$-sized chunks), and gives good response time. Turnaround is usually worse than SJF, and $q$ is delicate: too large and it degrades to FCFS; too small and context-switch overhead dominates. Rule of thumb: $q$ well above the switch cost (10–100 ms versus microseconds), with ~80% of bursts shorter than $q$.

### Multilevel queue

Split processes into classes — interactive vs batch, system vs user — each with its own ready queue and its own algorithm (RR for interactive, FCFS for batch). Processes are assigned to a queue permanently. Scheduling *between* queues is either **fixed priority** (serve all of a higher queue before any of a lower — risks starving low queues) or **time slice** (each queue gets a percentage of CPU time).

### Multilevel feedback queue

The general case: like multilevel queue, but processes **migrate** between queues based on behaviour. Use too much CPU and you drop to a lower-priority queue; wait too long and you are promoted (aging). Configured by the number of queues, each queue's algorithm, and the promotion/demotion rules.

A typical setup: Q0 is RR with $q = 8$, Q1 is RR with $q = 16$, Q2 is FCFS. New jobs enter Q0; a job that does not finish its quantum drops a level. Serve Q0, then Q1, then Q2, with higher-queue arrivals preempting lower ones. It adapts to whether a process is I/O-bound or CPU-bound, but it is the hardest to tune.

### Computing the numbers

The Gantt-chart arithmetic above is easy to get wrong by hand and easy to script:

```python
def avg_waiting(procs, policy="fcfs", quantum=4):
    # procs: list of (name, arrival, burst)
    procs = sorted(procs, key=lambda p: p[1])
    rem = {p[0]: p[2] for p in procs}
    arr = {p[0]: p[1] for p in procs}
    burst = {p[0]: p[2] for p in procs}
    t, done, completion = 0, set(), {}
    order = [p[0] for p in procs]
    while len(done) < len(procs):
        ready = [n for n in order if n not in done and arr[n] <= t and rem[n] > 0]
        if not ready:
            t += 1
            continue
        if policy == "fcfs":
            n = ready[0]; t += rem[n]; rem[n] = 0
        elif policy == "srtf":
            n = min(ready, key=lambda x: rem[x]); t += 1; rem[n] -= 1
        elif policy == "rr":
            n = ready[0]
            run = min(quantum, rem[n]); t += run; rem[n] -= run
            order.append(order.pop(order.index(n)))  # rotate to tail
        if rem[n] == 0 and n not in done:
            done.add(n); completion[n] = t
    waits = [completion[n] - arr[n] - burst[n] for n in burst]
    return sum(waits) / len(waits)


P = [("P1", 0, 24), ("P2", 0, 3), ("P3", 0, 3)]
print(avg_waiting(P, "fcfs"))                                    # 17.0
print(avg_waiting([("P2", 0, 3), ("P3", 0, 3), ("P1", 0, 24)], "fcfs"))   # 3.0
print(round(avg_waiting(P, "rr", quantum=4), 2))                 # 5.67
S = [("P1", 0, 7), ("P2", 2, 4), ("P3", 4, 1), ("P4", 5, 4)]
print(avg_waiting(S, "srtf"))                                    # 3.0 (Gantt: P1 P2 P3 P2 P4 P1)
```

## Thread scheduling

Where the OS has **kernel threads**, it schedules *threads*, not processes. With **user threads** (many-to-one or many-to-many), the kernel schedules processes and the thread library picks a user thread — **process contention scope** (PCS), threads competing for their process's slice. With **kernel threads** (one-to-one), the kernel schedules threads directly — **system contention scope** (SCS), all threads competing system-wide. SCS gives true parallelism and stops one blocking thread from freezing the process; modern systems use it.

## Multiprocessor scheduling

With multiple cores, the scheduler also picks *which* core.

- **Asymmetric multiprocessing** — one master core does all scheduling; simple, but a bottleneck.
- **Symmetric multiprocessing (SMP)** — every core schedules itself, from a shared or a per-core ready queue. The standard approach.

SMP challenges:

- **Processor affinity** — a thread runs faster on the core whose cache still holds its data. **Soft affinity** prefers the last core but allows migration; **hard affinity** pins a thread to a set of cores.
- **Load balancing** — keep every core busy. **Push migration** moves threads off overloaded cores; **pull migration** has an idle core take work from a busy one. This works against affinity, so systems balance only on significant imbalance and otherwise use per-core queues.

## Real-time scheduling

When meeting a deadline matters as much as the result:

- **Hard real-time** — a missed deadline is a failure (flight control). **Soft real-time** — misses are tolerable but undesirable (media playback).
- The focus is minimising **latency**: **interrupt latency** (arrival to handler start) and **dispatch latency** (stopping one task, starting another). Preemptible kernels help.
- **Rate-monotonic scheduling** — static priorities, higher for shorter-period (more frequent) tasks. Optimal among static-priority schemes under its assumptions.
- **Earliest-Deadline-First** — dynamic priority, the nearest deadline wins. Optimal when the system is not overloaded.
- **Proportional share** — each task gets a fixed fraction of CPU time.

## Evaluating a policy

- **Deterministic modelling** — run a fixed workload through each algorithm and compute the metrics. Exact, fast, but specific to that workload.
- **Queuing models** — describe arrival and service rates with distributions and compute averages analytically. General, but leans on unrealistic assumptions.
- **Simulation** — model the system and drive it with trace data or generated events. Flexible and accurate, but costly to build and run.

Real selection combines simulation with implementing the algorithm and measuring it on the live system.

## The one idea to keep

There is no best scheduler, only a best fit for a goal: FCFS is fair and terrible for interactivity, SJF is optimal for waiting time and impossible to run exactly, round robin is responsive and fair at the cost of turnaround. The multilevel feedback queue wins in practice because it does not commit — it watches each process and moves it to the queue that suits how it is actually behaving.
