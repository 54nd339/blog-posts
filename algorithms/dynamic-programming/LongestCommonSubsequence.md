---
title: Longest Common Subsequence - The DP Behind diff and DNA Alignment
description: Find the longest sequence of characters appearing in order in two strings, not necessarily contiguously. A brute-force search is exponential; a table of prefix-versus-prefix subproblems solves it in O(nm) and reconstructs the subsequence itself.
date: 2021-03-21
draft: false
slug: /algorithms/LongestCommonSubsequence
tags:
  - Algorithms
  - Dynamic Programming
---

A **subsequence** of a string is what's left after deleting zero or more characters without reordering the rest. `"GTAB"` is a subsequence of both `"AGGTAB"` and `"GXTXAYB"` — the letters appear in that order in each, just not next to each other. The **longest common subsequence** (LCS) of two strings is the longest such shared string. It's how `diff` finds the unchanged lines between two files, and how bioinformatics aligns DNA.

Checking every subsequence of one string against the other is $2^n$ work. Dynamic programming does it in $O(nm)$.

## The idea

Compare the two strings from their ends. Let `dp[i][j]` be the LCS length of the first `i` characters of `s1` and the first `j` of `s2`. Look at the last characters of those prefixes:

- **They match.** That character ends the LCS. Whatever the best alignment of the shorter prefixes was, add one: `dp[i][j] = 1 + dp[i-1][j-1]`.
- **They don't match.** At least one of the two last characters is not in the LCS, so drop one and take the better result: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.
- **Either prefix is empty.** The LCS is `0`.

Every `dp[i][j]` depends only on cells above and to the left, so a single pass fills the table.

## The code

Tabulation, bottom-up:

```python
def lcs_length(s1, s2):
    n, m = len(s1), len(s2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if s1[i - 1] == s2[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[n][m]


assert lcs_length("AGGTAB", "GXTXAYB") == 4    # "GTAB"
```

To recover the subsequence itself, walk back from `dp[n][m]`: a diagonal step where the characters matched contributed a letter; otherwise move toward the larger neighbour.

```python
def lcs(s1, s2):
    n, m = len(s1), len(s2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if s1[i - 1] == s2[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    out = []
    i, j = n, m
    while i > 0 and j > 0:
        if s1[i - 1] == s2[j - 1]:
            out.append(s1[i - 1])
            i, j = i - 1, j - 1
        elif dp[i - 1][j] >= dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
    return "".join(reversed(out))


assert lcs("AGGTAB", "GXTXAYB") == "GTAB"
```

The recursive form is the same recurrence with an `@lru_cache` on `(i, j)`.

## Cost

There are $(n+1)(m+1)$ subproblems, each $O(1)$: **$O(nm)$ time**, $O(nm)$ space. If you only need the length, keep just the previous row and drop to $O(\min(n, m))$ space — but then you lose the table needed to reconstruct the string.

## Where it leads

LCS is the template for sequence-alignment DP: edit distance, `diff`, and DNA alignment are all this recurrence with different costs on match, insert, and delete. The same fill-a-table-of-subproblems structure powers [0/1 knapsack](/citadel/algorithms/01Kanpsack) and [matrix chain multiplication](/citadel/algorithms/MatrixChainMultiplication) — what changes between them is only the shape of the subproblem and the recurrence tying them together.
