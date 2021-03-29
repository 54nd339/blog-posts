---
title: Graph Coloring - Backtracking to an m-Coloring
description: Assign colors to a graph's vertices so no edge joins two of the same color. Finding the fewest colors is NP-hard; deciding whether m colors suffice is a backtracking search that colors vertices in order and rewinds on conflict.
date: 2021-03-29
draft: false
slug: /algorithms/GraphColoring
tags:
  - Algorithms
  - Backtracking
---

Coloring a map so no two bordering countries share a color is the picture; the **graph coloring problem** is the abstraction. Given an undirected graph $G = (V, E)$, assign each vertex a color so that every edge joins two different colors. The smallest number of colors that works is the graph's **chromatic number** $\chi(G)$.

Finding $\chi(G)$ is NP-hard. But for a fixed $m$, deciding *whether* an $m$-coloring exists is a clean backtracking search, and running it for $m = 1, 2, 3, \ldots$ finds the chromatic number.

## The idea

Order the vertices $0, 1, \ldots, |V|-1$ and color them in that order:

1. For the current vertex, try each color $1$ to $m$.
2. Skip a color if any already-colored neighbour has it.
3. If a color is safe, assign it and recurse on the next vertex.
4. If no color works, return — the previous vertex will try its next color.
5. When every vertex is colored, report success.

An always-safe upper bound for $m$ is $\Delta + 1$, one more than the maximum degree: each vertex has at most $\Delta$ neighbours, so at least one of $\Delta + 1$ colors is free. (Brooks' theorem sharpens this: $\chi(G) \le \Delta$ unless $G$ is a complete graph or an odd cycle.)

## The code

```python
def graph_coloring(adj, m):
    n = len(adj)
    color = [0] * n                       # 0 = uncolored

    def safe(v, c):
        return all(color[u] != c for u in adj[v])

    def paint(v):
        if v == n:
            return True
        for c in range(1, m + 1):
            if safe(v, c):
                color[v] = c
                if paint(v + 1):
                    return True
                color[v] = 0              # backtrack
        return False

    return color if paint(0) else None


def chromatic_number(adj):
    for m in range(1, len(adj) + 1):
        if graph_coloring(adj, m):
            return m


# a 5-cycle: needs 3 colors (odd cycle)
cycle5 = [[1, 4], [0, 2], [1, 3], [2, 4], [3, 0]]
assert chromatic_number(cycle5) == 3
# a square (4-cycle): 2 colors
square = [[1, 3], [0, 2], [1, 3], [2, 0]]
assert chromatic_number(square) == 2
```

## Cost

With $m$ colors and $|V|$ vertices, the search tree has up to $m^{|V|}$ leaves, and each safety check costs $O(\deg v)$: worst case $O(m^{|V|} \cdot |V|)$. That exponential is expected — this is an NP-hard problem. Space is $O(|V|)$ for the color array and the recursion.

## Where it's used

The abstraction is everywhere two things "can't happen together":

- **Exam scheduling** — vertices are exams, edges join exams a student shares; colors are time slots.
- **Register allocation** — vertices are variables, edges join variables live at the same time; colors are CPU registers.
- **Frequency assignment** — vertices are transmitters, edges join ones close enough to interfere; colors are frequencies.

[Sudoku](/citadel/algorithms/SudokuSolver) is a graph-coloring instance with 81 vertices and 9 colors. The backtracking structure is shared with [N-queens](/citadel/algorithms/NQueens) and the [Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle), and the NP-hardness places it in the landscape of [complexity classes](/citadel/algorithms/ComplexityClasses).
