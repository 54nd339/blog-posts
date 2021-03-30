---
title: Hamiltonian Cycle - Backtracking a Tour of Every Vertex
description: Does a graph contain a cycle that visits every vertex exactly once and returns to the start? Unlike shortest paths, this has no known efficient algorithm. Backtracking builds the tour vertex by vertex and rewinds at every dead end.
date: 2021-03-30
draft: false
slug: /algorithms/HamiltonianCycle
tags:
  - Algorithms
  - Backtracking
---

A **Hamiltonian path** visits every vertex of a graph exactly once. A **Hamiltonian cycle** is one that also returns to where it started. The question "does this graph have a Hamiltonian cycle?" sounds like the shortest-path question, but it's a different world: shortest paths are [solved in polynomial time](/citadel/algorithms/PathFinding), while deciding whether *any* Hamiltonian cycle exists is NP-complete. The travelling salesman problem is the weighted optimisation version — the shortest Hamiltonian cycle — and it inherits the hardness.

## The idea

Fix the start at vertex 0 (a cycle has no distinguished start, so this loses nothing) and grow a path one position at a time:

1. At position `pos`, try each vertex not yet in the path.
2. A candidate is valid only if there's an edge from the last path vertex to it.
3. If valid, place it and recurse on `pos + 1`.
4. When the path holds all $n$ vertices, check for an edge from the last one back to vertex 0. If it's there, that's a Hamiltonian cycle.
5. Any dead end returns, and the caller tries its next candidate — the backtrack.

## The code

```python
def hamiltonian_cycle(adj):             # adj: adjacency sets, vertices 0..n-1
    n = len(adj)
    path = [0]
    used = {0}

    def extend(pos):
        if pos == n:
            return 0 in adj[path[-1]]   # edge back to the start closes the cycle
        for v in range(1, n):
            if v not in used and v in adj[path[-1]]:
                path.append(v)
                used.add(v)
                if extend(pos + 1):
                    return True
                path.pop()              # backtrack
                used.remove(v)
        return False

    return path + [0] if extend(1) else None


# square with one diagonal: 0-1-2-3-0 is a Hamiltonian cycle
g1 = [{1, 3}, {0, 2}, {1, 3}, {2, 0}]
assert hamiltonian_cycle(g1) == [0, 1, 2, 3, 0]
# a "T" shape has a leaf vertex: no Hamiltonian cycle
g2 = [{1}, {0, 2, 3}, {1}, {1}]
assert hamiltonian_cycle(g2) is None
```

## Cost

With the start fixed, the search considers orderings of the other $n-1$ vertices: $O((n-1)!)$ in the worst case, times $O(n)$ per feasibility check. The edge check prunes — a candidate with no edge from the current end is rejected immediately — but the worst case stays factorial, so this is only practical for small graphs. Space is $O(n)$ for the path and the recursion.

## The takeaway

Hamiltonian cycle is the backtracking template applied to *ordering*: the partial solution is a prefix of a permutation, the check is "is there an edge to extend it", and the undo pops the last vertex. It sits with [N-queens](/citadel/algorithms/NQueens), [Sudoku](/citadel/algorithms/SudokuSolver), and [graph coloring](/citadel/algorithms/GraphColoring) as a constraint search with an exponential worst case — and its NP-completeness is one of the anchor points of the [P-versus-NP](/citadel/algorithms/ComplexityClasses) map. For large instances you drop exactness and use heuristics or approximation.
