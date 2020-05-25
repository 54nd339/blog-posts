---
title: Topology and Fractals - Shape Without Distance, Detail Without End
description: Two geometries that each drop something Euclid kept. Topology forgets distance and keeps only what survives stretching — homeomorphism, the Euler characteristic worked on several solids, genus, the Möbius strip and Klein bottle. Fractal geometry keeps detail at every scale and lets dimension be a fraction, with the Koch, Sierpiński, and Mandelbrot examples computed.
date: 2020-05-25
draft: false
slug: /maths/topology-and-fractals
tags:
  - Mathematics
  - Geometry
---

A topologist, the joke goes, cannot tell a coffee mug from a doughnut. It sounds like a punchline about absent-mindedness. It is actually a precise mathematical statement: the mug and the doughnut each have exactly one hole, and one can be continuously deformed into the other — flatten the cup, shrink the handle to a ring — without cutting or gluing anything. To a topologist those are *the same object*, and a sphere is a genuinely different one, because you cannot make a hole by stretching.

[Coordinate geometry](/citadel/maths/3d-geometry) is built on distance and angle. The two geometries here each throw one thing away and find that less structure reveals more. **Topology** discards measurement entirely and asks only what survives continuous deformation — and discovers that a single integer, the Euler characteristic, plus a yes/no on orientability, classifies *every* closed surface there can be. **Fractal geometry** keeps measurement but discards the assumption that a shape must look smooth once you zoom in far enough — and finds that dimension is naturally a real number, not an integer. This post covers homeomorphism and the surface invariants, the one-sided oddities, then self-similarity, the box-counting dimension, and the Koch curve, Sierpiński triangle, and Mandelbrot set worked through.

---
## Topology: what open sets are enough for

A **topological space** is a set of points plus a chosen collection of subsets called **open sets**, satisfying three axioms: the empty set and the whole space are open; any union of open sets is open; any *finite* intersection of open sets is open. There is no distance function anywhere in that definition.

The surprise is how much still works. With only open sets you can define **continuity** — a map is continuous if the preimage of every open set is open — and that definition agrees with the $\varepsilon$–$\delta$ one on the real line while also making sense on spaces where "$\varepsilon$" would be meaningless (the space of all continuous functions, say, or a finite set of web pages linked by clicks). You can define **connectedness** (the space is not two disjoint open pieces), **compactness** (every open cover has a finite subcover — the abstract version of "closed and bounded"), and **convergence**, all without a ruler.

---
## Homeomorphism and the invariants

Two spaces are **homeomorphic** if there is a continuous bijection between them whose inverse is also continuous — each can be stretched and bent into the other with no tearing and no gluing. This is topology's "$=$". A property that is shared by all homeomorphic spaces is a **topological invariant**, and invariants are how you *prove* two spaces are different: exhibit one invariant they disagree on.

- **Connectedness** — one piece or several. A line minus a point is disconnected; a plane minus a point is not.
- **Compactness** — an open interval $(0,1)$ is not compact, $[0,1]$ is; so they are not homeomorphic despite looking similar.
- **Genus** $g$ — the number of holes: $0$ for a sphere, $1$ for a torus (doughnut), $2$ for a double torus.
- **Euler characteristic** $\chi$ — for any polyhedron, or any surface cut into polygonal faces,
  $$ \chi = V - E + F, $$
  vertices minus edges plus faces, and this number does not depend on how you cut it.

**Euler's formula, checked on several solids.**

| Solid | $V$ | $E$ | $F$ | $V - E + F$ |
| --- | --- | --- | --- | --- |
| Tetrahedron | 4 | 6 | 4 | 2 |
| Cube | 8 | 12 | 6 | 2 |
| Octahedron | 6 | 12 | 8 | 2 |
| Icosahedron | 12 | 30 | 20 | 2 |
| Torus (as a grid of quads) | 16 | 32 | 16 | 0 |

Every solid that can be inflated to a sphere gives $\chi = 2$; the torus gives $\chi = 0$. In general, for an orientable closed surface of genus $g$,

$$ \chi = 2 - 2g. $$

Since $\chi$ is a homeomorphism invariant, a sphere ($\chi = 2$) and a torus ($\chi = 0$) are provably not the same surface — no clever stretching connects them. The mug and the doughnut both have $\chi = 0$ and $g = 1$, and indeed they are homeomorphic.

---
## One-sided surfaces

The **Möbius strip** is a paper band given a half-twist before its ends are joined. It has one side and one edge: run a pencil along the middle and you return to the start having covered "both" faces without lifting the pencil, and run it along the boundary and you traverse the entire edge in one loop. It is **non-orientable** — there is no way to define "clockwise" consistently over the whole surface, because a small clock slid once around the strip comes back mirror-reversed.

Glue two Möbius strips along their single edges and you get a **Klein bottle**: a closed non-orientable surface with no inside and no outside. It cannot be built in three dimensions without passing through itself, but it embeds cleanly in four. The **classification theorem for closed surfaces** is the payoff of the whole subject: every closed surface is exactly one of a sphere with $g$ handles attached (orientable, $\chi = 2 - 2g$) or a sphere with $k$ cross-caps attached (non-orientable, $\chi = 2 - k$). Two numbers — $\chi$ and an orientability bit — and the list is complete. Nothing else can exist.

