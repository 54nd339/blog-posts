---
title: Two Pointers - Turning a Nested Loop Into One Sweep
description: A guide to the Two Pointers section of NeetCode 150 — when a monotonic rule tells you which of two indices to advance, an O(n squared) pair search becomes a single O(n) pass with O(1) space.
date: 2021-05-09
draft: false
slug: /dsa/two-pointers
tags:
  - Competitive Programming
  - Two Pointers
  - Arrays & Hashing
---

Five problems, one shape: two indices moving through the data, and a rule that decides which one moves next. When that rule is *monotonic* — advancing a pointer permanently rules out possibilities that can never come back — the whole search finishes in one linear pass, using no extra memory.

This is the section where "sort it first" starts earning its keep, because a sorted array is what makes the move rule monotonic.

## The two layouts

**Converging pointers.** One at each end, walking inward until they meet. Used when the objective depends on both endpoints and you can argue that the worse endpoint pairs badly with everything still in range.

- [Valid Palindrome](/citadel/dsa/valid-palindrome) — compare ends, skip non-alphanumeric characters in place.
- [Two Sum II](/citadel/dsa/two-sum-ii) — sum too small, raise the low pointer; too big, lower the high pointer.
- [3Sum](/citadel/dsa/3sum) — fix the smallest element, converge the other two, skip equal neighbours to dedupe.
- [Container With Most Water](/citadel/dsa/container-with-most-water) — always move the shorter wall; the taller one can't improve.
- [Trapping Rain Water](/citadel/dsa/trapping-rain-water) — advance whichever side has the smaller running max.

**Trailing pointers (fast/slow).** Both start at the same end; one runs ahead by a fixed offset or a fixed rule. This is the same idea applied to linked lists and in-place array rewriting — see [Linked List Cycle](/citadel/dsa/linked-list-cycle) and [Remove Nth Node From End of List](/citadel/dsa/remove-nth-node-from-end-of-list).

## Why it is linear

Each step advances at least one pointer toward the other, and neither ever backtracks. So the pointers cover a combined distance of `n` and the loop runs $O(n)$ times. The catch is *proving the move is safe*: when you advance `lo`, you must be able to say "no better answer uses `nums[lo]` with anything left in `[lo+1, hi]`". Every problem above has a one-sentence version of that argument, and finding it is the actual work.

## Recognising it

Reach for two pointers when:

- The brute force is "for each `i`, loop over `j` to find a partner", and the array is sorted or can be sorted.
- The answer involves a pair of endpoints and a quantity like distance, sum, or `min` of the two.
- You are asked for $O(1)$ extra space and the hash-map solution would cost $O(n)$.

Stay with a hash map instead when the array *cannot* be reordered (indices matter and you can't carry them) or the move rule would not be monotonic.

## Where this goes next

Slide the two pointers in the *same* direction, keeping a window between them, and you have the [Sliding Window](/citadel/dsa/sliding-window) pattern — the next section, and the natural sequel to this one.
