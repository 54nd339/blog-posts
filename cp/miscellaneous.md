---
title: Miscellaneous Competitive Programming - Games, Schedules, and Puzzles
description: A map of the contest topics that do not fit the algebra, data-structure, string, geometry, or graph chapters - impartial game theory, exchange-argument scheduling, the Josephus recurrence, puzzle-solvability invariants, and the Stern-Brocot tree.
date: 2024-10-27
draft: false
slug: /cp/miscellaneous
tags:
  - Competitive Programming
  - Miscellaneous
  - Sequences
---

Every contest problem set has a few that resist classification: a two-player game, a scheduling optimisation, a counting puzzle with a hidden invariant. The techniques are small and self-contained, but each has a sharp idea you either know or you do not. This section collects them.

## Impartial games

Both players have the same moves; the last to move wins (normal play). The theory is complete:

- [Sprague-Grundy and Nim](/citadel/cp/sprague-grundy-nim) — every position has a Grundy number (the mex of its options' Grundy numbers); a sum of independent games is the **XOR** of theirs, and a position is losing iff that XOR is $0$.
- [Games on arbitrary graphs](/citadel/cp/games-on-graphs) — when the position graph has cycles, there is a third outcome, *draw*. Retrograde analysis labels every vertex WIN / LOSE / DRAW by a backward BFS from the terminal positions.

Partisan games (players have different moves — chess, checkers endgames) need surreal-number / combinatorial-game-theory machinery and are rare in contests; [game theory in the economic sense](/citadel/miscs/game-theory) (Nash equilibria, auctions) is a different subject again.

## Scheduling

[Scheduling jobs](/citadel/cp/scheduling) on machines to minimise a cost is almost always a **greedy with an exchange argument**: assume an optimal order, show that swapping an adjacent out-of-order pair does not increase the cost, conclude the sorted order is optimal. Shortest-processing-time, Smith's ratio rule, earliest-due-date, Moore-Hodgson, and Johnson's two-machine rule are all this pattern.

## Sequences and counting puzzles

- [The Josephus problem](/citadel/cp/josephus-problem) — $n$ in a circle, every $k$-th eliminated; the survivor from the $O(n)$ recurrence $J(n) = (J(n-1) + k) \bmod n$, or $O(k \log n)$ for small $k$.
- [The MEX of an array](/citadel/cp/mex-of-an-array) — the smallest missing non-negative integer, static in $O(n)$, under updates with an ordered set, and over subarrays with a segment tree of last occurrences.
- [The 15-puzzle](/citadel/cp/fifteen-puzzle-solvability) — solvable iff the tile-permutation parity matches the blank's row parity; half of all scrambles are unreachable.

## Rationals

- [The Stern-Brocot tree and Farey sequences](/citadel/cp/stern-brocot-tree) — every positive rational in lowest terms, once, as a binary search tree of mediants; the L/R path to a fraction is its [continued-fraction](/citadel/cp/continued-fractions) expansion, and it gives best rational approximations with a bounded denominator.

## Recognising the tool

- "two players, optimal play, who wins" + independent sub-games → Grundy numbers, XOR.
- "who wins" on a graph with cycles → retrograde WIN/LOSE/DRAW.
- "minimise total/max completion time / lateness / number of late jobs" → sort by the right key, prove by exchange.
- "every $k$-th removed from a circle" → Josephus recurrence.
- "is this scrambled board solvable" → a permutation-parity invariant.
- "best fraction with denominator $\le N$", "enumerate rationals" → Stern-Brocot.

## Where this connects

The [game theory](/citadel/cp/sprague-grundy-nim) here uses the mex from [MEX](/citadel/cp/mex-of-an-array); the Stern-Brocot tree is the [Euclidean algorithm](/citadel/cp/euclidean-gcd) drawn as a tree; scheduling exchange arguments are the same proof style as the [greedy](/citadel/algorithms/ActivitySelection) chapter, one level up.
