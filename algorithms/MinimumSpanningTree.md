---
title: Minimum Spanning Trees - Connecting the Dots Efficiently with Kruskal's & Prim's Algorithms
description: Discover the Minimum Spanning Tree (MST) problem and how greedy algorithms like Kruskal's and Prim's efficiently find the cheapest way to connect all vertices in a weighted graph, with applications in network and circuit design.
date: 2023-02-13
draft: false
slug: /pensieve/algorithms/minimum-spanning-tree
tags:
  - DSA
  - Greedy
---

Hey everyone, and welcome back to the blog! Imagine you're tasked with connecting a set of cities with fiber optic cables, or designing a computer network, or even laying out connections on a printed circuit board. In all these scenarios, you want to ensure all points are connected, but you also want to minimize the total length of cable or wire used to save costs. This is the essence of the **Minimum Spanning Tree (MST)** problem, a fundamental challenge in graph theory with a wide array of practical applications.

Today, we'll explore what an MST is, its key properties, and then dive into two famous greedy algorithms used to find it: **Kruskal's Algorithm** and **Prim's Algorithm**.

## What is a Minimum Spanning Tree (MST)? Connecting Everything on a Budget
Let's start with the basics:
* **Graph:** We're dealing with a **connected, edge-weighted, undirected graph**. This means we have a set of vertices (nodes) and edges (connections between vertices), each edge has a weight or cost associated with it, and you can get from any vertex to any other vertex.
* **Spanning Tree:** A spanning tree of a graph is a subgraph that includes *all* the vertices of the original graph and is a tree (meaning it's connected and has no cycles).
* **Minimum Spanning Tree (MST):** An MST is a spanning tree where the sum of the weights of all the edges in the tree is minimized. It's the "cheapest" way to connect all the vertices in the graph.

### Key Properties of an MST:
* It must connect all vertices of the graph (it's a spanning tree).
* It must be acyclic (it contains no cycles, which is a property of any tree).
* It must have the minimum possible total edge weight among all possible spanning trees for that graph.
* An interesting property: If all edge weights in a connected graph are distinct, then the MST is unique. If some edge weights are the same, there might be multiple MSTs with the same total minimum weight.

## Why Care About MSTs? Real-World Applications

Minimum Spanning Trees aren't just an academic exercise; they have numerous practical applications:

* **Network Design:** Designing efficient and cost-effective communication networks like telephone networks, computer networks (e.g., connecting LANs), or even transportation networks (e.g., laying down pipelines or railway lines).
* **Circuit Design:** Used in designing printed circuit boards (PCBs) where the objective is to connect all components using the minimum total length of conductive wire.
* **Clustering:** MSTs can be used in machine learning and data mining for clustering data points. By removing the most expensive edges from an MST, you can naturally partition data into clusters.
* **Image Segmentation:** Can be used in image processing to segment images.
* **Approximation Algorithms:** MSTs are sometimes used as a subroutine in approximation algorithms for NP-hard problems like the Traveling Salesman Problem.

## Finding the MST: Kruskal's Algorithm
Kruskal's algorithm is a classic greedy algorithm that finds an MST by iteratively adding the next "safest" and "cheapest" edge.

### How Kruskal's Algorithm Works:

1.  **Sort Edges:** Start by sorting all the edges in the graph $G=(V,E)$ in non-decreasing order of their weights.
2.  **Initialize MST:** Create an empty set $T$ that will eventually hold the edges of the MST.
3.  **Initialize Disjoint Sets:** Consider each vertex in $V$ as being in its own separate set or component (this is where a Disjoint Set Union - DSU data structure, also known as Union-Find, is typically used).
4.  **Iterate Through Sorted Edges:** For each edge $(u, v)$ in the sorted list:
    * **Check for Cycles:** Determine if adding the edge $(u, v)$ to $T$ would create a cycle with the edges already in $T$. Using a DSU structure, this means checking if vertices $u$ and $v$ are already in the same set (i.e., `FindSet(u) == FindSet(v)`).
    * **Add Edge if Safe:** If adding $(u, v)$ does *not* create a cycle (i.e., `FindSet(u) != FindSet(v)`), add the edge $(u, v)$ to $T$ and merge the sets containing $u$ and $v$ (i.e., `Union(u, v)`).
5.  **Completion:** Repeat step 4 until $T$ contains $V-1$ edges (the number of edges in a spanning tree for $V$ vertices), or until all edges have been considered.
6.  Return $T$.

### Pseudocode for Kruskal's Algorithm:

```pseudocode
function KruskalMST(Graph G):
    MST_edges = ∅ // Initialize an empty set for the MST edges

    // For each vertex v in G, create a disjoint set containing just v
    for each vertex v in G.vertices:
        MakeSet(v) // Initializes a new set with v as its only member

    // Sort all edges in G by their weight in non-decreasing order
    Sort G.edges by weight in non-decreasing order

    // Iterate through the sorted edges
    for each edge (u, v) with weight w in G.edges:
        // If u and v are not already connected (i.e., in different sets)
        if FindSet(u) ≠ FindSet(v):
            // Add this edge to our MST
            MST_edges = MST_edges ∪ {(u, v)}
            // Merge the sets containing u and v
            Union(u, v)
   
    return MST_edges
```
(Here, MakeSet(v), FindSet(u), and Union(u,v) are operations of a Disjoint Set Union data structure.)

![Kruskal's Algorithm](https://upload.wikimedia.org/wikipedia/commons/b/bb/KruskalDemo.gif)

## Finding the MST: Prim's Algorithm
Prim's algorithm is another popular greedy algorithm for finding an MST. Unlike Kruskal's, which builds the MST by adding edges from all over the graph, Prim's algorithm builds the tree one vertex at a time, starting from an arbitrary vertex and always adding the cheapest possible connection from the currently formed tree to a vertex not yet in the tree.

### How Prim's Algorithm Works:

1. **Initialization:**
    * Initialize an empty set $T$ (or an array `parent` as in your pseudocode) to store the edges (or structure) of the MST.
    * Choose an arbitrary starting vertex $r$ from the graph to be the root of the MST.
    * For all vertices $u$ in the graph, maintain a `key[u]` value, which is the minimum weight of an edge connecting $u$ to any vertex already in the growing MST. Initialize `key[u] = ∞` for all $u$, and `key[r] = 0`.
    * Maintain a `parent[u]` array to store the parent of $u$ in the MST (initially NIL for all).
    * Use a min-priority queue `Q` containing all vertices not yet in the MST, prioritized by their `key` values. Initially, `Q` contains all vertices of $G$.

2. **Build the Tree Iteratively:**

* While the priority queue Q is not empty:
    * **Extract Minimum:** Extract the vertex $u$ from `Q` that has the minimum `key` value. This $u$ (and the edge `(parent[u], u)` if `parent[u]` is not NIL) is added to the MST.
    * Update Neighbors: For each vertex $v$ adjacent to $u$:
        * If $v$ is still in the priority queue `Q` (meaning $v$ is not yet in the MST) AND the weight of the edge $(u, v)$ is less than the current `key[v]`:
            * Update `parent[v] = u`.
            * Update `key[v] = weight(u, v)` (and update $v$'s position in the priority queue Q).

3. **Completion:** When `Q` is empty, the `parent` array (or the set $T$) defines the MST.

4. Return the set of edges formed by `(u, parent[u])` for each $u$ where `parent[u]` is not NIL.

### Pseudocode for Prim's Algorithm:

```pseudocode
// G: The graph (vertices and edges with weights)
// r: An arbitrary starting root vertex

function PrimMST(Graph G, Vertex r):
    // key[u] stores the minimum weight of an edge connecting u to a vertex in the MST
    // parent[u] stores the parent of u in the MST
    for each vertex u in G.vertices:
        key[u] = ∞
        parent[u] = NIL
   
    key[r] = 0 // Start with vertex r, its key is 0 (cost to connect to itself)

    // Q is a min-priority queue of vertices, ordered by key[u]
    // Initialize Q with all vertices from G
    Q = G.vertices

    while Q is not empty:
        // Extract vertex u not yet in the MST with the minimum key value
        u = Extract-Min(Q) // Removes u from Q

        // For each vertex v adjacent to u
        for each vertex v in G.adjacent_vertices(u):
            // If v is still in Q (not yet in MST) and the edge (u,v) is lighter
            // than the current best known edge to connect v to the MST
            if v is in Q and weight(u, v) < key[v]:
                parent[v] = u
                key[v] = weight(u, v)
                // (Implicitly, v's priority in Q needs to be updated here)
   
    // The MST can be reconstructed from the parent array
    // e.g., for each v != r, the edge (v, parent[v]) is in the MST.
    return parent // (or the set of edges derived from parent)
```

![Prim's Algorithm](https://upload.wikimedia.org/wikipedia/commons/9/9b/PrimAlgDemo.gif)

## Time and Space Complexity

The efficiency of these algorithms depends on the data structures used, particularly for the priority queue in Prim's and the disjoint set union in Kruskal's.

| Algorithm | Time Complexity (Typical Implementations) | Space Complexity |
|-----------|-------------------------------------------|------------------|
| **Kruskal's** | O(E log E) or O(E log V) | O(V + E) |
|  | Dominated by edge sorting (O(E log E)). DSU operations (MakeSet, FindSet, Union) with path compression and union by rank/size are nearly constant time on average per operation, leading to an additional O(E α(V)) where α is the very slowly growing inverse Ackermann function. So, O(E log E) or O(E log V) if E is much larger than V. | |
| **Prim's** |  |  |
| - Adjacency Matrix & linear scan for min key | O(V²) | O(V²) (for adj matrix) or O(V + E) (adj list) |
| - Adjacency List & Binary Heap (for Priority Queue) | O((V + E) log V) or simply O(E log V) if E ≥ V-1 | O(V + E) |
| - Adjacency List & Fibonacci Heap (for Priority Queue) | O(E + V log V) | O(V + E) |


Where:

- $V$ is the number of vertices in the graph.
- $E$ is the number of edges in the graph.

**Choosing Between Kruskal's and Prim's:**
* **Kruskal's algorithm** is generally preferred for sparse graphs (where $E$ is much smaller than $V^2$, roughly $E \approx O(V)$). In this case, O(E log V) or O(E log E) can be better than Prim's O(V²) with a simple implementation.
* **Prim's algorithm**, especially with a Fibonacci heap, can be faster for dense graphs (where $E$ is close to $V^2$). The O(V²) version (using an adjacency matrix or linear scan of keys) is simple to implement and efficient for dense graphs. The O(E log V) version (with binary heap) is a good general-purpose choice.

## Conclusion: Connecting the World Efficiently

Minimum Spanning Trees are a fundamental concept in graph theory with a surprising number of practical applications in designing efficient networks and systems. Algorithms like Kruskal's and Prim's provide elegant and efficient greedy approaches to solving this optimization problem. Understanding their mechanics and complexities is crucial for any developer or system designer tasked with finding the "cheapest" way to connect a set of points.

They are essential tools for solving optimization problems in various domains, and understanding their time and space complexity is crucial for selecting the most appropriate algorithm for a given graph.
