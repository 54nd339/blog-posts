---
title: Deadlock - Detecting, Avoiding, and Recovering From Gridlock
description: "When a set of processes each wait forever for a resource another holds - the four conditions that must all hold, resource allocation graphs, and the four strategies: prevention by breaking a condition, avoidance with the Banker's algorithm, detection and recovery, and deliberately ignoring it."
date: 2022-06-26
draft: false
slug: /operating-system/process-synchronisation
tags:
  - Operating Systems
  - Processes
  - Scheduling
---

Process A holds a lock on file 1 and asks for file 2. Process B holds file 2 and asks for file 1. Neither will release what it has until it gets what it wants, so both wait forever. That is a **deadlock** — a set of processes, each blocked waiting for a resource held by another process in the same set. It comes up wherever [concurrent processes](/citadel/operating-system/process-thread) share reusable resources, and the OS has four quite different ways to deal with it.

## The model

The system has **resource types** $R_1, \dots, R_m$ (CPU cycles, memory, printers, locks), each with some number of interchangeable **instances**. A process uses a resource in three steps: **request** it (block if unavailable), **use** it, **release** it.

## The four conditions

A deadlock can arise only when all four of these hold at once:

1. **Mutual exclusion** — at least one resource is non-sharable; only one process holds an instance at a time.
2. **Hold and wait** — a process holds at least one resource while waiting for another.
3. **No preemption** — resources are released only voluntarily, never taken by force.
4. **Circular wait** — there is a cycle of processes $P_0, P_1, \dots, P_n$ where each $P_i$ waits for a resource held by $P_{(i+1)}$, and $P_n$ waits for one held by $P_0$.

Break any one and deadlock is impossible. Every strategy below is an attack on one or more of these, or an acceptance that they might all hold.

### Resource allocation graphs

Draw processes as circles and resource types as boxes (with a dot per instance). A **request edge** $P_i \rightarrow R_j$ means $P_i$ is waiting for an instance of $R_j$; an **assignment edge** $R_j \rightarrow P_i$ means $P_i$ holds one.

- **No cycle** — no deadlock.
- **Cycle, all resources single-instance** — deadlock, definitely.
- **Cycle, some resources multi-instance** — deadlock possible, not certain; a process outside the cycle might free an instance that breaks it.

## The four strategies

1. **Prevention** — structurally guarantee one of the four conditions can never hold.
2. **Avoidance** — allow the conditions but use knowledge of future needs to refuse any request that could lead to deadlock.
3. **Detection and recovery** — let deadlocks happen, find them, break them.
4. **Ignore it** — assume they are rare enough that a hung system and a reboot is an acceptable cost. This is what Windows and Linux do for general user processes.

## Prevention

Attack a condition:

- **Mutual exclusion** — make resources sharable where possible (read-only files). Many resources are inherently non-sharable, so this rarely fully applies.
- **Hold and wait** — require a process to request *all* its resources before it starts, or to hold *none* when it requests more (release everything, then re-request). Both waste resources that sit idle after early allocation, and both can starve a process that needs several popular resources.
- **No preemption** — if a process holding resources requests one it cannot get, force it to release everything it holds; it restarts when it can reacquire the whole set. Hard to apply to resources like printers mid-job.
- **Circular wait** — impose a total order $F$ on resource types and require each process to request them in increasing order. A cycle would imply $F(R_{k_0}) < F(R_{k_1}) < \dots < F(R_{k_0})$, a contradiction. This one is practical and widely used for kernel locks.

## Avoidance

Avoidance needs each process to declare, in advance, the **maximum** it might ever need of each resource. The system then only enters **safe states** — those with a **safe sequence** $\langle P_1, \dots, P_n \rangle$ in which each $P_i$'s remaining needs can be met by the currently free resources plus everything held by the processes before it. A safe state guarantees everyone can finish; an **unsafe state** does not *guarantee* deadlock but allows it.

For single-instance resources, extend the resource allocation graph with **claim edges** (dashed, "$P_i$ might request $R_j$ later"). Grant a request only if turning its request edge into an assignment edge creates no cycle, counting claim edges.

### The Banker's algorithm

