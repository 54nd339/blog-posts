---
title: Combinatorics for Competitive Programming - Counting Techniques
description: A map of contest counting — binomial coefficients modulo a prime, the standard bijections (stars and bars, Catalan), inclusion-exclusion, and Burnside for counting up to symmetry.
date: 2024-06-08
draft: false
slug: /cp/combinatorics
tags:
  - Competitive Programming
  - Combinatorics
  - Counting
---

Counting problems come in a few recognisable shapes: choose a subset (binomials), distribute identical items (stars and bars), count balanced structures (Catalan), count while removing bad cases (inclusion-exclusion), or count distinct configurations under symmetry (Burnside). Almost all of it runs on the [algebra section's](/citadel/cp/algebra) modular-inverse machinery.

## The foundation: n choose k modulo a prime

[Binomial coefficients](/citadel/cp/binomial-coefficients) modulo a prime $p$: precompute factorials and inverse factorials in $O(n)$, then $\binom{n}{k} = n!\,(k!)^{-1}\,((n-k)!)^{-1}$ in $O(1)$. For $n$ larger than $p$, [Lucas' theorem](/citadel/cp/binomial-coefficients) combines base-$p$ digits. This one primitive underlies Catalan numbers, paths in a grid, stars and bars, and most DP-free counting.

## The standard bijections

- **[Stars and bars](/citadel/cp/stars-and-bars)** — the number of ways to write $n$ as an ordered sum of $k$ non-negative integers is $\binom{n + k - 1}{k - 1}$. The go-to for "distribute $n$ identical objects into $k$ boxes", with add-ons for lower/upper bounds.
- **[Catalan numbers](/citadel/cp/catalan-numbers)** $C_n = \frac{1}{n+1}\binom{2n}{n}$ — balanced brackets, triangulations, binary trees, Dyck paths, non-crossing matchings. Recognise them by "count the structures that never go negative".
- **[Balanced bracket sequences](/citadel/cp/balanced-bracket-sequences)** — Catalan with ranking/unranking (find the $k$-th, or the index of a given one).
- **Cycles, derangements, Stirling and Bell numbers** — permutation statistics; derangements $D_n = (n-1)(D_{n-1} + D_{n-2})$.

## Removing the bad cases: inclusion-exclusion

[Inclusion-exclusion](/citadel/cp/inclusion-exclusion): $|A_1 \cup \dots \cup A_m| = \sum |A_i| - \sum |A_i \cap A_j| + \dots$. Used for "count arrangements avoiding a set of forbidden patterns", coprime counts via the [Möbius function](/citadel/cp/eulers-totient-function), surjection counts, and the derangement formula. Often the $2^m$ subset sum is the whole algorithm; sometimes Möbius makes it $O(n \log n)$.

## Counting up to symmetry: Burnside

[Burnside's lemma / Polya enumeration](/citadel/cp/burnside-polya): the number of distinct colourings under a group $G$ of symmetries is $\frac{1}{|G|}\sum_{g \in G} |\text{Fix}(g)|$ — average the number of colourings each symmetry leaves unchanged. Necklaces, bracelets, cube faces, grids under rotation/reflection.

## Miscellany with a combinatorial flavour

- **[Placing bishops](/citadel/cp/placing-bishops-on-a-chessboard)** — a diagonal-DP counting problem.
- **[Counting labeled graphs](/citadel/cp/counting-labeled-graphs)** — connected vs all, via an exponential-generating-function recurrence.
- **[Legendre's formula](/citadel/cp/binomial-coefficients)** — the exponent of a prime in $n!$, for divisibility of binomials and factorials.

## Recognising the technique

- "how many ways to choose / arrange" with a clean formula → binomials.
- "distribute identical items", "compositions of $n$" → stars and bars.
- "never dips below zero", "balanced", "non-crossing" → Catalan.
- "count things avoiding a set of properties" → inclusion-exclusion (or Möbius).
- "distinct under rotation/reflection" → Burnside.

## Where this goes next

The [determinant](/citadel/cp/matrix-determinant) gives more counting: [Kirchhoff](/citadel/cp/kirchhoff-theorem) for spanning trees, Lindström-Gessel-Viennot for non-intersecting paths. And [generating functions](/citadel/cp/operations-on-polynomials) — power series whose coefficients are your counts — tie the whole section together with FFT-speed convolution.
