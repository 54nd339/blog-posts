---
title: Suffix Array - Sorting All Suffixes in O(n log n)
description: Sorting a string's suffixes by ranking them on the first 2^k characters and doubling k, then Kasai's O(n) LCP array, together answering distinct-substring counts and longest repeated substring.
date: 2024-05-15
draft: false
slug: /cp/suffix-array
tags:
  - Competitive Programming
  - String Processing
  - Suffix Structures
---

A suffix array is the list of a string's suffixes sorted lexicographically, stored as their starting indices. Paired with the LCP array (longest common prefix of adjacent sorted suffixes) it answers, in near-linear time, a whole class of "over all substrings" questions: how many distinct substrings, the longest one that repeats, the $k$-th smallest, the longest common substring of several strings.

## The problem

For $s$ of length $n$ (append a sentinel `$` smaller than every real character), compute:

- `sa[0 \dots n-1]` — starting indices of the suffixes in sorted order,
- `rank[i]` — the position of suffix $i$ within `sa` (the inverse permutation),
- `lcp[k]` — the length of the longest common prefix of `sa[k-1]` and `sa[k]`.

Example: $s = \texttt{"banana\$"}$. Sorted suffixes: `$`, `a$`, `ana$`, `anana$`, `banana$`, `na$`, `nana$` → `sa = [6, 5, 3, 1, 0, 4, 2]`. Dropping the sentinel suffix leaves `[5, 3, 1, 0, 4, 2]`, which is what the code below returns for `"banana"`.

## The idea: sort by prefixes of doubling length

Rank every suffix by its first character. Then repeatedly: rank each suffix by its first $2^k$ characters, using the previous round's ranks — the first $2^k$ chars of suffix $i$ are (rank of first $2^{k-1}$ chars of $i$, rank of first $2^{k-1}$ chars of $i + 2^{k-1}$). That is a pair of small integers, so a radix or comparison sort orders all suffixes for the next power. After $\lceil \log_2 n \rceil$ rounds, $2^k \ge n$ and the order is total.

Comparison sort per round → $O(n \log^2 n)$; radix sort per round → $O(n \log n)$. Both are fine for $n \le 5 \times 10^5$.

## Kasai's LCP in O(n)

Compute `lcp` between each suffix and the one *before it in sorted order*, walking suffixes in order of **starting index** (not sorted order). Key fact: if suffix $i$ has LCP $h$ with its sorted-predecessor, then suffix $i+1$ has LCP $\ge h - 1$ with *its* predecessor. So carry $h$, subtract $1$ each step, extend by direct comparison. Total extension work is $O(n)$.

## Algorithm

```python
def suffix_array(s: str) -> list[int]:
    s = s + "\x00"
    n = len(s)
    sa = sorted(range(n), key=lambda i: s[i])
    rank = [0] * n
    for i in range(1, n):
        rank[sa[i]] = rank[sa[i - 1]] + (s[sa[i]] != s[sa[i - 1]])
    k = 1
    tmp = [0] * n
    while k < n:
        def key(i):
            return (rank[i], rank[i + k] if i + k < n else -1)
        sa.sort(key=key)
        tmp[sa[0]] = 0
        for i in range(1, n):
            tmp[sa[i]] = tmp[sa[i - 1]] + (key(sa[i]) != key(sa[i - 1]))
        rank = tmp[:]
        if rank[sa[-1]] == n - 1:
            break
        k <<= 1
    return sa[1:]                         # drop the sentinel suffix

def lcp_array(s: str, sa: list[int]) -> list[int]:
    n = len(s)
    rank = [0] * n
    for i, p in enumerate(sa):
        rank[p] = i
    lcp = [0] * n
    h = 0
    for i in range(n):
        if rank[i] > 0:
            j = sa[rank[i] - 1]
            while i + h < n and j + h < n and s[i + h] == s[j + h]:
                h += 1
            lcp[rank[i]] = h
            if h:
                h -= 1
        else:
            h = 0
    return lcp                            # lcp[k] = LCP(sa[k-1], sa[k]); lcp[0] = 0
```

The `sa.sort(key=...)` version is $O(n \log^2 n)$; swap in a two-pass counting sort on the rank pairs for $O(n \log n)$.

## What it answers

- **Number of distinct substrings:** $\dfrac{n(n+1)}{2} - \sum_k lcp[k]$ — every substring is a prefix of some suffix; the LCP sum removes the ones counted twice.
- **Longest repeated substring:** $\max_k lcp[k]$, occurring at `sa[k]`.
- **$k$-th smallest distinct substring:** walk `sa` accumulating `(len of suffix) - lcp[k]` new substrings per entry.
- **LCP of any two suffixes $i, j$:** $\min$ of `lcp` over the range $(\,rank[i], rank[j]\,]$ — a [sparse table](/citadel/cp/sparse-table) RMQ makes it $O(1)$.
- **Longest common substring of two strings:** concatenate with a separator, build `sa` + `lcp`, take the max `lcp[k]` where `sa[k-1]` and `sa[k]` come from different strings.
- **Pattern search:** binary search the pattern as a prefix range in `sa` — $O(|P|\log n)$.

## Complexity

- **Build:** $O(n \log n)$ with radix sort, $O(n \log^2 n)$ with comparison sort.
- **LCP (Kasai):** $O(n)$.
- **Space:** $O(n)$.

## Common pitfalls

- **No sentinel.** Append a character smaller than the alphabet so no suffix is a prefix of another; otherwise sorting and LCP logic get edge cases.
- **`rank` not updated each round.** The next round's keys use *this* round's ranks; forgetting to copy `tmp` back into `rank` scrambles everything.
- **Early termination.** Stop once all ranks are distinct (`rank[sa[-1]] == n - 1`), or you waste $\log n$ rounds.
- **Kasai walks by index, not by sorted order.** The $h \ge h - 1$ invariant only holds when you iterate suffixes in increasing starting index.
- **`lcp[0]`.** Undefined / set to $0$ — there is no suffix before `sa[0]`.

## The keystone

Sort suffixes by their first $2^k$ characters using the previous round's ranks, double $k$ until the order is total — $O(n \log n)$. Kasai then builds the LCP array in $O(n)$ using "LCP drops by at most one as the start index advances", and `sa` + `lcp` together answer distinct-substring counts, longest repeats, and cross-string common substrings.
