---
title: String Processing - Hashes, Automata, and Suffix Structures
description: A map of the contest string toolkit — polynomial hashing for O(1) substring equality, the linear pattern-matchers (KMP, Z, Aho-Corasick), and the suffix family that answers everything about all substrings at once.
date: 2024-05-08
draft: false
slug: /cp/string-processing
tags:
  - Competitive Programming
  - String Processing
  - String Matching
---

Almost every string problem is one of: "does pattern $P$ occur in text $T$", "compare two substrings fast", "how many distinct substrings", "longest repeated / palindromic / bordered substring". Three families of tools cover all of it — hashing, the linear automata, and the suffix structures — and knowing which to reach for is most of the battle.

## Hashing: cheap, probabilistic

[Polynomial string hashing](/citadel/cp/string-hashing) assigns each string a number so that equal strings hash equal, and a precomputed prefix-hash array gives the hash of **any substring in $O(1)$**. That single primitive solves substring comparison, [Rabin-Karp](/citadel/cp/rabin-karp) pattern search, longest common substring (binary search + hash set), and palindrome checks. The catch is collisions — use two moduli, or a 64-bit random base, and treat it as "almost certainly correct".

## The linear pattern-matchers

Deterministic, $O(n + m)$, no collision risk:

- [Prefix function / KMP](/citadel/cp/prefix-function-kmp) — for each prefix, the length of its longest proper border. Powers single-pattern search, "period of a string", "shortest string that has $s$ as a border".
- [Z-function](/citadel/cp/z-function) — for each position, the longest substring starting there that matches a prefix. Interchangeable with KMP for most tasks; sometimes cleaner.
- [Aho-Corasick](/citadel/cp/aho-corasick) — KMP generalised to a **set** of patterns: a trie plus failure links, matching all patterns in one $O(\text{text} + \text{total pattern length})$ pass. The base for "count occurrences of many needles" and for DP over "forbidden substrings".
- [Manacher](/citadel/cp/manachers-algorithm) — all maximal palindromic substrings in $O(n)$, the palindrome analogue of the Z-function.

## The suffix structures

Build once in $O(n)$ or $O(n \log n)$, then answer many substring questions:

- [Suffix array](/citadel/cp/suffix-array) (+ LCP array) — all suffixes sorted; gives distinct-substring counts, longest repeated substring, $k$-th substring, and (with a sparse table on LCP) longest common prefix of any two suffixes in $O(1)$.
- [Suffix automaton](/citadel/cp/suffix-automaton) — the minimal DFA recognising every substring; $O(n)$ states, and the cleanest route to "number of distinct substrings", "occurrences of each substring", "longest common substring of several strings".
- [Suffix tree](/citadel/cp/suffix-tree-ukkonen) — the compressed trie of all suffixes; Ukkonen builds it in $O(n)$. More powerful than a suffix array but heavier to code; the automaton usually substitutes.
- [Lyndon factorisation](/citadel/cp/lyndon-factorization) — Duval's $O(n)$ decomposition into non-increasing Lyndon words; gives the minimal string rotation for free.

## Recognising which tool

- "does $P$ occur in $T$", one pattern → KMP or Z, or a hash.
- "occurrences of many patterns" → Aho-Corasick.
- "compare arbitrary substrings", "longest common substring by binary search" → hashing.
- "count / enumerate distinct substrings", "longest repeated substring" → suffix array or suffix automaton.
- "longest palindromic substring / count palindromes" → Manacher (or hashing + binary search).
- "smallest rotation" → Lyndon (Booth's or Duval's).

## Where this goes next

String DP problems combine these with the [DP](/citadel/cp/dynamic-programming) section — Aho-Corasick automaton states become DP states for "avoid these substrings". The [expression-parsing](/citadel/cp/expression-parsing) post is the odd one out: not matching but turning an infix string into a value with a shunting-yard stack.
