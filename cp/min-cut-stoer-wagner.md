---
title: Global Minimum Cut - The Stoer-Wagner Algorithm
description: Finding the cheapest way to split an undirected weighted graph into two non-empty parts without fixing a source or sink - the maximum-adjacency ordering, the cut-of-the-phase, and merging the last two vertices, all in O(V^3).
date: 2024-10-16
draft: false
slug: /cp/min-cut-stoer-wagner
tags:
  - Competitive Programming
  - Graphs
  - Network Flow
---

The [max-flow min-cut theorem](/citadel/cp/maximum-flow) gives the minimum cut **separating a fixed $s$ from a fixed $t$**. The *global* minimum cut asks for the cheapest split of the whole graph into two non-empty sides, over all $\binom{V}{2}$ possible pairs. You could run max flow for every pair, but Stoer-Wagner finds it directly in $O(V^3)$, with nothing but an adjacency matrix and no flow machinery at all.

## The problem

Given an undirected graph with non-negative edge weights, partition the vertices into two non-empty sets $A$ and $B$ minimising the total weight of edges crossing between them. (Directed graphs still need flow; Stoer-Wagner is for the undirected case.)

Example: a 4-cycle $0\text{-}1\text{-}2\text{-}3\text{-}0$ with all weights $1$ plus a chord $0\text{-}2$ of weight $5$. Every cut crosses at least two cycle edges; the minimum cut has weight $2$, e.g. $\{0, 3\}$ vs $\{1, 2\}$.

## The idea: minimum cut phase + merge

The algorithm runs $V - 1$ **phases**. Each phase does two things: it computes one candidate cut (the "cut-of-the-phase"), and it merges two vertices so the graph shrinks by one.

**Maximum-adjacency ordering.** Start a phase with an arbitrary vertex in set $A$. Repeatedly add to $A$ the vertex most strongly connected to it — the one maximising $\sum_{a \in A} w(v, a)$ — updating those sums as you go, until every vertex is in $A$. Call the last two vertices added $s$ (second-to-last) and $t$ (last).

**Cut-of-the-phase.** When $t$ is added it is the only vertex outside $A$, so its connection sum equals $w(\{t\}, V \setminus \{t\})$ — the weight of the cut that isolates $t$. Crucially, this is provably a *minimum $s$-$t$ cut* of the current (merged) graph. Record its value.

**Merge.** Contract $s$ and $t$ into a single vertex: add $t$'s edge weights onto $s$'s (and symmetrically). Any global min cut of the original graph either separates $s$ from $t$ — and then the cut-of-the-phase already found it — or keeps them together, in which case the merged graph still contains it. So no minimum cut is ever lost.

After $V - 1$ phases the graph is a single vertex; the smallest cut-of-the-phase seen is the global minimum cut.

## Algorithm

```python
import math

def stoer_wagner(n, w):
    w = [row[:] for row in w]                     # w[i][j] symmetric, w[i][i] = 0
    vertices = list(range(n))
    groups = [[i] for i in range(n)]             # original vertices merged into each
    best, best_side = math.inf, None

    while len(vertices) > 1:
        in_a = [False] * n
        conn = [0] * n                          # sum of weights into A
        prev = last = -1
        cut_of_phase = 0
        for _ in range(len(vertices)):
            sel = -1
            for v in vertices:                  # most tightly connected vertex not in A
                if not in_a[v] and (sel == -1 or conn[v] > conn[sel]):
                    sel = v
            cut_of_phase = conn[sel]            # weight of {sel} to current A
            in_a[sel] = True
            prev, last = last, sel
            for v in vertices:
                if not in_a[v]:
                    conn[v] += w[sel][v]

        if cut_of_phase < best:
            best = cut_of_phase
            best_side = groups[last][:]

        for v in vertices:                       # merge 'last' into 'prev'
            w[prev][v] += w[last][v]
            w[v][prev] += w[v][last]
        groups[prev].extend(groups[last])
        vertices.remove(last)

    return best, best_side
```

`best_side` is one side of the cut (the original vertices behind the last-added super-vertex of the best phase); the other side is everything else.

## Complexity

- **Time:** $V - 1$ phases, each an $O(V^2)$ maximum-adjacency ordering (with an array scan for the max) → $O(V^3)$. A Fibonacci-heap priority queue drops a phase to $O(E + V \log V)$ for $O(VE + V^2 \log V)$ overall, rarely worth it in contests.
- **Space:** $O(V^2)$ for the matrix.

## Common pitfalls

- **Reading `cut_of_phase` at the wrong moment.** It is `conn[sel]` *at the instant `sel` is chosen*, before its own edges are folded into other vertices' sums. Capture it every iteration; after the loop it holds the value for the final vertex.
- **Merging the wrong pair.** Contract the **last two** vertices of the ordering (`prev` and `last`), not the first vertex with anything.
- **Directed graphs.** Stoer-Wagner assumes symmetric weights. For a directed global min cut, you still need $2(V-1)$ max-flow runs (fix one vertex, alternate it as source and sink against every other).
- **Negative weights.** The maximum-adjacency argument needs non-negative weights. Negative edges break the correctness proof.
- **Isolated vertices / disconnected graph.** The answer is then $0$ (put one component on each side); the algorithm returns this naturally since some cut-of-the-phase will be $0$.

## The keystone

Order the vertices by maximum adjacency; the last vertex's connection weight is a minimum cut between it and the second-to-last vertex of the current graph. Record it, merge those two, and repeat $V - 1$ times — every global minimum cut either separates some merged pair (caught as its cut-of-the-phase) or survives all the merges, so the smallest phase value is the global minimum cut, in $O(V^3)$ with no flow.
