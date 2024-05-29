---
title: Linear Algebra for Competitive Programming - Gauss and Determinants
description: The short contest linear-algebra toolkit — Gaussian elimination for real systems and for XOR systems over GF(2), matrix rank, determinants, and where matrix exponentiation and Kirchhoff's theorem plug in.
date: 2024-05-29
draft: false
slug: /cp/linear-algebra
tags:
  - Competitive Programming
  - Linear Algebra
  - Matrix Algorithms
---

Contest linear algebra is a small kit: solve a linear system, compute a rank, compute a determinant. Everything is $O(n^3)$ Gaussian elimination with different arithmetic — real numbers, integers modulo a prime, or bits under XOR. The [maths linear algebra post](/citadel/maths/linear-algebra) covers the theory; this is the algorithmic checklist.

## The two workhorses

- [Gaussian elimination](/citadel/cp/gauss-linear-system) — row-reduce $A\mathbf{x} = \mathbf{b}$ to find a solution, the solution space dimension, or "no solution". Over $\mathbb{R}$ use partial pivoting for stability; modulo a prime use a [modular inverse](/citadel/cp/modular-inverse) for the pivot; over **GF(2)** (linear equations on bits, XOR constraints) pack rows into bitsets and elimination is `row ^= pivot_row` — this is the **XOR basis / linear basis** used for "max XOR subset", "is this value reachable by XOR", "count distinct XOR values".
- [Matrix determinant](/citadel/cp/matrix-determinant) — the product of pivots after elimination, with a sign flip per row swap. Modulo a prime it is exact; the fraction-free Bareiss (Crout-style) variant keeps it integer.

## What plugs into these

- **Matrix exponentiation** — raise a $k \times k$ transition matrix to the $n$-th power in $O(k^3 \log n)$ to evaluate a linear recurrence or count length-$n$ walks. Built in the [Fibonacci](/citadel/cp/fibonacci-numbers) post.
- **[Kirchhoff's theorem](/citadel/cp/kirchhoff-theorem)** — the number of spanning trees of a graph is any cofactor of its Laplacian, i.e. one determinant.
- **[Matrix-tree / BEST theorem](/citadel/cp/eulerian-path)**, **cycle space and cut space** over GF(2), **[2-SAT](/citadel/cp/two-sat)**-adjacent bit tricks.
- **Linear recurrence recovery** — Berlekamp-Massey finds the shortest recurrence fitting a sequence; combined with matrix power (or Kitamasa) it extrapolates in $O(k^2 \log n)$.

## Recognising it

- "solve these equations", "how many free variables" → Gaussian elimination.
- "constraints are XOR of subsets equals something", "largest XOR you can make" → GF(2) elimination / linear basis.
- "count spanning trees", "count perfect matchings of a planar graph" → a determinant.
- "$n$-th term of a linear recurrence, $n$ up to $10^{18}$" → matrix power (or Berlekamp-Massey + Kitamasa).

## Where this goes next

The determinant reappears in [combinatorics](/citadel/cp/combinatorics) (counting via the matrix-tree theorem, Lindström-Gessel-Viennot for non-crossing paths) and in [geometry](/citadel/cp/geometry) (the cross product *is* a $2 \times 2$ determinant; orientation and area tests are its sign and magnitude).