For multi-instance resources. With $n$ processes and $m$ resource types, track: `Available[m]` (free instances), `Max[n][m]` (each process's declared maximum), `Allocation[n][m]` (currently held), and `Need = Max - Allocation`.

The **safety check** asks whether a state is safe: repeatedly find a process whose `Need` fits in the current work vector, pretend it runs and releases its `Allocation`, and add that back. If every process can be finished this way, the state is safe.

The **resource-request check** runs when $P_i$ requests: verify the request is within `Need` and within `Available`, tentatively grant it, run the safety check on the hypothetical state, and only commit if it stays safe — otherwise $P_i$ waits.

```python
def is_safe(available, allocation, need):
    n, m = len(allocation), len(available)
    work = list(available)
    finish = [False] * n
    order = []
    changed = True
    while changed:
        changed = False
        for i in range(n):
            if not finish[i] and all(need[i][j] <= work[j] for j in range(m)):
                for j in range(m):
                    work[j] += allocation[i][j]
                finish[i] = True
                order.append(i)
                changed = True
    return all(finish), order


def request_ok(available, allocation, need, i, req):
    if any(req[j] > need[i][j] for j in range(len(req))):
        raise ValueError("request exceeds declared maximum")
    if any(req[j] > available[j] for j in range(len(req))):
        return False, "must wait: resources unavailable"
    avail2 = [available[j] - req[j] for j in range(len(req))]
    alloc2 = [row[:] for row in allocation]
    need2 = [row[:] for row in need]
    for j in range(len(req)):
        alloc2[i][j] += req[j]
        need2[i][j] -= req[j]
    safe, order = is_safe(avail2, alloc2, need2)
    return (safe, f"grant; safe sequence {order}") if safe else (False, "deny: unsafe")


# Classic textbook state: 5 processes, 3 resource types (A=10, B=5, C=7)
available  = [3, 3, 2]
allocation = [[0,1,0],[2,0,0],[3,0,2],[2,1,1],[0,0,2]]
maximum    = [[7,5,3],[3,2,2],[9,0,2],[2,2,2],[4,3,3]]
need = [[maximum[i][j] - allocation[i][j] for j in range(3)] for i in range(5)]

print(is_safe(available, allocation, need))      # (True, [1, 3, 4, 0, 2])
print(request_ok(available, allocation, need, 1, [1, 0, 2]))  # (True, ...)
print(request_ok(available, allocation, need, 4, [3, 3, 0]))  # (False, 'deny: unsafe')
```

Avoidance uses resources better than prevention, but declaring a maximum upfront is often impractical.

## Detection

If you neither prevent nor avoid, you must be able to spot a deadlock after the fact.

- **Single-instance resources** — maintain a **wait-for graph** (processes only; $P_i \rightarrow P_j$ if $P_i$ waits for something $P_j$ holds) and run cycle detection periodically. A cycle is a deadlock.
- **Multi-instance resources** — a Banker-style sweep using `Available`, `Allocation`, and `Request` (current requests, not declared maxima). Any process that cannot be finished by the sweep is deadlocked.

How often to run detection is a trade: frequently means early detection but overhead; rarely means deadlocks grow before they are caught.

## Recovery

Once detected:

- **Process termination** — abort all deadlocked processes (fast, wasteful), or abort one at a time re-running detection between (slower, less lost work). Victim selection weighs priority, run time, progress, resources held and needed, and interactive vs batch.
- **Resource preemption** — take resources from processes and reassign until the cycle breaks. This needs **rollback** of the victim to a safe checkpoint, and a guard against always picking the same victim (**starvation**) — for instance, factoring rollback count into the cost.

## The combined reality

Real systems mix strategies by subsystem: resource ordering (prevention) for kernel locks; avoidance only where maximum needs are genuinely known; detection and recovery in database systems, where lock deadlocks are common and aborting a transaction is cheap; and the ostrich approach for everything else.

## The one idea to keep

Deadlock needs all four conditions, so every real defence is a choice about which one to sacrifice and when: prevention gives one up permanently and pays in utilisation; avoidance gives up nothing but needs to see the future; detection gives up nothing and pays in cleanup. General-purpose operating systems mostly conclude the bookkeeping is not worth it and let the rare deadlock be a reboot.
