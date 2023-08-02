---
title: Real-Time Systems - Deadlines, Scheduling, and RTOS
description: In a real-time system, a correct result produced too late is a wrong result. Hard, firm, and soft deadlines, the timing model for a task, the scheduling algorithms that meet deadlines (EDF, rate-monotonic) and the priority-inversion traps around shared resources, what makes an RTOS different, and real-time communication and databases.
date: 2023-08-02
draft: false
slug: /miscs/realtime-systems
tags:
  - Real-Time Systems
  - Operating Systems
---

A **real-time system** monitors or controls an external environment, and its correctness depends on *when* a result is produced as well as *what* the result is. An airbag that fires 100 ms after impact computed the right thing at the wrong time, which is to say it failed. This post covers how deadlines are classified, how tasks are modelled and scheduled to meet them, the resource-sharing traps that break the schedule, and what a real-time operating system provides that a general-purpose one does not.

## What makes a system real-time

A real-time system runs a cycle: **sensors** gather data from the environment, the **computing system** processes it, **actuators** effect changes, and feedback loops let it monitor the results. Its properties:

- **Timeliness** — results must land within specified time bounds.
- **Predictability (determinism)** — timing behaviour must be known or tightly bounded, not just fast on average.
- **Reliability and safety** — many real-time systems are safety-critical; a failure can be catastrophic.
- **Concurrency** — many sensors and actuators handled at once.
- **Resource constraints** — often embedded, with limited CPU, memory, and power.

**Types, by deadline criticality:**

| Type | Missing a deadline means | Example |
| --- | --- | --- |
| **Hard** | catastrophic failure; all deadlines must be *guaranteed* | flight control, nuclear plant control, pacemaker |
| **Firm** | the result is worthless, but no catastrophe; drop it | a late video-conference frame |
| **Soft** | degraded utility; the result still has diminished value | video streaming, data acquisition |

## Modelling a task

- **Deadline** — when the task must finish.
- **Period** — for a periodic task, the interval between activations.
- **Release time** — when the task becomes ready.
- **Worst-case execution time (WCET)** — the maximum uninterrupted run time. Determining it accurately is a hard problem in itself.

A periodic task is often written as a tuple $T_i = (P_i, C_i, D_i)$ — period, WCET, relative deadline.

## Scheduling

The scheduler decides which task runs when, so that critical deadlines are met. See [process scheduling](/citadel/operating-system/process-scheduling) for the general case.

- **Preemptive** (a higher-priority task interrupts a running one — most real-time systems) vs **non-preemptive**.
- **Static priority** (fixed before execution) vs **dynamic priority** (changes with system state).

### Clock-driven

Decisions at predetermined times, driven by a clock interrupt.

- **Table-driven** — the whole schedule is computed offline and stored; the dispatcher follows it. Maximally predictable, inflexible.
- **Cyclic (frame-based)** — time is divided into fixed **frames**, each running a pre-defined set of task slices. Simple and deterministic; can waste CPU when execution times vary.
- **Hybrid** — clock-driven for critical periodic tasks, event-driven for the rest.

### Event-driven (priority-driven)

Decisions triggered by arrivals, completions, and resource requests. More flexible, better utilisation.

- **Earliest Deadline First (EDF)** — dynamic priority: the task with the nearest absolute deadline runs. On a uniprocessor with preemption and independent tasks, EDF is **optimal** — if any algorithm can meet all deadlines, EDF can.
- **Rate-Monotonic (RMA)** — static priority for periodic tasks: shorter period means higher priority. Optimal among static-priority algorithms when deadlines equal periods. The Liu–Layland schedulability test: a set of $n$ tasks is schedulable if
  $$\sum_{i=1}^{n} \frac{C_i}{P_i} \le n\left(2^{1/n} - 1\right)$$
  which tends to $\ln 2 \approx 0.693$ as $n$ grows.
- **Deadline-Monotonic (DMA)** — static priority by shortest *relative deadline*; equivalent to RMA when deadlines equal periods.

## Resource sharing and priority inversion

When tasks share a resource, a high-priority task $T_H$ can be blocked waiting for a lock held by a low-priority task $T_L$. If a medium-priority task $T_M$ then preempts $T_L$, $T_H$ is delayed for an unbounded time by a task below it — **priority inversion**. This is what stalled the Mars Pathfinder mission in 1997. Solutions:

