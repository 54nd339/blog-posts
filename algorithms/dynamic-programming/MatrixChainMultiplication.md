---
title: Matrix Chain Multiplication - The Cheapest Order to Multiply a Chain
description: Matrix product is associative, so a chain can be parenthesized many ways, and the scalar-multiplication cost swings wildly between them. Dynamic programming finds the cheapest parenthesization in O(n cubed) and reconstructs it.
date: 2021-03-25
draft: false
slug: /algorithms/MatrixChainMultiplication
tags:
  - Algorithms
  - Dynamic Programming
---

Multiplying a $p \times q$ matrix by a $q \times r$ matrix costs $p \cdot q \cdot r$ scalar multiplications. Matrix product is associative, so for a chain $A_1 A_2 \cdots A_n$ the answer is the same whatever order you multiply in — but the cost is not.

Take $A_1$ as $10 \times 30$, $A_2$ as $30 \times 5$, $A_3$ as $5 \times 60$:

- $(A_1 A_2) A_3$: $10\cdot30\cdot5 + 10\cdot5\cdot60 = 1500 + 3000 = 4500$.
- $A_1 (A_2 A_3)$: $30\cdot5\cdot60 + 10\cdot30\cdot60 = 9000 + 18000 = 27000$.

Six times the work for the same result. **Matrix chain multiplication** is the problem of choosing the parenthesization that minimizes scalar multiplications — not doing the multiplications, just deciding the order. The number of parenthesizations grows like the Catalan numbers, so brute force is exponential.

## The idea

Describe the chain by a dimension array `p` of length $n+1$, where matrix $A_i$ is $p_{i-1} \times p_i$. Any parenthesization of $A_i \cdots A_j$ makes one **outermost** split at some $k$ with $i \le k < j$: first form $A_i \cdots A_k$ (a $p_{i-1} \times p_k$ matrix), then $A_{k+1} \cdots A_j$ (a $p_k \times p_j$ matrix), then multiply those two at cost $p_{i-1}\, p_k\, p_j$.

Let `m[i][j]` be the minimum cost for $A_i \cdots A_j$. Trying every split:

$$m[i][j] = \begin{cases} 0 & i = j \\ \min\limits_{i \le k < j}\big(m[i][k] + m[k+1][j] + p_{i-1}\, p_k\, p_j\big) & i < j \end{cases}$$

Because a subchain's optimal cost is reused across many longer chains, fill `m` by increasing chain length so every subproblem is ready when needed.

## The code

```python
def matrix_chain_order(p):
    n = len(p) - 1                       # number of matrices
    m = [[0] * (n + 1) for _ in range(n + 1)]
    s = [[0] * (n + 1) for _ in range(n + 1)]   # split points, for reconstruction

    for length in range(2, n + 1):
        for i in range(1, n - length + 2):
            j = i + length - 1
            m[i][j] = float("inf")
            for k in range(i, j):
                cost = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j]
                if cost < m[i][j]:
                    m[i][j] = cost
                    s[i][j] = k
    return m, s


def parenthesization(s, i, j):
    if i == j:
        return f"A{i}"
    k = s[i][j]
    return f"({parenthesization(s, i, k)}{parenthesization(s, k + 1, j)})"


m, s = matrix_chain_order([10, 30, 5, 60])
assert m[1][3] == 4500
assert parenthesization(s, 1, 3) == "((A1A2)A3)"
```

The `s` table records the best split for every subchain; `parenthesization` reads it recursively to print the actual grouping.

## Cost

Three nested loops — chain length, start index, split point — each up to $n$: **$O(n^3)$ time**. The two $n \times n$ tables give **$O(n^2)$ space**. That replaces a Catalan-number search (exponential) with a cubic one.

## The takeaway

Matrix chain multiplication is the standard example of *interval* DP: the subproblem is a contiguous range, and the recurrence tries every way to split that range in two. Once you see that pattern it recurs — optimal binary search trees, polygon triangulation, and parsing all use it. It's a sibling of [0/1 knapsack](/citadel/algorithms/01Kanpsack) and [longest common subsequence](/citadel/algorithms/LongestCommonSubsequence) in method; and note this is about the *order* of multiplications, whereas [Strassen's algorithm](/citadel/algorithms/Strassen) speeds up a *single* matrix multiply.
