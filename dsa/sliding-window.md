---
title: Sliding Window - Two Pointers Moving the Same Way
description: A guide to the Sliding Window section of NeetCode 150 — a contiguous range with both edges advancing rightward, plus a running summary of what is inside it, replaces a quadratic scan of all substrings.
date: 2021-06-11
draft: false
slug: /dsa/sliding-window
tags:
  - Competitive Programming
  - Sliding Window
  - Two Pointers
---

Take the two-pointer idea and point both pointers the same way. Now the region between them is a *window* — a contiguous subarray or substring — and the question becomes "what is the best window?" for some notion of best. You slide the right edge to grow the window, slide the left edge to shrink it, and keep a small running summary (a count map, a sum, a majority tally) of the window's contents so every check is $O(1)$.

Done right, this visits each element a constant number of times: $O(n)$ instead of the $O(n^2)$ or $O(n^3)$ of enumerating every substring.

## Two flavours

**Fixed-size window.** The window length is given. Slide it one step at a time; each step adds one element on the right and drops one on the left. The summary updates in $O(1)$.

- [Permutation in String](/citadel/dsa/permutation-in-string) — does any length-`len(s1)` window match `s1`'s letter counts?
- [Sliding Window Maximum](/citadel/dsa/sliding-window-maximum) — the max of every length-`k` window (needs a monotonic deque, not just a scalar).

**Variable-size window.** Grow the right edge until the window violates (or satisfies) a condition, then move the left edge to restore (or minimise). The left edge only ever moves right.

- [Best Time to Buy and Sell Stock](/citadel/dsa/best-time-to-buy-and-sell-stock) — degenerate: the left edge is pinned to the cheapest day so far.
- [Longest Substring Without Repeating Characters](/citadel/dsa/longest-substring-without-repeating-characters) — grow while distinct, jump `left` past the last duplicate.
- [Longest Repeating Character Replacement](/citadel/dsa/longest-repeating-character-replacement) — valid while `length - majority_count <= k`.
- [Minimum Window Substring](/citadel/dsa/minimum-window-substring) — grow to cover the target, shrink while still covering.

## Why it is linear

Both edges move only rightward and never past the end, so together they take at most `2n` steps. The requirement is that the window's validity be *monotonic* in a way you can exploit: for "longest valid window", once the window is invalid, only shrinking from the left can help; for "shortest window meeting a condition", once it meets the condition, only shrinking can improve it. If validity is not monotonic, a plain sliding window will not work.

## Choosing the running summary

- A **scalar** (sum, min, max-so-far) when the condition is arithmetic.
- A **hash map / count array** when the condition is about character or value frequencies.
- A **monotonic deque** when you need the min or max *of the current window* and it can leave from either end.
- A **have/need + formed counter** when the condition is "covers a target multiset".

## Where this goes next

The monotonic-deque and monotonic-stack machinery introduced by [Sliding Window Maximum](/citadel/dsa/sliding-window-maximum) is the whole point of the next section, [Stack](/citadel/dsa/stack).
