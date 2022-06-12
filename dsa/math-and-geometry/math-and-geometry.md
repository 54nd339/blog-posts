---
title: Math & Geometry - Index Arithmetic and Digit Mechanics
description: A guide to the Math and Geometry section of NeetCode 150 — matrix problems solved by moving boundaries or decomposed transforms, and number problems solved by carry propagation, fast exponentiation, or cycle detection.
date: 2022-06-12
draft: false
slug: /dsa/math-and-geometry
tags:
  - Competitive Programming
  - Math & Geometry
  - Matrix
---

This section splits cleanly. The matrix half is about **index arithmetic** — walking a grid with shrinking boundaries, or factoring a transform into two easy passes. The number half is **digit and exponent mechanics** — carry propagation, logarithmic-time powers, and spotting that an iterated map must cycle.

## Matrix problems

- [Rotate Image](/citadel/dsa/rotate-image) — 90° clockwise = transpose, then reverse each row; or a four-way cyclic swap per ring.
- [Spiral Matrix](/citadel/dsa/spiral-matrix) — four boundaries (`top`, `bottom`, `left`, `right`) that march inward; one straight loop per edge.
- [Set Matrix Zeroes](/citadel/dsa/set-matrix-zeroes) — use row 0 and column 0 as the "which rows/cols to clear" markers, plus two scalar flags for their own status; $O(1)$ space.

The recurring idea: **boundary variables** or a **decomposition into reflections/reversals** replace fiddly per-cell direction logic.

## Number problems

- [Happy Number](/citadel/dsa/happy-number) — iterate digit-square sums; bounded values force a cycle, so it is cycle detection (hash set or Floyd).
- [Plus One](/citadel/dsa/plus-one) — carry from the last digit; prepend `1` if all digits were 9.
- [Pow(x, n)](/citadel/dsa/powx-n) — halve the exponent, square the base; $O(\log n)$; invert for negative `n`.
- [Multiply Strings](/citadel/dsa/multiply-strings) — `num1[i] * num2[j]` accumulates into `res[i + j + 1]` with carry to `res[i + j]`; the array is the number.
- [Detect Squares](/citadel/dsa/detect-squares) — hash map of point counts; fix a diagonal corner, multiply the counts of the two forced corners.

The recurring ideas: **carry propagation** (least to most significant, with a possible extra digit), **binary exponentiation**, and **"fix the defining points, look up and multiply the rest"** for geometry.

## Recognising it

- A grid to rotate, spiral, or mark → boundary walk or decomposed transform.
- A number given as a string or digit array, "without overflow" / "without big-integer types" → digit-array arithmetic.
- "x to the power n efficiently" → square-and-halve.
- "repeat this replacement until ..." over a bounded value → it cycles; detect it.
- Counting shapes over a set of points → coordinate-keyed hash map.

## Where this goes next

[Bit Manipulation](/citadel/dsa/bit-manipulation) is the same "mechanics" mindset one level lower — carries, masks, and shifts on the individual bits of a single integer.
