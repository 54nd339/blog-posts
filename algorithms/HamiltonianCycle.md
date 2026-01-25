---
title: The Hamiltonian Cycle - A Backtracking Quest for the Perfect Tour
description: Explore the Hamiltonian Cycle problem, a classic challenge in graph theory. We dive into how a backtracking algorithm with DFS can determine if a tour exists that visits every vertex exactly once and returns to the start.
date: 2023-02-28
draft: false
slug: /pensieve/algorithms/hamiltonian-cycle
tags:
  - DSA
  - Backtracking
---

Hey everyone, and welcome back to the blog! Imagine you're planning an epic road trip to visit a set of cities, and you want to find a route that visits every single city exactly once, eventually returning to your starting point. This is the essence of the **Hamiltonian Cycle Problem**, a famous and challenging puzzle in graph theory.

It sounds a bit like the Traveling Salesman Problem (TSP), and they are related, but with a key difference. While TSP seeks the *shortest* such tour, the Hamiltonian Cycle problem simply asks: *does such a tour even exist*? This problem is a classic example of an NP-complete problem, meaning that finding an efficient (polynomial-time) solution for all cases is highly unlikely. However, we can tackle it using techniques like **backtracking**. Let's explore!

## The Problem: Finding the "Visit Every City Once" Loop
Let's formally define the Hamiltonian Cycle problem:

Given an undirected graph $G = (V, E)$, where $V$ is the set of vertices (our "cities") and $E$ is the set of edges (the "roads" connecting them):
* A **Hamiltonian Path** is a path in the graph that visits every vertex exactly once.
* A **Hamiltonian Cycle (or Hamiltonian Circuit)** is a Hamiltonian Path that is also a cycle, meaning it starts and ends at the same vertex, having visited every other vertex exactly once in between.

The problem is to determine if such a Hamiltonian Cycle exists in the given graph.

Your provided text correctly notes the similarity to TSP but highlights the key distinction: "the Hamiltonian cycle problem is to find if there exists a tour that visits every city exactly once, *without the need to return to the starting city*" for the path variant, and *with* the return for the cycle variant which is more common. The core is visiting each vertex once. For the cycle, the start and end being the same after visiting all others is key.

## The Strategy: Backtracking with Depth-First Search (DFS)

Since finding a Hamiltonian Cycle is NP-complete, we often rely on algorithms that might explore many possibilities. **Backtracking** is a common approach. The idea is to build a path, vertex by vertex, and if we reach a dead end or violate a condition, we "backtrack" (undo the last step) and try a different option.

