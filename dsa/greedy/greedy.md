---
title: Greedy - When the Local Choice Is Provably Global
description: A guide to the Greedy section of NeetCode 150 — each problem has a one-pass rule that is optimal because of a structural fact, most often a running frontier, a monotone operation, or a forced move.
date: 2022-05-13
draft: false
slug: /dsa/greedy
tags:
  - Competitive Programming
  - Greedy
  - Arrays
---

A greedy algorithm commits to the best-looking choice at each step and never revisits it. That only works when a structural property guarantees the local choice cannot block a better global outcome. Every problem here has such a property; the skill is spotting it and trusting it.

## The recurring justifications

- **A running frontier is all that matters.** The only useful fact about the past is one number — the farthest index reachable, the best sum ending here.
- **The operation is monotone.** Element-wise `max`, or "water only rises" — so anything that overshoots the goal is unusable and can be filtered out.
- **The move is forced.** The smallest remaining element has nowhere else to go, so it must start the next group.
- **A negative prefix is dead weight.** If a running accumulator dips below zero, drop it and restart — no start inside that prefix can be optimal.

## Problem → rule

- [Maximum Subarray](/citadel/dsa/maximum-subarray) — Kadane: `cur = max(x, cur + x)`; abandon a negative running sum.
- [Jump Game](/citadel/dsa/jump-game) — keep the farthest reachable index; fail if you fall behind it.
- [Jump Game II](/citadel/dsa/jump-game-ii) — same frontier, count how many times you must extend it (BFS by ranges).
- [Gas Station](/citadel/dsa/gas-station) — feasible iff `sum(gas) >= sum(cost)`; the start is just past where the running tank bottoms out.
- [Hand of Straights](/citadel/dsa/hand-of-straights) — the smallest card must open a run; consume `groupSize` consecutive values.
- [Merge Triplets to Form Target Triplet](/citadel/dsa/merge-triplets-to-form-target-triplet) — element-wise `max` is monotone; discard any triplet that overshoots, then check coverage.
- [Partition Labels](/citadel/dsa/partition-labels) — a part must stretch to the last occurrence of every letter it holds; cut when the cursor meets that running max.
- [Valid Parenthesis String](/citadel/dsa/valid-parenthesis-string) — carry `[lo, hi]`, the range of possible open counts, instead of a single value.

## Recognising it

- "maximum / minimum number of ...", "can you reach ...", "fewest ... to ..." with an array or string.
- A DP would work but feels heavy, and there is an obvious "just take the biggest / nearest / last" move.
- You can argue — in one or two sentences — why that move never forecloses a better answer. If you cannot, it is probably DP.

## The catch

Greedy fails silently. [Coin Change](/citadel/dsa/coin-change) with `coins = [1,3,4], amount = 6` is the standard warning: greedy takes `4 + 1 + 1`, the optimum is `3 + 3`. Always have the justification before you trust the one-pass.

## Where this goes next

[Intervals](/citadel/dsa/intervals) is greedy with a specific object: sort the ranges by start or end, then sweep once, merging or counting. The "sort, then one linear pass" structure is the whole section.