---
## Fractal geometry: detail that never resolves

A **fractal** is a shape that stays intricate at every magnification — rough in a structured, repeating way rather than smoothing out. Benoît Mandelbrot coined the term in 1975, from the Latin *fractus*, "broken." The defining features:

1. **Self-similarity** — the whole is made of smaller copies of itself, exactly (the Koch curve) or statistically (a coastline: any stretch, zoomed, looks like coastline).
2. **Detail at all scales** — zooming in always reveals more structure, never a limiting smooth curve.
3. **Non-integer dimension** — a number capturing how fast that detail accumulates as you look closer.

### Making dimension a number

Cut a line segment into scaled-down copies of itself: $N = 2$ pieces each shrunk by factor $S = 2$. A square: $N = 4$ pieces at $S = 2$. A cube: $N = 8$ at $S = 2$. The relationship is $N = S^D$, so the exponent is

$$ D = \frac{\log N}{\log S}, $$

giving $D = 1, 2, 3$ for the line, square, cube — the ordinary dimensions. Now apply it to the **Koch curve**: replace the middle third of a segment with the two other sides of an equilateral triangle, and repeat forever. Each step turns one piece into $N = 4$ pieces, each scaled by $S = 3$:

$$ D = \frac{\log 4}{\log 3} \approx 1.262. $$

More than a line, less than a region — it "fills space" at a rate between the two. The **Sierpiński triangle** (subdivide into four, delete the middle, recurse) has $N = 3$, $S = 2$, so $D = \log 3 / \log 2 \approx 1.585$.

For shapes without exact self-similarity, the **box-counting dimension** generalises the idea: cover the set with a grid of boxes of side $\varepsilon$, count how many $N(\varepsilon)$ are non-empty, and take

$$ D = \lim_{\varepsilon \to 0} \frac{\log N(\varepsilon)}{\log(1/\varepsilon)}. $$

The **Hausdorff dimension** is the rigorous version used in proofs; for self-similar sets all three agree. Many fractals arise as the **attractor** of an *iterated function system* — a handful of contraction maps whose combined image, applied repeatedly, converges to one fixed intricate set no matter what shape you start from.

![The Mandelbrot set: a solid black shape — a large heart-shaped cardioid with a circular bulb to its left and progressively smaller buds around the edge — against a blue background, its entire boundary fringed with fine self-similar filaments and miniature copies of the whole set.](../images/mandelbrot-set.jpg "The Mandelbrot set: the complex numbers c for which iterating z ↦ z² + c from 0 stays bounded. The set is connected; its boundary has fractal dimension 2. Source: Wikimedia Commons.")

### The classic fractals

- **Koch snowflake** — the Koch curve on all three sides of a triangle. Its enclosed area converges to $\tfrac{8}{5}$ of the starting triangle's, but its perimeter multiplies by $\tfrac43$ every step and **diverges**: a finite area wrapped in an infinitely long boundary.
- **Sierpiński triangle** — dimension $\approx 1.585$; it is also exactly the pattern of odd entries in Pascal's triangle, and the image an IFS of three half-scale maps converges to.
- **Mandelbrot set** — the complex numbers $c$ for which iterating $z \mapsto z^2 + c$ from $z = 0$ never escapes to infinity. The set itself is connected and has positive area; its *boundary* is a fractal of dimension $2$ — as crinkled as a curve can possibly be. Freeze $c$ and vary the starting $z$ instead and you get the matching **Julia set**, the dividing line between starting points that escape and those that stay bounded. All of this lives in the [complex plane](/citadel/maths/complex-numbers) and comes from one quadratic map iterated.

Fractal structure is not exotic: coastlines (measured length grows without bound as the ruler shrinks — Richardson's data, Mandelbrot's "How Long Is the Coast of Britain?"), snowflakes, lightning, river basins, the branching of lungs, blood vessels, and trees, which all pack maximal surface or reach into a fixed volume. Applications include procedural terrain in graphics, compact multi-band antennas, and the fractal structure of [chaotic strange attractors](/citadel/maths/applied-maths).

---
## The one idea to keep

Both geometries are subtractions from Euclid that end up revealing more than they discard. Topology keeps only what continuous deformation cannot destroy, and gets a complete classification of closed surfaces out of two invariants: the Euler characteristic $\chi = V - E + F$ (which is $2$ for every sphere-like solid and $2 - 2g$ for genus $g$) and orientability. Fractal geometry keeps roughness instead of idealising it away, and finds dimension is a real number $D = \log N / \log S$ measuring how detail multiplies under magnification — $1.262$ for the Koch curve, $2$ for the Mandelbrot boundary. Between them they give a working vocabulary for shapes — a knotted protein, a jagged coastline, a branching lung — that distance-and-angle geometry cannot describe.
