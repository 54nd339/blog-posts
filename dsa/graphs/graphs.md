---
title: Graphs - Traversal Is the Whole Toolbox
description: A guide to the Graphs section of NeetCode 150 — grids and adjacency lists are the same object, and DFS, BFS, and union-find cover counting, shortest paths, and cycle detection.
date: 2021-12-05
draft: false
slug: /dsa/graphs
tags:
  - Competitive Programming
  - Graphs
  - Breadth-First Search
---

Most of this section is one idea: a graph is a set of nodes and a neighbour relation, and you explore it with DFS, BFS, or union-find. A grid is just a graph whose nodes are cells and whose edges are the four orthogonal moves. Once you see that, "islands", "rooms", "courses", and "words" are the same problem with a different neighbour function.

## The three engines

- **DFS** — recursion or an explicit stack. Natural for "flood this component", "does a path exist", and cycle detection with a grey/black colour.
- **BFS** — a queue. The one that gives **shortest paths** on an unweighted graph, and layer counts when you drain the queue one `len(q)` snapshot at a time.
- **Union-Find** — no traversal at all. Answers "are these two connected" and "how many components" as you add edges, in near-constant time per operation.

## Patterns by problem

**Flood fill / connected components.** Scan for an unvisited seed, flood its whole component, repeat.

- [Number of Islands](/citadel/dsa/number-of-islands) — count the floods.
- [Max Area of Island](/citadel/dsa/max-area-of-island) — each flood returns a size.
- [Number of Connected Components](/citadel/dsa/number-of-connected-components-in-an-undirected-graph) — same, on an edge list.
- [Clone Graph](/citadel/dsa/clone-graph) — the flood builds a copy via an original→clone map.

**Multi-source BFS.** Seed the queue with *every* source at distance 0; the first visit to a node is its nearest-source distance.

- [Walls and Gates](/citadel/dsa/walls-and-gates) — sources are the gates.
- [Rotting Oranges](/citadel/dsa/rotting-oranges) — sources are the rotten oranges; the answer is the layer count.

**Search from the boundary inward.** When "reaches the edge" or "enclosed" is the property, flood from the border instead of from every cell.

- [Pacific Atlantic Water Flow](/citadel/dsa/pacific-atlantic-water-flow) — two border floods, intersected.
- [Surrounded Regions](/citadel/dsa/surrounded-regions) — border flood marks survivors, sweep flips the rest.

**Topological sort / cycle detection** on a directed graph.

- [Course Schedule](/citadel/dsa/course-schedule) — is it acyclic (grey/black DFS, or Kahn).
- [Course Schedule II](/citadel/dsa/course-schedule-ii) — emit the order (Kahn peels it, DFS post-order reverses it).

**Union-Find** for connectivity as edges arrive.

- [Graph Valid Tree](/citadel/dsa/graph-valid-tree) — connected and `n - 1` edges, no edge re-joins a component.
- [Redundant Connection](/citadel/dsa/redundant-connection) — return the first edge that re-joins a component.

**BFS shortest path on an implicit graph.**

- [Word Ladder](/citadel/dsa/word-ladder) — nodes are words, edges are one-letter edits; wildcard buckets find neighbours fast.

## Recognising it

- A grid with "connected", "region", "island", "reach" → grid DFS/BFS.
- "minimum steps / moves / transformations", unweighted → BFS.
- "nearest gate / exit / source" for many cells at once → multi-source BFS.
- "prerequisites", "ordering", "depends on" → topological sort.
- "are these connected", edges streaming in → union-find.

## The core traversal

```python
def dfs(node):
    visited.add(node)
    for nei in neighbours(node):
        if nei not in visited:
            dfs(nei)
```

Everything above is this loop with `neighbours` redefined and an accumulator bolted on.

## Where this goes next

[Advanced Graphs](/citadel/dsa/advanced-graphs) adds **edge weights**: BFS becomes Dijkstra, "connect everything cheaply" becomes a minimum spanning tree (Prim/Kruskal on the union-find primitive), and Bellman-Ford handles a bounded number of hops.
