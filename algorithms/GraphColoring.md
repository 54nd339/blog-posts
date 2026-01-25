---
title: Graph Coloring - A Backtracking Approach to a Classic Combinatorial Puzzle
description: Dive into the Graph Coloring problem, a famous challenge in computer science. We explore how to find a valid coloring with the minimum number of colors using a backtracking algorithm, ensuring no two adjacent vertices share the same color.
date: 2023-02-26
draft: false
slug: /pensieve/algorithms/graph-coloring
tags:
  - DSA
  - Backtracking
---

Hey everyone, and welcome back to the blog! Ever tried coloring a map where no two adjacent regions have the same color? That simple childhood activity is actually a visual representation of a famous and fascinating problem in computer science and mathematics: the **Graph Coloring problem**.

This classic combinatorial challenge requires us to assign "colors" (which can be any distinct labels) to the vertices (or nodes) of a graph such that no two adjacent vertices share the same color. The ultimate goal is often to find a valid coloring using the *minimum* number of colors. It's a well-known example of a constraint satisfaction problem and serves as a perfect illustration of how **backtracking algorithms** can be used to find solutions. Let's explore how we can tackle this colorful puzzle!

## The Problem: Painting by Numbers (or Vertices!)
Let's formally define the Graph Coloring problem:

Given an undirected graph $G = (V, E)$, where $V$ is the set of vertices and $E$ is the set of edges:
* We need to assign a color to each vertex $v \in V$.
* The constraint is that if there is an edge $(u, v) \in E$ (meaning vertices $u$ and $v$ are adjacent), then vertex $u$ and vertex $v$ must be assigned different colors.
* The optimization version of the problem asks for the minimum number of colors needed for such a valid coloring. This minimum number is called the **chromatic number** of the graph.

The problem is famously NP-hard in its general form (finding the absolute minimum number of colors), but for a fixed number of colors `m`, we can determine if an `m`-coloring exists.

## The Strategy: Backtracking to Find a Valid Coloring

