---
title: Graphs for Competitive Programming - Structure, Trees, and Flows
description: A map of contest graph algorithms beyond the basics — the DFS-tree family (bridges, SCCs, 2-SAT), the shortest-path variants, spanning-tree extensions, flows and matchings, and the heavy tree decompositions.
date: 2024-09-14
draft: false
slug: /cp/graphs
tags:
  - Competitive Programming
  - Graphs
  - Graph Theory
---

The [dsa graphs guide](/citadel/dsa/graphs) covers traversal, flood fill, and union-find — the interview core. This section is the competitive layer on top: what the DFS tree tells you, the shortest-path algorithms past plain Dijkstra, spanning-tree extensions, the flow/matching toolkit, and the tree decompositions that turn path queries into segment-tree queries.

## The DFS tree and what hangs off it

A [depth-first search](/citadel/cp/depth-first-search) imposes a tree on the graph, with every non-tree edge a **back edge** (undirected) or one of back/forward/cross (directed). Entry/exit timestamps plus the "lowest reachable ancestor" (`low`) value give:

- [Bridges and cut vertices](/citadel/cp/bridges-and-cut-vertices) — edges/vertices whose removal disconnects the graph, in $O(V + E)$.
- [Strongly connected components](/citadel/cp/strongly-connected-components) — Tarjan or Kosaraju; the condensation is a DAG.
- [2-SAT](/citadel/cp/two-sat) — an implication graph whose SCCs decide satisfiability.
- Biconnected components, the block-cut tree, and [finding bridges online](/citadel/cp/finding-bridges-online).

## Shortest paths, the contest variants

- Plain **Dijkstra** / **Bellman-Ford** / **Floyd-Warshall** — see [path finding](/citadel/algorithms/PathFinding).
- [0-1 BFS](/citadel/cp/zero-one-bfs) — a deque replaces the heap when edge weights are $0$ or $1$: $O(V + E)$.
- [D'Esopo-Pape](/citadel/cp/desopo-pape) — fast in practice, exponential worst case; know the trap.
- [Finding a negative cycle](/citadel/cp/finding-negative-cycle) — Bellman-Ford with parent pointers.
- [Paths of fixed length](/citadel/cp/fixed-length-paths) — matrix exponentiation on the adjacency matrix, over the $(\min, +)$ semiring for shortest bounded-length paths.

## Spanning trees, extended

Prim and Kruskal are in [minimum spanning tree](/citadel/algorithms/MinimumSpanningTree). Beyond:

- [Second-best MST](/citadel/cp/second-best-mst) — swap exactly one edge.
- [Kirchhoff's theorem](/citadel/cp/kirchhoff-theorem) — count spanning trees via a [determinant](/citadel/cp/matrix-determinant).
- [Prüfer code](/citadel/cp/prufer-code) — the bijection between labeled trees and sequences (Cayley's formula).
- [Eulerian path](/citadel/cp/eulerian-path) — Hierholzer's algorithm and the degree conditions.

## Flows and matchings

- [Maximum flow](/citadel/cp/maximum-flow) — Edmonds-Karp, Dinic, push-relabel; min-cut is the dual.
- [Minimum-cost flow](/citadel/cp/min-cost-flow) — successive shortest paths with Johnson potentials.
- [Flows with demands](/citadel/cp/flows-with-demands) — lower bounds on edges.
- [The assignment problem](/citadel/cp/assignment-problem) — the Hungarian algorithm, $O(n^3)$.
- [Bipartite matching](/citadel/cp/bipartite-matching) — Kuhn's augmenting paths, König's theorem (min vertex cover = max matching).
- [Global min cut](/citadel/cp/min-cut-stoer-wagner) — Stoer-Wagner, no flow needed.

## Trees as data structures

- [Lowest common ancestor](/citadel/cp/lowest-common-ancestor) — binary lifting, Euler tour + RMQ, offline Tarjan.
- [Heavy-light decomposition](/citadel/cp/heavy-light-decomposition) — path queries become $O(\log^2 n)$ segment-tree queries.
- [Centroid decomposition](/citadel/cp/centroid-decomposition) — divide-and-conquer on trees for path-counting problems.
- [The rerooting technique](/citadel/cp/rerooting-technique) — one DP that yields the answer for *every* root.

## Recognising the tool

- "edge/vertex whose removal disconnects" → bridges / cut vertices.
- "mutually reachable groups", "implications" → SCC, 2-SAT.
- weights $\in \{0, 1\}$ → 0-1 BFS.
- "maximum matching / minimum vertex cover in a bipartite graph" → Kuhn + König.
- "max flow / min cut", "is there a feasible assignment" → flow.
- "queries on paths in a tree" → HLD or LCA + prefix sums.
- "count paths with property X in a tree" → centroid decomposition.

## Where this goes next

The [Miscellaneous](/citadel/cp/miscellaneous) section has game theory on graphs; [geometry](/citadel/cp/geometry) feeds in via [planar duality](/citadel/cp/planar-graph-faces) and [Manhattan MST](/citadel/cp/manhattan-distance). The [determinant](/citadel/cp/matrix-determinant) counts spanning trees and perfect matchings of planar graphs.
