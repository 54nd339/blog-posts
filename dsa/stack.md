---
title: Stack - Last In, First Resolved
description: A guide to the Stack section of NeetCode 150 — plain LIFO handles matching and expression evaluation, and the monotonic variant answers "nearest greater or smaller element" for a whole array in linear time.
date: 2021-06-29
draft: false
slug: /dsa/stack
tags:
  - Competitive Programming
  - Stack
  - Monotonic Stack
---

A stack is the right structure whenever the most recent unresolved thing is the next thing you must deal with. Two sub-patterns cover this whole section: the plain stack for nesting and evaluation, and the *monotonic* stack for "next greater / next smaller element" queries.

## Plain stack: nesting and evaluation

Push when you start something, pop when you finish it, and the top is always the innermost open context.

- [Valid Parentheses](/citadel/dsa/valid-parentheses) — push openers, match each closer against the top.
- [Min Stack](/citadel/dsa/min-stack) — augment each entry with the running minimum so `getMin` is $O(1)$.
- [Evaluate Reverse Polish Notation](/citadel/dsa/evaluate-reverse-polish-notation) — push operands, let each operator reduce the top two.
- [Generate Parentheses](/citadel/dsa/generate-parentheses) — the recursion's call stack *is* the bracket depth; backtracking prunes invalid branches.

## Monotonic stack: nearest greater or smaller

Keep the stack's values sorted (increasing or decreasing). When a new element breaks the order, pop the violators — and each pop is the exact moment that popped element's "next greater/smaller" neighbour is discovered. Every element is pushed and popped once, so the whole array is processed in $O(n)$.

- [Daily Temperatures](/citadel/dsa/daily-temperatures) — decreasing stack of day indices; a warmer day resolves all cooler days below it.
- [Car Fleet](/citadel/dsa/car-fleet) — sort by position, stack of fleet arrival times; a later time starts a new fleet.
- [Largest Rectangle in Histogram](/citadel/dsa/largest-rectangle-in-histogram) — increasing stack of bar indices; popping a taller bar fixes its right boundary.

The [Sliding Window Maximum](/citadel/dsa/sliding-window-maximum) deque from the previous section is the same idea with removal allowed at both ends.

## Recognising it

Plain stack when:

- The input has nesting or scope (brackets, tags, function calls).
- You are evaluating or parsing an expression.
- You need "undo" or "most recent" semantics.

Monotonic stack when:

- The brute force is "for each `i`, scan left or right until you find a bigger/smaller value".
- The answer for each position depends on the nearest element that beats it in some ordering.
- You are computing spans, ranges, or areas bounded by "the first thing shorter/taller than me".

## Where this goes next

The monotonic stack's amortised analysis — each element touched a constant number of times despite a nested loop — is the same charging argument that makes [union-find](/citadel/dsa/graphs) and many greedy sweeps linear. The recursion stack view of [Generate Parentheses](/citadel/dsa/generate-parentheses) leads straight into [Backtracking](/citadel/dsa/backtracking).
