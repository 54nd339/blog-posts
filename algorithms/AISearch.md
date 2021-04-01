---
title: AI Search Algorithms - Uninformed, Informed, and Local Search
description: How an AI explores a space of states to reach a goal. Uninformed methods (BFS, DFS, uniform-cost) search blindly; informed methods (A*, greedy best-first) use a heuristic; local search (hill climbing, simulated annealing, genetic algorithms) improves one state at a time.
date: 2021-04-01
draft: false
slug: /algorithms/AISearch
tags:
  - Algorithms
  - AI
  - Search
---

Many AI problems reduce to search: from a start state, apply actions, and find a state that satisfies a goal — a robot's path, a puzzle's solution, a game's next move. The search space is usually far too large to enumerate, so the algorithms differ in how they decide what to look at next. Three families: **uninformed** (no domain knowledge), **informed** (guided by a heuristic), and **local** (improve a single current state).

The graph-search family is one loop with a swappable frontier:

```python
import heapq, itertools

def best_first(start, goal, neighbors, priority):
    tie = itertools.count()
    frontier = [(priority(start, 0), next(tie), start, 0, [start])]
    best_g = {start: 0}
    while frontier:
        _, _, node, g, path = heapq.heappop(frontier)
        if node == goal:
            return path, g
        for nxt, cost in neighbors(node):
            ng = g + cost
            if ng < best_g.get(nxt, float("inf")):
                best_g[nxt] = ng
                heapq.heappush(frontier, (priority(nxt, ng), next(tie), nxt, ng, path + [nxt]))
    return None, float("inf")
```

Change `priority` and you change the algorithm.

## Uninformed search

**Breadth-first search (BFS)** expands the shallowest node first — a FIFO frontier, i.e. `priority = lambda node, g: g` counting *edges*. It's **complete** (finds a solution if one exists) and **optimal** on unweighted graphs (fewest edges). Time and space are $O(b^d)$ for branching factor $b$ and solution depth $d$ — the space cost is what hurts.

**Depth-first search (DFS)** expands the deepest node first — a LIFO frontier. Space drops to $O(bm)$ for maximum depth $m$ (or $O(m)$ recursively), but it's **not optimal** and not complete on infinite spaces. It finds *a* path, not the shortest. Backtracking solvers like [N-queens](/citadel/algorithms/NQueens) and [Sudoku](/citadel/algorithms/SudokuSolver) are DFS with pruning.

**Uniform-cost search (UCS)** expands the node with the lowest path cost $g(n)$ — `priority = lambda node, g: g` over real weights. This is exactly [Dijkstra's algorithm](/citadel/algorithms/PathFinding) with a goal test; complete and optimal for non-negative costs.

## Informed search

These use a **heuristic** $h(n)$ estimating the cost from $n$ to the goal.

**A\* search** orders the frontier by $f(n) = g(n) + h(n)$ — cost so far plus estimated cost remaining:

```python
def astar(start, goal, neighbors, h):
    return best_first(start, goal, neighbors,
                      priority=lambda node, g: g + h(node))
```

A\* is complete and optimal when $h$ is **admissible** (never overestimates the true remaining cost) and, for graph search, **consistent** ($h(u) \le \text{cost}(u, v) + h(v)$). With $h \equiv 0$ it degenerates to UCS; with a perfect $h$ it walks straight to the goal.

**Greedy best-first search** orders by $h(n)$ alone, ignoring $g(n)$. Often faster than A\*, but **not optimal** and not complete — it chases the heuristic into dead ends.

**Iterative-deepening A\* (IDA\*)** runs repeated depth-first searches, each bounded by an $f$-cost limit that rises to the smallest $f$ that exceeded the last limit. It keeps A\*'s optimality with DFS's $O(d)$ space — the tool when A\*'s frontier won't fit in memory (the 15-puzzle, large maps).

## Local search

When only the goal state matters, not the path, keep one current state and nudge it.

**Hill climbing** repeatedly moves to the best neighbour, stopping at a peak:

```python
def hill_climb(state, neighbors, value):
    while True:
        best = max(neighbors(state), key=value, default=None)
        if best is None or value(best) <= value(state):
            return state
        state = best
```

Simple and fast, but it gets stuck on local optima, plateaus, and ridges.

**Simulated annealing** escapes local optima by sometimes accepting a worse move, with probability $e^{-\Delta/T}$ that shrinks as the "temperature" $T$ cools:

```python
import math, random

def simulated_annealing(state, random_neighbor, value, schedule):
    for t in range(1, 100000):
        T = schedule(t)
        if T <= 1e-12:
            return state
        nxt = random_neighbor(state)
        delta = value(nxt) - value(state)          # higher value is better
        if delta > 0 or random.random() < math.exp(delta / T):
            state = nxt
    return state
```

Cool slowly enough and it converges on the global optimum with probability approaching 1. Used for the travelling salesman problem and circuit layout.

**Genetic algorithms** keep a *population* of candidate states and evolve it: select parents by fitness, **crossover** them into children, **mutate** slightly, repeat over generations. Probabilistic, good on large messy spaces (scheduling, parameter tuning, evolving network architectures), with no optimality guarantee.

## Key takeaways

- **Uninformed search** (BFS, DFS, UCS) explores with no domain knowledge. BFS is optimal on edge count, UCS on weighted cost, DFS neither but cheap on memory.
- **Informed search** (A\*, greedy best-first) uses a heuristic. A\* is optimal with an admissible, consistent heuristic; greedy best-first trades that guarantee for speed; IDA\* trades memory for time.
- **Local search** (hill climbing, simulated annealing, genetic algorithms) improves a single state and suits optimization where the path doesn't matter.
- The choice turns on whether you need optimality, whether you have a good heuristic, and how much memory you can spend.
