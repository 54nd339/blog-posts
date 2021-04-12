---
title: Arrays and Hashing - The Pattern Behind the First Nine
description: A guide to the Arrays & Hashing section of NeetCode 150 — the one shared idea is replacing a repeated scan with an O(1) hash lookup, and each problem is a different disguise for it.
date: 2021-04-12
draft: false
slug: /dsa/arrays-and-hashing
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Hash Table
---

Nine problems open the NeetCode 150, and they all teach one reflex: when a brute-force solution keeps re-scanning the array to answer "have I seen this?" or "how many of these are there?", replace that scan with a hash set or hash map and the time drops from $O(n^2)$ to $O(n)$.

The cost is $O(n)$ memory and, sometimes, giving up the natural ordering of the data. That trade is almost always worth it, and knowing *when it is not* — when sorting or a fixed-size count array is the better tool — is the rest of the lesson.

## The core move

A hash map answers three questions in $O(1)$ average time:

- **Membership** — is `x` present? (`set`)
- **Association** — what value did I store for key `x`? (`dict`)
- **Frequency** — how many times has `x` occurred? (`Counter` / `dict` of counts)

Most $O(n^2)$ array solutions are doing one of these the slow way, inside a loop. Spot which one, and the fix writes itself.

## The nine problems

| Problem | The disguise | The hashing move |
| --- | --- | --- |
| [Contains Duplicate](/citadel/dsa/contains-duplicate) | any repeat? | set of seen values, check before insert |
| [Valid Anagram](/citadel/dsa/valid-anagram) | same multiset of letters? | frequency table as a canonical form |
| [Two Sum](/citadel/dsa/two-sum) | which pair sums to target? | map value → index, look up the complement |
| [Group Anagrams](/citadel/dsa/group-anagrams) | cluster equivalent words | dict keyed by a canonical signature |
| [Top K Frequent Elements](/citadel/dsa/top-k-frequent-elements) | k most common values | count map, then bucket sort by count |
| [Encode and Decode Strings](/citadel/dsa/encode-and-decode-strings) | serialise a list of strings | length-prefix framing (no hashing, but same section's "design a format" muscle) |
| [Product of Array Except Self](/citadel/dsa/product-of-array-except-self) | all-but-one product, no division | prefix/suffix accumulators |
| [Valid Sudoku](/citadel/dsa/valid-sudoku) | any rule broken? | one seen-set per row, column, and box |
| [Longest Consecutive Sequence](/citadel/dsa/longest-consecutive-sequence) | longest run of consecutive ints | set membership, count only from run starts |

Two of these — Product of Array Except Self and Encode/Decode Strings — are in the section for a different reason: they train the "array as scratch space" and "design an unambiguous format" habits that recur later. The other seven are pure hashing.

## How to recognise it

Reach for a hash structure when:

- The brute force has a nested loop whose inner job is "search the rest of the array for something".
- You need counts or groupings, and the keys are values (not positions).
- You want to detect a collision, a complement, or a duplicate as early as possible.

Reach for something else when:

- The alphabet is tiny and fixed — a 26-slot `list` beats a `dict` on constant factors ([Valid Anagram](/citadel/dsa/valid-anagram)).
- The sort key is a bounded integer — bucket or counting sort is $O(n)$ ([Top K Frequent Elements](/citadel/dsa/top-k-frequent-elements)).
- Order matters and you can afford $O(n \log n)$ — sorting is simpler and uses $O(1)$ extra space.

## Where this goes next

The hash map does not disappear after this section — it is the auxiliary structure inside [sliding-window](/citadel/dsa/longest-substring-without-repeating-characters) solutions, [prefix-sum](/citadel/dsa/product-of-array-except-self) counting, graph adjacency, and memoised recursion. Get comfortable here; everything downstream assumes "look it up in $O(1)$" is a tool you reach for without thinking.
