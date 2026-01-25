---
title: Finding Common Ground - The Longest Common Subsequence (LCS) Problem Explained
description: Dive into the Longest Common Subsequence (LCS) problem, a classic dynamic programming challenge. We explore how to find the length of the LCS using both iterative (tabulation) and recursive (memoization) approaches.
date: 2023-02-18
draft: false
slug: /pensieve/algorithms/longest-common-subsequence
tags:
  - DSA
  - DP
---

Hey everyone, and welcome back to the blog! In the world of computer science, especially when dealing with strings or sequences, we often encounter problems that involve finding commonalities. One such fundamental and widely studied problem is finding the **Longest Common Subsequence (LCS)** between two sequences.

It's not just an academic puzzle; understanding LCS has applications in bioinformatics (comparing DNA sequences), data comparison (like the `diff` utility), and more. The beauty of LCS lies in its elegant solution using **Dynamic Programming**. Today, let's unravel this problem, explore its solution through both iterative (tabulation) and recursive (memoization) DP approaches, and understand why it works.

## The Problem: What's a Longest Common Subsequence?
Given two strings (or sequences), `s1` and `s2`, the task is to find the length of the longest subsequence that is common to both of them.

Let's clarify what a **subsequence** is:
A subsequence is a sequence that can be derived from another sequence by deleting zero or more elements *without changing the order of the remaining elements*.

For example:
* If `s1 = "AGGTAB"`
* And `s2 = "GXTXAYB"`

A common subsequence could be "GTAB" (length 4). Notice that the characters G, T, A, B appear in that order in both `s1` and `s2`, although not necessarily contiguously. "GTAB" is, in fact, the longest common subsequence for these two strings.

The problem we're solving is to find the *length* of this longest common subsequence.

## Solving with Dynamic Programming: Two Flavors

The LCS problem exhibits optimal substructure and overlapping subproblems, the two hallmarks that make it suitable for Dynamic Programming (DP). This means we can break the problem down into smaller, related subproblems, solve them, and use their solutions to build up the solution to the original problem.

### 1. Tabulation (Iterative Bottom-Up Approach)

