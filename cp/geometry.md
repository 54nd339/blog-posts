---
title: Geometry for Competitive Programming - Primitives, Hulls, and Sweeps
description: A map of computational geometry for contests — the cross-product primitive everything is built on, the convex-hull and sweep-line workhorses, and the numerical-robustness rules that decide whether a solution passes.
date: 2024-08-13
draft: false
slug: /cp/geometry
tags:
  - Competitive Programming
  - Geometry
  - Computational Geometry
---

The [maths geometry posts](/citadel/maths/2d-geometry) are about equations of shapes. Contest geometry is about **algorithms on points**: is this polygon convex, do these segments cross, what is the convex hull, which point is nearest. Almost all of it rests on one primitive — the 2-D cross product — and almost all of the pain is floating-point robustness.

## The one primitive: the cross product

For vectors $\vec a = (a_x, a_y)$ and $\vec b = (b_x, b_y)$, the scalar cross product is

$$\vec a \times \vec b = a_x b_y - a_y b_x.$$

Its **sign** is an orientation test: given points $P, Q, R$, the sign of $(Q - P) \times (R - P)$ says whether $P \to Q \to R$ turns left ($+$), right ($-$), or is collinear ($0$). From that one test:

- **[Triangle / polygon area](/citadel/cp/polygon-area)** — half the absolute cross product; the shoelace formula sums it around a polygon.
- **[Segment intersection](/citadel/cp/lines-and-segments)** — two segments cross iff each straddles the other's line (opposite orientation signs), with collinear-overlap handled separately.
- **[Point in polygon](/citadel/cp/point-in-polygon)** — ray casting for general polygons, an $O(\log n)$ orientation binary search for convex ones.
- **[Convex hull](/citadel/cp/convex-hull-construction)** — keep turning the same way (Andrew's monotone chain).

**Work in integers when you can.** If the input coordinates are integers, the cross product is an exact integer (use 64-bit, or watch for overflow up to $\sim 4 \cdot 10^{18}$). Orientation tests are then exact — no epsilon.

## The workhorses

- **[Convex hull](/citadel/cp/convex-hull-construction)** ($O(n \log n)$) — the starting point for diameter (rotating calipers), width, smallest enclosing rectangle, and [Minkowski sums](/citadel/cp/minkowski-sum).
- **Sweep line** — sort events by $x$ (or by angle), process with an ordered structure. Powers [segment-intersection detection](/citadel/cp/intersecting-segments-sweep), [union of segments / rectangles](/citadel/cp/length-of-union-of-segments), closest pair, and [point location](/citadel/cp/point-location).
- **[Half-plane intersection](/citadel/cp/halfplane-intersection)** ($O(n \log n)$) — feasible region of linear constraints, the geometric dual of the convex hull.
- **[Convex hull trick / Li Chao tree](/citadel/cp/convex-hull-trick)** — a DP optimisation that is really "lower envelope of lines".
- **Randomised**: [minimum enclosing circle](/citadel/cp/minimum-enclosing-circle) (Welzl, expected $O(n)$), and simulated annealing for optimum-point problems.

## Robustness rules

- Prefer **integer / exact** arithmetic; only go to `double` when the problem forces it (rotations, circle intersections, sqrt).
- With doubles, compare against an **epsilon** (`1e-9` for well-scaled inputs), and choose it relative to the coordinate magnitude.
- Avoid `atan2` for sorting by angle when a cross-product comparator works — it is slower and less exact. When you must, split by half-plane first.
- Never test a determinant for exactly `== 0` in floating point.
- Degeneracies (collinear points, coincident points, vertical lines, zero-length segments) are where solutions fail — handle them explicitly.

## Recognising the tool

- "is it convex", "area", "do they cross", "which side" → cross-product primitive.
- "smallest convex shape containing", "farthest pair", "width" → convex hull + calipers.
- "do any of these $n$ segments intersect", "total covered length/area" → sweep line.
- "region satisfying these linear inequalities" → half-plane intersection.
- "smallest circle covering all points" → Welzl.
- "nearest pair of points" → [divide and conquer](/citadel/algorithms/ClosestPair).

## Where this goes next

Several geometry results feed [graphs](/citadel/cp/graphs): [planar graph faces](/citadel/cp/planar-graph-faces) is a geometry construction with a graph output, and [Manhattan-distance MST](/citadel/cp/manhattan-distance) reduces a geometric problem to a spanning tree. The [determinant](/citadel/cp/matrix-determinant) underlies orientation and area; [continued fractions](/citadel/cp/continued-fractions) show up in [lattice-point counting](/citadel/cp/lattice-points-under-segment).