The algorithm typically works as follows:
1.  Start at an arbitrary vertex (let's say vertex 0) and add it to our current path.
2.  Try to extend the path by adding an adjacent, unvisited vertex.
3.  If we can add such a vertex, mark it as visited, add it to the path, and recursively try to extend the path from this new vertex.
4.  If we successfully extend the path to include all vertices in the graph:
    * Check if there's an edge from the last vertex in the path back to the starting vertex (vertex 0).
    * If yes, a Hamiltonian Cycle exists!
5.  If we cannot add any more valid vertices to the current path before including all vertices, or if we've included all vertices but cannot return to the start, then this path doesn't lead to a solution. We "backtrack": remove the last added vertex from the path, mark it as unvisited, and try a different adjacent vertex from the previous step.
6.  If all possibilities from a certain point are exhausted without finding a cycle, we backtrack further.

### Pseudocode for Hamiltonian Cycle Detection

Here's a conceptual pseudocode based on your provided structure, which attempts to find a Hamiltonian Cycle:

```pseudocode
// graph: Adjacency matrix or list representing the graph
// graph.numberOfVertices: Total number of vertices

function HamiltonianCycle(graph):
    n = graph.numberOfVertices
    // 'path' array stores the current path being built.
    // Initialize with -1 (or similar) to indicate no vertex assigned yet.
    path = new Array(n)
    for i from 0 to n-1:
        path[i] = -1
   
    // Start the path from vertex 0
    path[0] = 0
   
    // Try to build the rest of the path starting from the next position (pos=1)
    if HamiltonianCycleUtil(graph, path, 1) is false:
        print("No Hamiltonian cycle exists")
        return false // Or some indication of failure
    else:
        print("Hamiltonian cycle exists:")
        // To print the full cycle, add the starting vertex at the end
        print_path_with_return_to_start(path)
        return true // Or print path

// Recursive utility function to find a Hamiltonian cycle
// graph: The graph
// path: Current path being constructed
// pos: The current position in the 'path' array we are trying to fill
function HamiltonianCycleUtil(graph, path, pos):
    n = graph.numberOfVertices

    // Base Case: If all vertices are included in the path
    if pos == n:
        // Check if there's an edge from the last vertex in the path
        // back to the starting vertex (path[0])
        if graph.isEdge(path[pos-1], path[0]) is true:
            return true // Hamiltonian cycle found!
        else:
            return false // Path complete, but not a cycle

    // Try different vertices as the next candidate for the current position 'pos'
    // We iterate through all vertices (except the starting vertex 0, which is fixed at path[0])
    for v_candidate from 1 to n-1: // Assuming vertices are 0 to n-1
        if isSafeToAdd(v_candidate, graph, path, pos) is true:
            path[pos] = v_candidate // Add vertex to path

            // Recur to construct the rest of the path
            if HamiltonianCycleUtil(graph, path, pos + 1) is true:
                return true // Cycle found in deeper recursion

            // If adding v_candidate doesn't lead to a solution, backtrack
            path[pos] = -1 // Remove v_candidate from path (reset)
           
    return false // No vertex leads to a solution from this 'pos'

// Helper function to check if vertex 'v_check' can be safely added at 'path[pos]'
function isSafeToAdd(v_check, graph, path, pos):
    // 1. Check if there's an edge from the last added vertex (path[pos-1])
    //    to the candidate vertex v_check.
    if graph.isEdge(path[pos-1], v_check) is false:
        return false

    // 2. Check if the candidate vertex v_check has already been included in the path.
    for i from 0 to pos-1:
        if path[i] == v_check:
            return false // Vertex already in path
           
    return true // It's safe to add v_check

// Helper to print the path
function print_path_with_return_to_start(path):
    for i from 0 to length(path)-1:
        print path[i] + " -> "
    print path[0] // Return to start
```

### Explanation of the Algorithm:

1. `HamiltonianCycle(graph)` (Main Function):
    * Initializes an array `path` to store the vertices in the current Hamiltonian path being explored. The first vertex of the path is fixed (e.g., vertex `0`).
    * Calls the recursive utility function `HamiltonianCycleUtil` to try and build a path starting from the second position (`pos = 1`).
    * If `HamiltonianCycleUtil` returns `true`, a cycle is found and printed. Otherwise, it indicates no such cycle exists.

2. `HamiltonianCycleUtil(graph, path, pos)` (Recursive Helper):
    * **Base Case:** If `pos` equals `n` (the number of vertices), it means we have successfully included all `n` vertices in our path `array`. Now, we just need to check if there's an edge from the last vertex in the path (`path[pos-1]`) back to the starting vertex (`path[0]`). If yes, a Hamiltonian cycle is complete, and we return `true`. Otherwise, this path is not a cycle, so return `false`.
    * **Recursive Step:** Iterate through all possible vertices v_candidate (typically from 1 to n-1, since vertex 0 is our fixed start) to be placed at the current pos in the path.
        * For each `v_candidate`, call `isSafeToAdd` to check if it's valid to add this vertex to the path at the current position.
        * If `isSafeToAdd` returns `true`:
            * Add `v_candidate` to `path[pos]`.
            * Recursively call `HamiltonianCycleUtil` for the next position (`pos + 1`).
            * If this recursive call returns `true` (meaning a full cycle was found starting with the current path), then propagate `true` up the call stack.
            * **Backtrack:** If the recursive call returns `false` (meaning adding `v_candidate` at `path[pos]` did not lead to a solution), then remove `v_candidate` from `path[pos]` (e.g., by setting `path[pos] = -1`) and try the next `v_candidate`.
    * **Failure for current `pos`:** If all vertices have been tried for the current `pos` and none lead to a Hamiltonian cycle, return `false`.

3. `isSafeToAdd(v_check, graph, path, pos)` (Safety Check):
    This function ensures two conditions:
    * There must be an edge in the graph from the previously added vertex (`path[pos-1]`) to the `v_check`.
    * The `v_check` must not have already been included in the `path` at an earlier position (to ensure each vertex is visited exactly once).

## Complexity Analysis

The Hamiltonian Cycle problem is famously NP-complete, meaning there's no known polynomial-time algorithm to solve it for all graphs.

* **Time Complexity:** The backtracking algorithm, in the worst case, might explore all possible permutations of vertices to see if they form a Hamiltonian cycle. If you fix the starting vertex, there are $(n-1)!$ permutations for the remaining $n-1$ vertices. For each permutation, checking if it forms a cycle involves checking $n$ edges. So, the time complexity is roughly in the order of O(n!) or O(n * (n-1)!), which is exponential. For dense graphs, checks might prune the search space a bit faster, but the worst-case remains dire.
* **Space Complexity:** The space complexity is primarily for storing the `path` array, which is of size $n$, and for the recursion call stack, which can go up to depth $n$. Therefore, the space complexity is O(n).

Due to this high time complexity, the backtracking approach is only feasible for relatively small graphs. For larger graphs, heuristic algorithms or approximation algorithms are often used to find "good enough" solutions if an exact solution is not strictly required or computationally viable.

## Applications & Significance

While finding Hamiltonian Cycles is hard, the problem has connections to various theoretical and practical areas:

* **Operations Research:** Planning routes or sequences where each point must be visited once.
* **Network Design:** Certain network topology verification tasks.
* **Computational Biology:** Analyzing DNA sequences.
* **Traveling Salesman Problem (TSP):** The Hamiltonian Cycle problem is a subproblem of TSP (TSP asks for the shortest Hamiltonian Cycle in a weighted graph).

Understanding problems like the Hamiltonian Cycle gives us insights into the limits of computation and the nature of NP-complete problems, which are abundant in computer science and real-world optimization.

## Key Takeaways

* The **Hamiltonian Cycle Problem** seeks to find a cycle in a graph that visits every vertex exactly once and returns to the starting vertex.
* It is an NP-complete problem, meaning efficient solutions for large graphs are unlikely.
* **Backtracking with Depth-First Search (DFS)** is a common algorithmic approach to find such cycles, systematically exploring possibilities.
* The worst-case time complexity of this backtracking algorithm is exponential (e.g., O(n!)), making it suitable primarily for smaller graphs.
* The problem is fundamental in graph theory and has implications for various optimization and routing challenges.

It's a fascinating example of how a simply stated problem can lead to profound computational complexity!