In the tabulation method, we build a 2D table (let's call it `dp`) from the bottom up, where `dp[i][j]` stores the length of the LCS of the first `i` characters of `s1` (i.e., `s1[0...i-1]`) and the first `j` characters of `s2` (i.e., `s2[0...j-1]`).

#### Pseudocode: Tabulation

```pseudocode
// s1, s2: The input strings
function lcs_tabulation(s1, s2):
    n = length(s1) // Length of the first string
    m = length(s2) // Length of the second string

    // Create a DP table of size (n+1) x (m+1)
    // dp[i][j] will store the length of LCS of s1[0..i-1] and s2[0..j-1]
    dp = array of size (n+1) x (m+1)

    // Fill dp table in bottom-up manner
    for i = 0 to n: // Iterate through lengths of prefix of s1
        for j = 0 to m: // Iterate through lengths of prefix of s2
            if i == 0 or j == 0:
                // If either string (or prefix) is empty, LCS is 0
                dp[i][j] = 0
            else if s1[i-1] == s2[j-1]:
                // If the last characters match, they are part of the LCS.
                // Add 1 to the LCS of the strings without these last characters.
                dp[i][j] = 1 + dp[i-1][j-1]
            else:
                // If the last characters don't match, the LCS is the maximum of:
                // 1. LCS of s1[0..i-2] and s2[0..j-1] (excluding last char of s1)
                // 2. LCS of s1[0..i-1] and s2[0..j-2] (excluding last char of s2)
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
   
    // dp[n][m] contains the length of LCS for s1[0..n-1] and s2[0..m-1]
    return dp[n][m]
```

#### Explanation of Tabulation:

1. We create a 2D table `dp` of size `(n+1) x (m+1)`, where `n` is the length of `s1` and `m` is the length of `s2`.
2. **Initialization (Base Cases):** The first row (`dp[0][j]`) and the first column (`dp[i][0]`) are filled with 0s. This is because if one of the strings (or prefixes) is empty, their longest common subsequence has a length of 0.
3. Filling the Table: We iterate through the table, filling each cell dp[i][j] based on the characters s1[i-1] and s2[j-1] (using i-1 and j-1 because our DP table is 1-indexed relative to string prefixes, while strings are often 0-indexed).
    * If `s1[i-1] == s2[j-1]`: The current characters match! This character is part of the LCS. So, the length of the LCS for `s1[0...i-1]` and `s2[0...j-1]` is 1 + the length of the LCS for `s1[0...i-2]` and `s2[0...j-2]` (which is `dp[i-1][j-1]`).
    * If `s1[i-1] != s2[j-1]`: The current characters do not match. In this case, the LCS of `s1[0...i-1]` and `s2[0...j-1]` must be the longer of two possibilities:
        * The LCS of `s1[0...i-2]` and `s2[0...j-1]` (i.e., `dp[i-1][j]`, effectively ignoring the current character of `s1`).
        * The LCS of `s1[0...i-1]` and `s2[0...j-2]` (i.e., `dp[i][j-1]`, effectively ignoring the current character of `s2`). We take the max of these two values.
4. The final answer, the length of the LCS of the entire strings `s1` and `s2`, will be stored in `dp[n][m]`.

### 2. Memoization (Recursive Top-Down Approach)

The memoization approach solves the problem recursively but stores the results of subproblems in a lookup table (memoization table) to avoid recomputing them.

#### Pseudocode: Memoization

```pseudocode
// Main function to set up memoization
function lcs(s1, s2):
    n = length(s1)
    m = length(s2)

    // Create a memoization table, initialized with a value like -1 (to indicate not computed)
    memo = array of size (n+1) x (m+1) filled with -1
   
    // Call the recursive helper function
    // Note: passing n and m representing current lengths of s1 and s2 being considered
    return lcs_memoization_helper(s1, s2, n, m, memo)

// Recursive helper function with memoization
function lcs_memoization_helper(s1, s2, i, j, memo):
    // i: current length of prefix of s1 being considered (s1[0...i-1])
    // j: current length of prefix of s2 being considered (s2[0...j-1])

    // Base Case: If either string prefix is empty
    if i == 0 or j == 0:
        return 0
   
    // If this subproblem's result is already computed, return it
    if memo[i][j] != -1:
        return memo[i][j]
   
    // If the last characters of the current prefixes match
    if s1[i-1] == s2[j-1]:
        memo[i][j] = 1 + lcs_memoization_helper(s1, s2, i-1, j-1, memo)
    else:
        // If characters don't match, explore two possibilities and take the max
        val1 = lcs_memoization_helper(s1, s2, i-1, j, memo) // Exclude char from s1
        val2 = lcs_memoization_helper(s1, s2, i, j-1, memo) // Exclude char from s2
        memo[i][j] = max(val1, val2)
       
    return memo[i][j]
```

#### Explanation of Memoization:

1. The main `lcs` function initializes a memoization table `memo` (often filled with a sentinel value like -1 to indicate that a subproblem hasn't been solved yet). It then calls a recursive helper function.
2. The `lcs_memoization_helper` function takes the current lengths `i` and `j` of the prefixes of `s1` and `s2` being considered.
3. **Base Cases:** If `i` or `j` is 0 (meaning one of the prefixes is empty), the LCS is 0.
4. **Memoization Check:** Before any computation, it checks `memo[i][j]`. If it's not -1, the result for this subproblem has already been computed, so it's returned directly.
5. **Recursive Step:**
    * If `s1[i-1] == s2[j-1]` (current characters match), the LCS length is `1 + LCS` of `s1` up to `i-1` and `s2` up to `j-1`. This result is stored in memo[i][j].
    * If they don't match, we recursively find the LCS by:
        * Excluding the last character of `s1` (call with `i-1, j`).
        * Excluding the last character of `s2` (call with `i, j-1`). The maximum of these two recursive calls is the answer for `memo[i][j]`.
6. The result stored in `memo[i][j]` is returned.

## Proof of Correctness (Intuition)

The correctness for both DP approaches stems from the optimal substructure of the LCS problem:

* **Case 1: Last characters match (s1[i-1] == s2[j-1]):** The LCS must include this character. The remaining problem is to find the LCS of the preceding substrings s1[0...i-2] and s2[0...j-2]. The length is 1 + LCS(s1_prefix_i-1, s2_prefix_j-1).
* **Case 2: Last characters don't match (s1[i-1] != s2[j-1]):** This matching pair cannot be part of the LCS. So, the LCS is either the LCS of s1[0...i-2] and s2[0...j-1] (if we discard s1[i-1]) OR the LCS of s1[0...i-1] and s2[0...j-2] (if we discard s2[j-1]). We take the maximum of these two.

Both tabulation (building solutions to subproblems iteratively) and memoization (solving subproblems recursively and storing results) correctly explore these conditions and ensure that each subproblem is solved only once.

## Space & Time Complexity Analysis

* **Time Complexity:** For both the tabulation and memoization approaches, the time complexity is **O(n * m)**, where `n` is the length of `s1` and `m` is the length of `s2`.
    * In tabulation, we fill an `(n+1) x (m+1)` table, and each cell takes constant time to compute.
    * In memoization, there are `(n+1) x (m+1)` distinct subproblems. Each subproblem is computed only once due to memoization, and each computation involves constant time work (comparisons, recursive calls to already computed or base cases).

* **Space Complexity:**
    * **Tabulation Approach:** Requires a 2D array `dp` of size `(n+1) x (m+1)` to store the LCS lengths. Thus, the space complexity is **O(n * m)**. (It's possible to optimize space to O(min(n,m)) if only the length is needed, by only keeping track of the previous row/column).
    * **Memoization Approach:** Requires a 2D array `memo` of size `(n+1) x (m+1)` for storing results, and also space for the recursion call stack, which can go up to `O(n+m)` in the worst case. So, the space complexity is dominated by the memoization table, making it **O(n * m)**.

## Key Takeaways

* The Longest Common Subsequence (LCS) problem is a classic dynamic programming problem aiming to find the length of the longest subsequence common to two given strings.
* It can be solved efficiently using either tabulation (iterative DP) or memoization (recursive DP).
* Both approaches rely on breaking the problem into overlapping subproblems and building up the solution.
* The time complexity for both standard DP solutions is O(n * m), and the space complexity is also O(n * m).

Understanding LCS and its DP solution is fundamental for tackling many sequence alignment and comparison problems in bioinformatics, text processing, and beyond!