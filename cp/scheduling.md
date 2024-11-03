---
title: Scheduling Jobs - Exchange Arguments and Johnson's Rule
description: The classic single- and two-machine scheduling greedies - shortest processing time, Smith's ratio rule, earliest due date, Moore-Hodgson for the fewest late jobs, and Johnson's rule for the two-machine flow shop - each proved by an adjacent-swap exchange argument.
date: 2024-11-03
draft: false
slug: /cp/scheduling
tags:
  - Competitive Programming
  - Miscellaneous
  - Greedy
---

Scheduling problems ask for the order to run $n$ jobs on one or more machines to optimise some cost. A handful of them have clean greedy answers, and they all share one proof technique: assume an optimal schedule, look at two adjacent jobs in the "wrong" order, show that swapping them does not make the cost worse. Repeat, and the optimum is the sorted order.

## The exchange argument, once

Suppose the cost depends only on the completion times, and jobs $i$ then $j$ run consecutively starting at time $t$. Swapping them changes only their own contributions (everything before is untouched; everything after ends at the same time because $p_i + p_j$ is unchanged). Compare the two local costs; whichever inequality makes "keep $i$ first" better becomes the **sort key**.

## Single machine

### Minimise total completion time — shortest processing time first

Order by processing time $p_i$ ascending. Job in position $k$ contributes its completion time to the sum once, but also delays all $n - k$ jobs after it — so short jobs should go first.

### Minimise weighted total completion time — Smith's rule

Each job has weight $w_i$; minimise $\sum w_i C_i$. Swapping adjacent $i, j$: keeping $i$ first is better iff $w_j p_i \le w_i p_j$, i.e. sort by the ratio $p_i / w_i$ ascending.

```python
from fractions import Fraction

def smith_order(jobs):                       # jobs: list of (p, w)
    order = sorted(range(len(jobs)), key=lambda i: Fraction(jobs[i][0], jobs[i][1]))
    t = cost = 0
    for i in order:
        t += jobs[i][0]
        cost += jobs[i][1] * t
    return order, cost
```

### Minimise maximum lateness — earliest due date

Each job has a due date $d_i$; lateness is $C_i - d_i$; minimise $\max_i (C_i - d_i)$. Sort by $d_i$ ascending. Exchange: if $d_i > d_j$ but $i$ runs first, swapping cannot increase the max lateness.

### Minimise the number of late jobs — Moore-Hodgson

Maximise how many jobs finish by their due date. Process jobs in EDD order, accumulating time; whenever adding a job would make the running time exceed its due date, **drop the longest job scheduled so far** (a max-heap of processing times). The kept set is a maximum-size feasible set.

```python
import heapq

def moore_hodgson(jobs):                     # jobs: list of (p, d); returns count on time
    heap, t = [], 0
    for p, d in sorted(jobs, key=lambda x: x[1]):
        heapq.heappush(heap, -p)
        t += p
        if t > d:
            t += heapq.heappop(heap)        # remove the largest p (adds a negative)
    return len(heap)
```

## Two machines: Johnson's rule

Every job passes through machine 1 then machine 2, taking $a_i$ and $b_i$. All jobs go through the machines in the *same order* (a permutation flow shop); minimise the makespan (time the last job leaves machine 2).

**Johnson's rule.** Split jobs into $A = \{i : a_i \le b_i\}$ and $B = \{i : a_i > b_i\}$. Run all of $A$ first, sorted by $a_i$ ascending; then all of $B$, sorted by $b_i$ descending.

```python
def johnson_order(jobs):                     # jobs: list of (a, b)
    A = sorted((i for i in range(len(jobs)) if jobs[i][0] <= jobs[i][1]),
               key=lambda i: jobs[i][0])
    B = sorted((i for i in range(len(jobs)) if jobs[i][0] > jobs[i][1]),
               key=lambda i: jobs[i][1], reverse=True)
    return A + B

def makespan(order, jobs):
    t1 = t2 = 0
    for i in order:
        a, b = jobs[i]
        t1 += a
        t2 = max(t2, t1) + b
    return t2
```

The intuition: jobs with a small machine-1 time should go early so machine 2 starts working sooner; jobs with a small machine-2 time should go last so machine 1 is not left idle at the end.

## Complexity

Every rule above is a sort plus one linear pass: $O(n \log n)$. Moore-Hodgson's heap keeps it at $O(n \log n)$.

## Common pitfalls

- **Comparing ratios with floating point.** $p_i / w_i \le p_j / w_j$ should be tested as $p_i w_j \le p_j w_i$ with integers to avoid tie-break errors.
- **EDD does not minimise the number of late jobs.** It minimises *maximum* lateness. Fewest-late-jobs is Moore-Hodgson; minimum total tardiness is NP-hard.
- **Johnson's rule is two machines only.** Three-machine flow shop is NP-hard in general (it reduces to two only under special conditions on the middle machine).
- **The flow shop must be a single permutation.** Johnson assumes both machines process jobs in the same order; allowing reordering between machines is a different (harder) problem.
- **Idle time on machine 2.** In `makespan`, machine 2 cannot start job $i$ before it finishes on machine 1 *and* before machine 2 finishes the previous job — hence `max(t2, t1) + b`.

## The keystone

If a schedule's cost depends only on completion times, an adjacent swap changes just the two jobs involved — so the optimal order is whatever sort key makes "no beneficial swap remains". That gives shortest-processing-time, Smith's $p/w$ ratio, and earliest-due-date directly; Moore-Hodgson adds a max-heap to maximise on-time jobs, and Johnson's rule extends the idea to the two-machine flow shop.
