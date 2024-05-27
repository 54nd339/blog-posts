---
title: Finding Repetitions - Main-Lorentz and Tandem Repeats
description: Locating every maximal repetition (run) in a string in O(n log n) by divide and conquer, counting cross-boundary tandem repeats with Z-functions, and why the number of runs is only O(n).
date: 2024-05-27
draft: false
slug: /cp/finding-repetitions
tags:
  - Competitive Programming
  - String Processing
  - String Algorithms
---

A **repetition** (or *run*) is a maximal substring that is periodic with period at most half its length — like `abcabcab` (period $3$). Runs encode all the "square" and "cube" structure of a string. The Main-Lorentz algorithm finds every run in $O(n \log n)$ by divide and conquer; remarkably, there are only $O(n)$ runs in any string, so the output is small.

## The problem

Find all maximal repetitions of $s$: triples $(l, r, p)$ where $s[l \dots r]$ has smallest period $p$, $r - l + 1 \ge 2p$, and the periodicity cannot be extended left or right. From these you can count distinct squares, find the longest tandem repeat, test if $s$ contains any square, etc.

Example: $s = \texttt{"aabaabaaaabaaba"}$ contains the run `"aabaabaa"` (period $3$), `"aa"` at several places (period $1$), `"aabaabaaaabaaba"`-internal structure, and so on.

## The idea (Main-Lorentz)

Split $s$ into halves $u = s[0 \dots m)$ and $v = s[m \dots n)$. Every repetition either lies wholly in $u$, wholly in $v$, or **crosses the midpoint**. Recurse on the two halves; the work is finding the crossing repetitions.

A crossing repetition of period $p$ is a **tandem repeat** — two consecutive equal blocks of length $p$ — straddling position $m$. Classify by which half contains the block boundary (the "cntr"):

- for repetitions whose left block ends in $u$ (a *left* crossing), the block starts in $u$ and the repeat extends into $v$;
- symmetric for *right* crossings.

For each period $p$, whether a tandem repeat of period $p$ crosses at a given offset is decided in $O(1)$ using:

- $Z_1$ = the Z-function of $u$ reversed (longest common prefix of a suffix of $u$ with $u$ reversed — i.e. how far a match extends leftward),
- $Z_2$ = the Z-function of $v + \#\ + u$ (how far a match extends rightward from a point in $u$ into $v$).

Summing over all $p$ and both crossing types, the midpoint work is $O(m)$ per level, $O(n \log n)$ total.

## Why there are only O(n) runs

The **runs theorem** (Bannai et al., 2017): a string of length $n$ has fewer than $n$ runs, and the sum of their **exponents** ($\lfloor \text{length}/p \rfloor$) is $O(n)$. So enumerating runs and doing $O(1)$–$O(\log)$ work per run is efficient. Linear-time run-finding is possible (via [suffix arrays / Lyndon trees](/citadel/cp/lyndon-factorization)); Main-Lorentz's $O(n \log n)$ is simpler to implement.

## A practical harmonic algorithm

Instead of the divide-and-conquer, a cleaner route: for each period $p$, look only at the block **anchors** $0, p, 2p, \dots$. A run of period $p$ must fully contain two adjacent anchors $a$ and $b = a + p$. Compute how far the two blocks match to the right ($\text{lcp}(a, b)$) and to the left ($\text{lcs}(a-1, b-1)$ — longest common suffix of the prefixes ending there). If $\text{lcs} + \text{lcp} \ge p$, there is a run spanning $[\,a - \text{lcs},\ b + \text{lcp} - 1\,]$ with period $p$.

```python
def all_runs(s: str):
    n = len(s)

    def lcp(i, j):                              # O(1) with a suffix array + sparse table
        k = 0
        while j + k < n and s[i + k] == s[j + k]:
            k += 1
        return k

    def lcs(i, j):                              # longest common suffix of s[:i+1], s[:j+1]
        k = 0
        while i - k >= 0 and j - k >= 0 and s[i - k] == s[j - k]:
            k += 1
        return k

    found = {}
    for p in range(1, n // 2 + 1):
        for a in range(0, n - p, p):
            b = a + p
            right = lcp(a, b)
            left = lcs(a - 1, b - 1) if a > 0 else 0
            if left + right >= p:
                start, end = a - left, b + right - 1
                length = end - start + 1
                if 2 * p <= length and all(               # p is a period of the whole run
                        s[start + t] == s[start + t + p] for t in range(length - p)):
                    prev = found.get((start, end))
                    if prev is None or p < prev:          # keep the smallest period
                        found[(start, end)] = p

    # drop runs contained in a longer run of the same period
    runs = []
    for (st, en), p in found.items():
        if not any((s2, e2) != (st, en) and p2 == p and s2 <= st and en <= e2
                   for (s2, e2), p2 in found.items()):
            runs.append((st, en, p))
    return runs
```

With brute `lcp`/`lcs` this is $O(n^2 \log n)$; swapping in $O(1)$ longest-common-prefix queries from a [suffix array](/citadel/cp/suffix-array) + [sparse table](/citadel/cp/sparse-table) (and the mirror for `lcs`) makes it $O(n \log n)$, because $\sum_p n/p = O(n \log n)$ anchors.

For most problems you only need a specific consequence:

- **"Does $s$ contain a square?"** — check if any run has length $\ge 2 \times$ its period (every run does, so equivalently: does any run exist). $O(n \log n)$, or $O(n)$ with a suffix automaton test.
- **Longest tandem repeat** — the run with the largest $2p \le$ length; report $2p$.
- **Number of distinct squares** — bounded by $O(n)$; collect from the runs.

## Complexity

- **Main-Lorentz:** $O(n \log n)$ time, $O(n)$ output (number of runs $< n$).
- **Linear alternatives:** $O(n)$ via suffix structures + Lyndon roots, more code.

## Common pitfalls

- **Z-function direction.** Left-extension needs the Z-function of a *reversed* half; right-extension a concatenation with a separator. Mixing them up finds phantom repeats.
- **Counting the same run twice.** A run can be discovered from both crossing passes or from a recursive call at a coarser level — de-duplicate by $(l, r)$ after normalising each tandem repeat to its maximal run.
- **Period vs smallest period.** A run's period must be its *smallest* period; `abababab` has period $2$, not $4$. Reduce before recording.
- **Separator in the alphabet.** The `\x00` joins must use a symbol absent from $s$.
- **Off-by-one on `2p <= length`.** A repetition needs at least two full periods; a substring that is merely "bordered" is not a run.

## The keystone

Split the string, recurse, and find the midpoint-crossing tandem repeats with two Z-functions (one leftward on a reversed half, one rightward on a concatenation) — $O(n \log n)$ for all runs. The runs theorem caps their count below $n$, so anything you compute per run (longest square, square-free test, distinct squares) stays near-linear.
