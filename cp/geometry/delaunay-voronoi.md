---
title: Delaunay Triangulation and the Voronoi Diagram
description: The triangulation whose circumcircles are empty and its dual the Voronoi diagram, the properties contests exploit (nearest neighbours, Euclidean MST, largest empty circle), and how to build them.
date: 2024-08-29
draft: false
slug: /cp/delaunay-voronoi
tags:
  - Competitive Programming
  - Geometry
  - Computational Geometry
---

The **Voronoi diagram** of a point set partitions the plane into cells, one per point, where a cell is "everything closer to this point than to any other". Its dual — connect two points whose cells share an edge — is the **Delaunay triangulation**, the triangulation in which no point lies inside any triangle's circumcircle. Between them they answer a whole shelf of proximity questions.

## The problem

Given $n$ sites $P_1, \dots, P_n$:

- **Voronoi:** the cell of each site (a convex polygon, possibly unbounded).
- **Delaunay:** a triangulation of the sites where every triangle's circumcircle is **empty** (contains no other site).

They are dual: a Voronoi vertex is the circumcentre of a Delaunay triangle; a Voronoi edge is perpendicular to a Delaunay edge.

## Properties contests exploit

- **Nearest neighbour of each point** is a Delaunay neighbour — so all nearest-neighbour pairs are among the $O(n)$ Delaunay edges.
- **Euclidean minimum spanning tree** is a subgraph of the Delaunay triangulation — build Delaunay ($O(n \log n)$), then [Kruskal](/citadel/algorithms/MinimumSpanningTree) on its $O(n)$ edges. This is *the* way to get a Euclidean MST faster than $O(n^2)$.
- **Largest empty circle** (centre inside the convex hull) is centred at a Voronoi vertex or where a Voronoi edge meets the hull.
- **Closest pair** is a Delaunay edge (also solvable by [divide and conquer](/citadel/algorithms/ClosestPair)).
- **Delaunay maximises the minimum angle** over all triangulations — it avoids skinny triangles, which is why meshing uses it.
- $|\text{Delaunay edges}| \le 3n - 6$, $|\text{triangles}| \le 2n - 5$ — the output is linear.

## The in-circle test

The primitive: is point $D$ **inside** the circumcircle of $\triangle ABC$ (with $A, B, C$ in CCW order)?

$$\text{inCircle}(A, B, C, D) = \begin{vmatrix} a_x - d_x & a_y - d_y & (a_x - d_x)^2 + (a_y - d_y)^2 \\ b_x - d_x & b_y - d_y & (b_x - d_x)^2 + (b_y - d_y)^2 \\ c_x - d_x & c_y - d_y & (c_x - d_x)^2 + (c_y - d_y)^2 \end{vmatrix} > 0.$$

For integer coordinates up to $C$, this $3 \times 3$ [determinant](/citadel/cp/matrix-determinant) has magnitude up to $\sim C^4$ — use 128-bit integers (or Python) to keep it exact. Everything in Delaunay construction reduces to this test plus [orientation](/citadel/cp/basic-geometry).

## How it is built

- **Incremental (Bowyer-Watson):** add points one at a time; each new point deletes all triangles whose circumcircle it violates, leaving a star-shaped hole that you re-triangulate to the new point. $O(n^2)$ worst case, $O(n \log n)$ expected with a good point order / conflict graph.
- **Divide and conquer (Guibas-Stolfi):** split the sites by $x$, triangulate each half, merge along a "zig-zag" of cross edges. $O(n \log n)$ worst case; the classic library implementation, but intricate (quad-edge structure).
- **Fortune's sweep:** a sweep line + beach line of parabolic arcs builds the **Voronoi diagram** directly in $O(n \log n)$; take the dual for Delaunay.
- **Lifting to 3-D:** map $(x, y) \mapsto (x, y, x^2 + y^2)$; the lower convex hull of the lifted points projects to the Delaunay triangulation. Reduces Delaunay to [3-D convex hull](/citadel/cp/convex-hull-construction).

In contest practice, if $n \le a$ few thousand, an $O(n^2)$ incremental Delaunay is often enough; for larger $n$ you need the $O(n \log n)$ divide-and-conquer or Fortune's, or a library.

## Common pitfalls

- **Degeneracies.** Four cocircular points, or three collinear points, make the in-circle / orientation determinant zero. Perturb symbolically, or make a consistent tie choice — an inconsistent one produces a non-triangulation.
- **Precision in `inCircle`.** With `double`, the $C^4$-scale determinant loses all precision for large coordinates. Use exact integer arithmetic (Python, `__int128`, or an adaptive-precision predicate).
- **Unbounded Voronoi cells.** Cells on the convex hull are unbounded; clip them to a large bounding box for output.
- **Collinear input.** No triangulation exists; Delaunay degenerates to the line, Voronoi to parallel strips. Special-case it.
- **Euclidean MST shortcut correctness.** It relies on "EMST ⊆ Delaunay", which holds only for the *Euclidean* metric — not for Manhattan (use the [Manhattan-MST sweep](/citadel/cp/manhattan-distance) there).

## The keystone

Delaunay triangulation = "no point inside any circumcircle"; its dual, the Voronoi diagram, is "cells of nearest site". One in-circle determinant is the whole primitive. The payoff: nearest-neighbour pairs, the closest pair, and the Euclidean MST all live among the $O(n)$ Delaunay edges, computable in $O(n \log n)$.