- **Priority Inheritance Protocol (PIP)** — while $T_L$ holds a resource $T_H$ needs, $T_L$ temporarily inherits $T_H$'s priority, so $T_M$ cannot preempt it.
- **Highest Locker Protocol (HLP)** — a task locking a resource immediately takes the highest priority of any task that could ever use it.
- **Priority Ceiling Protocol (PCP)** — each resource has a **ceiling** equal to the highest priority of any task that can lock it; a task may lock a resource only if its priority exceeds the ceilings of all resources currently locked by others. PCP also prevents [deadlock](/citadel/operating-system/process-synchronisation) and bounds blocking time.

## Multiprocessor and distributed

- **Partitioning** — assign each task to a fixed processor, then schedule each processor with a uniprocessor algorithm. Optimal partitioning is NP-hard.
- **Global scheduling** — one queue, tasks run on any free processor and may migrate. Better utilisation, more complex.
- **[Distributed](/citadel/miscs/distributed-algorithms)** systems add inter-processor communication overhead and cross-node consistency.

## Fault tolerance

- **Time redundancy** — reserve time to re-execute a task if an error is detected.
- **Hardware redundancy** — a **primary/backup** pair on separate processors, or **N-version programming** (independent implementations run and vote).
- Schedulability analysis must account for these mechanisms.

**Clock synchronisation** in a distributed real-time system: **NTP** over the internet (not precise enough for hard real-time), **PTP (IEEE 1588)** for sub-microsecond synchronisation on a LAN, used in industrial automation and trading.

## Real-time operating systems

An RTOS is built for predictable timing:

- Preemptive, priority-based kernel; fast, low-latency interrupt response; deterministic scheduling; high-resolution timers; inter-task communication (semaphores, mutexes, message queues) with predictable behaviour.

**Are UNIX and Windows real-time?** The standard versions are not — their scheduling, virtual memory, and interrupt handling introduce unpredictable jitter. Real-time variants exist: Linux with the **PREEMPT_RT** patch makes the kernel highly preemptible; some Windows IoT editions add real-time capability. **POSIX.1b** standardises real-time interfaces (priority scheduling, real-time signals, timers), and many RTOSes aim for POSIX compliance.

**Examples:** pSOS and VRTX (historical, aerospace/defence), **QNX** (microkernel, automotive and medical), **FreeRTOS** and **Zephyr** (open-source, microcontrollers), **VxWorks** (aerospace, networking), ThreadX.

**Benchmarking an RTOS** measures interrupt latency, context-switch time, jitter (variation in periodic event timing), and API call times.

## Real-time communication

**Quality of Service** provides different guarantees to different flows. Parameters: bandwidth, delay, jitter, loss. Network models:

- **IntServ** — applications reserve resources along a path with **RSVP**. Strong guarantees, hard to scale.
- **DiffServ** — packets are marked with a priority class (DSCP); routers apply per-hop behaviours. More scalable, coarser.

**On a LAN**, standard Ethernet (CSMA/CD) is non-deterministic. Deterministic options: the historical Token Bus (802.4), and modern real-time Ethernet — **TTEthernet**, **EtherCAT**, **PROFINET IRT**, and **IEEE 802.1 TSN** (Time-Sensitive Networking). See [high-speed networks](/citadel/computer-networks/high-speed-networks).

**Over packet-switched networks**, guarantees are harder (variable queuing, congestion). Approaches: routing that weighs delay, jitter, and bandwidth rather than hop count; RSVP reservations; and rate control (token bucket, leaky bucket) to smooth traffic.

## Real-time databases

Databases where data has timing constraints and transactions have deadlines — trading systems, process control, network management.

- **Temporal data** — **valid time** (when a value is true in the world), **transaction time** (when it is stored), and **data freshness** (a value may go stale, so validity can carry a deadline).
- **Concurrency control** — standard two-phase locking ignores deadlines. Alternatives: **priority-based locking** (an earlier-deadline transaction preempts locks) and **deadline-aware optimistic concurrency** (validate at commit; abort the lower-priority transaction on conflict). See [transaction processing](/citadel/dbms/transaction-processing).
- **Implementations** — specialised RTDBs (eXtremeDB), and in-memory databases (Oracle TimesTen, SAP HANA, Redis) whose RAM-based operation gives the predictable low latency many real-time applications need.

## Key takeaways

- A real-time system's correctness includes timing; deadlines are **hard** (guaranteed), **firm** (drop if late), or **soft** (degraded value).
- On a uniprocessor, **EDF** is optimal with dynamic priorities; **rate-monotonic** is optimal with static priorities, subject to the Liu–Layland utilisation bound.
- Shared resources invite **priority inversion**; the priority ceiling protocol bounds blocking and prevents deadlock.
- An **RTOS** trades throughput for predictability — preemptive priority scheduling, low interrupt latency, bounded jitter — and the same determinism requirement extends to real-time networking (TSN, TTEthernet) and databases.