A common approach to solve the graph coloring problem (specifically, to find *a* valid coloring with a given number of colors, or to try and find one with $m$ colors where $m$ is related to the graph's properties) is to use a **backtracking algorithm**.

The basic idea is to:
1.  Start assigning colors to vertices one by one, usually in some predefined order.
2.  For each vertex, try to assign it a color from a predefined set of available colors.
3.  Check if the current color assignment is "safe" – meaning it doesn't conflict with the colors of already colored adjacent vertices.
4.  If it's safe, move to the next vertex and repeat.
5.  If it's not safe, or if you reach a point where no safe color can be assigned to the current vertex, **backtrack**: undo the color assignment for the current vertex (or previous ones) and try a different color or a different path in the decision tree.
6.  If all vertices are successfully colored, a valid coloring has been found. If all possibilities are exhausted without finding a valid coloring for a given number of colors, then no such coloring exists with that many colors.

To find the minimum number of colors, one might try to find a valid k-coloring starting with k=1, then k=2, and so on, until a valid coloring is found. Or, for practical purposes, we often try to color with $m$ colors, where $m$ is an upper bound like the maximum degree of the graph plus one (as shown in the pseudocode below).

### Pseudocode for Graph Coloring with Backtracking

Here's a conceptual look at the backtracking algorithm, adapted from your provided pseudocode, aiming to find *any* valid coloring using up to $m = \text{maxDegree}(graph) + 1$ colors.

```pseudocode
// graph: Represents the graph, typically with an adjacency list graph.adj[v]
// |V|: Number of vertices in the graph

function GraphColoring(graph):
    num_vertices = |V| // Total number of vertices
    // 'colors' array will store the color assigned to each vertex (0 means no color)
    // Vertices are assumed to be indexed from 0 to num_vertices-1
    colors = array of size num_vertices initialized with 0

    // Determine the number of colors to try (upper bound)
    // Brooks' Theorem states chromatic number <= maxDegree, unless it's a complete graph or odd cycle.
    // A simpler upper bound is maxDegree + 1, which always works.
    num_colors_to_try = maxDegree(graph) + 1

    if solveGraphColoringUtil(graph, num_colors_to_try, colors, 0): // Start with vertex 0
        print "Solution Exists: "
        print colors // Print the found coloring
    else:
        print "No solution exists with " + num_colors_to_try + " colors (or fewer as per logic)."

// Recursive utility function to solve graph coloring
// graph: The graph
// m_colors: The number of different colors available (e.g., 1, 2, ..., m_colors)
// colors: Array storing assigned colors for vertices
// v_idx: The current vertex index we are trying to color
function solveGraphColoringUtil(graph, m_colors, colors_array, v_idx):
    num_vertices = |V|

    // Base Case: If all vertices are assigned a color, then we are done
    if v_idx == num_vertices:
        return true // A valid coloring is found

    // Try assigning different colors to the current vertex v_idx
    for c = 1 to m_colors: // Iterate through available colors (1 to m)
        if isSafeToColor(graph, colors_array, v_idx, c):
            colors_array[v_idx] = c // Assign color c to vertex v_idx

            // Recur for the next vertex
            if solveGraphColoringUtil(graph, m_colors, colors_array, v_idx + 1):
                return true // If successful, propagate true up

            // If assigning color c doesn't lead to a solution,
            // then remove it (backtrack) and try the next color.
            colors_array[v_idx] = 0 // Backtrack: reset color
   
    // If no color can be assigned to this vertex from the 'm_colors' available
    return false

// Helper function to check if assigning color 'c' to vertex 'v' is safe
// graph: The graph
// colors_array: Current color assignments
// v_idx: The vertex to check
// c: The color to try
function isSafeToColor(graph, colors_array, v_idx, color_to_try):
    // Iterate through all adjacent vertices of v_idx
    for adjacent_vertex_idx in graph.adj[v_idx]:
        // If an adjacent vertex already has the color_to_try
        if colors_array[adjacent_vertex_idx] == color_to_try:
            return false // Not safe
    return true // Safe to assign this color

// Helper function to find the maximum degree of the graph
function maxDegree(graph):
    max_deg = 0
    for v_idx = 0 to |V|-1: // Iterate through all vertices
        current_degree = length(graph.adj[v_idx]) // Number of adjacent vertices
        if current_degree > max_deg:
            max_deg = current_degree
    return max_deg
```

### Explanation of the Algorithm:

1. ` GraphColoring(graph)` (Main Function):
    * Initializes a `colors` array (same size as the number of vertices) to store the color assigned to each vertex. Initially, all are uncolored (e.g., set to 0).
    * Determines `m_colors`, the number of colors to attempt. A common upper bound that guarantees a coloring is possible is $\text{maxDegree}(graph) + 1$. (The actual chromatic number might be much smaller).
    * Calls the recursive helper function `solveGraphColoringUtil` starting with the first vertex (index 0).
    * If the helper function returns `true`, it means a valid coloring was found, and it's printed. Otherwise, it reports that no solution exists with the given number of colors.

2. `solveGraphColoringUtil(graph, m_colors, colors_array, v_idx)` (Recursive Helper):
    * **Base Case:** If `v_idx` equals the total number of vertices, it means all vertices have been successfully assigned a color, so it returns `true`.
    * **Recursive Step:** For the current vertex `v_idx`, it tries to assign each available color `c` (from 1 to `m_colors`):
        * **Check Safety:** Before assigning, it calls `isSafeToColor` to check if color `c` can be safely assigned to `v_idx` (i.e., no adjacent, already-colored vertex has color `c`).
        * **Assign & Recur:** If it's safe, it assigns `colors_array[v_idx] = c` and then makes a recursive call to color the next vertex `(v_idx + 1)`.
            * If this recursive call returns `true` (meaning a solution was found for the rest of the graph), then the current assignment was good, so propagate true upwards.
        * **Backtrack:** If the recursive call returns `false` (meaning assigning color `c` to `v_idx` did not lead to a solution for subsequent vertices), then this assignment of `c` was not part of a valid overall coloring. So, we **backtrack**: reset colors_array`[v_idx] = 0` (uncolor it) and try the next available color `c+1` for `v_idx`.
    * **Failure:** If all `m_colors` have been tried for vertex `v_idx` and none lead to a solution, it returns `false`, triggering backtracking in the previous recursive call.

3. `isSafeToColor(graph, colors_array, v_idx, color_to_try)` (Safety Check):
    * This function iterates through all vertices `u` adjacent to `v_idx`.
    * If any adjacent vertex `u` already has the `color_to_try` assigned to it, then it's not safe to assign this color to `v_idx`, so it returns `false`.
    * If no adjacent vertex has `color_to_try`, it returns `true`.

## Complexity Analysis

* **Time Complexity:** In the worst case, the backtracking algorithm might explore many possible color assignments. If $m$ is the number of colors we are trying and $V$ is the number of vertices, the algorithm could explore up to $m^V$ assignments. For each assignment, checking safety might take time proportional to the degree of the vertex. So, a loose upper bound is often given as **O($m^V$ * $V$**) or similar exponential forms. This indicates why graph coloring is NP-hard for the general case of finding the chromatic number.
* **Space Complexity:** The space complexity is primarily due to the colors array, which is O(V), and the recursion call stack, which can go up to O(V) in depth. So, the space complexity is **O(V)**.

## Applications of Graph Coloring

The graph coloring problem, despite its computational hardness for finding the absolute minimum colors, has numerous practical applications:

* **Scheduling:** Assigning time slots (colors) to tasks (vertices) where conflicting tasks (adjacent vertices) cannot occur at the same time. For example, scheduling exams so that no student has two exams simultaneously.
* **Register Allocation in Compilers:** Assigning variables (vertices) to a limited number of CPU registers (colors) such that variables that are "live" at the same time (and thus conflict, forming an edge) get different registers.
* **Frequency Assignment:** Assigning radio frequencies (colors) to transmitters (vertices) such that nearby transmitters (adjacent vertices) don't interfere.
* **Map Coloring:** The original inspiration – coloring regions on a map.
* **Sudoku Solving:** Can be modeled as a graph coloring problem.
Resource Allocation in Distributed Systems.

## Key Takeaways

* The **Graph Coloring problem** involves assigning colors to graph vertices such that no two adjacent vertices share the same color, often with the goal of using the minimum number of colors.
* **Backtracking** is a common algorithmic approach to find a valid coloring. It explores possibilities, assigning colors one by one, and backtracks if a valid path is not found.
* The problem is NP-hard in its general form, meaning finding the absolute minimum number of colors for an arbitrary graph can be very computationally expensive.
* Despite its complexity, graph coloring has wide-ranging applications in scheduling, resource allocation, compiler design, and more.

It's a fantastic problem that demonstrates the power of recursive thinking and the nature of constraint satisfaction!