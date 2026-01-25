---
title: Matrix Chain Multiplication - Finding the Most Efficient Multiplication Order with Dynamic Programming
description: Multiplying a chain of matrices? The order matters! We explore the Matrix Chain Multiplication problem and how dynamic programming can find the optimal parenthesization to minimize scalar multiplications.
date: 2023-02-20
draft: false
slug: /pensieve/algorithms/matrix-chain-multiplication
tags:
  - DSA
  - DP
---

Hey everyone, and welcome back to the blog! When we think about multiplying matrices, it might seem straightforward. However, when you have a *sequence* or a *chain* of matrices to multiply, the order in which you perform these multiplications can drastically affect the total number of simple arithmetic operations (scalar multiplications) involved. This is where the **Matrix Chain Multiplication (MCM)** problem comes into play.

The problem isn't about actually performing the matrix multiplications themselves, but rather about **deciding the most efficient parenthesization** (the order of multiplications) to minimize the total number of scalar multiplications needed. It's a classic optimization problem that can be elegantly solved using **Dynamic Programming**. Let's dive in!

## The Problem: Order Matters in Multiplication!

Given a sequence of $n$ matrices $A_1, A_2, \ldots, A_n$, we want to compute their product:
$P = A_1 \times A_2 \times \ldots \times A_n$

Matrix multiplication is associative, meaning $(A_1 \times A_2) \times A_3 = A_1 \times (A_2 \times A_3)$. This means we can place parentheses in different ways to group the multiplications, but the final matrix product will be the same. However, the number of scalar (elementary) multiplications can vary dramatically depending on the parenthesization.

**Why does the order matter?**
Recall that to multiply a matrix $A$ of dimensions $p \times q$ with a matrix $B$ of dimensions $q \times r$, the resulting matrix $C$ will have dimensions $p \times r$, and the number of scalar multiplications required is $p \times q \times r$.

Consider three matrices:
* $A_1$: $10 \times 30$
* $A_2$: $30 \times 5$
* $A_3$: $5 \times 60$

We can multiply them in two ways:
1.  **$(A_1 \times A_2) \times A_3$**:
    * $A_1 \times A_2$: $(10 \times 30) \times (30 \times 5)$. Result is $10 \times 5$. Scalar multiplications = $10 \times 30 \times 5 = 1500$.
    * $(A_1A_2) \times A_3$: $(10 \times 5) \times (5 \times 60)$. Result is $10 \times 60$. Scalar multiplications = $10 \times 5 \times 60 = 3000$.
    * Total scalar multiplications = $1500 + 3000 = 4500$.

2.  **$A_1 \times (A_2 \times A_3)$**:
    * $A_2 \times A_3$: $(30 \times 5) \times (5 \times 60)$. Result is $30 \times 60$. Scalar multiplications = $30 \times 5 \times 60 = 9000$.
    * $A_1 \times (A_2A_3)$: $(10 \times 30) \times (30 \times 60)$. Result is $10 \times 60$. Scalar multiplications = $10 \times 30 \times 60 = 18000$.
    * Total scalar multiplications = $9000 + 18000 = 27000$.

Clearly, the first order (4500 operations) is much more efficient than the second (27000 operations)! The Matrix Chain Multiplication problem is to find this optimal parenthesization.

## Solving with Dynamic Programming: A Bottom-Up Approach

The MCM problem has optimal substructure (an optimal solution to the problem contains optimal solutions to subproblems) and overlapping subproblems (the same subproblems are solved multiple times in a naive recursive approach). This makes it ideal for Dynamic Programming.

We want to find the minimum number of scalar multiplications needed to multiply a chain of matrices $A_i \ldots A_j$. Let $m[i, j]$ be this minimum number. If we split this chain at matrix $A_k$ (where $i \le k < j$), we multiply $A_i \ldots A_k$ and $A_{k+1} \ldots A_j$ separately, and then multiply the two resulting matrices. The cost would be:

$$m[i, k] + m[k+1, j] + (\text{cost of multiplying the two resultant matrices})$$

The dimensions of the matrices are typically given in an array `p`. If we have $n$ matrices $A_1, A_2, \ldots, A_n$, where matrix $A_i$ has dimensions $p_{i-1} \times p_i$, then the array `p` will have length $n+1$. The cost of multiplying the result of $A_i \ldots A_k$ (which has dimensions $p_{i-1} \times p_k$) with the result of $A_{k+1} \ldots A_j$ (which has dimensions $p_k \times p_j$) is $p_{i-1} \times p_k \times p_j$.

So, the recurrence relation is:
$m[i, j] = 0$ if $i = j$
$m[i, j] = \min_{i \le k < j} \{ m[i, k] + m[k+1, j] + p_{i-1} \cdot p_k \cdot p_j \}$ if $i < j$

### Pseudocode for Matrix Chain Order

The following pseudocode implements this bottom-up dynamic programming approach. It takes an array `p` of dimensions and `n` (the number of matrices) as input. `p[0...n]` means matrix $A_i$ has dimensions $p[i-1] \times p[i]$.

```pseudocode
// p: Array of dimensions. p[0] is rows of A1, p[1] is cols of A1 (and rows of A2), ..., p[n] is cols of An.
// n: Number of matrices in the chain (A1, A2, ..., An).
// Length of p is n+1.

function MatrixChainOrder(p, n):
    // m[i, j] will store the minimum number of scalar multiplications needed
    // to compute the product of matrices A_i through A_j.
    // Dimensions are 1-indexed for matrices A_1 to A_n.
    m = array of size (n+1) x (n+1) // Or n x n if using 0-indexed matrices

    // s[i, j] will store the index k where the optimal split occurs
    // for multiplying A_i...A_j.
    s = array of size (n+1) x (n+1) // Or n x n

    // Cost is 0 for multiplying a single matrix (chain length of 1).
    for i = 1 to n:
        m[i, i] = 0

    // l is the chain length, from 2 up to n.
    for l = 2 to n:
        // i is the starting index of the chain.
        for i = 1 to (n - l + 1):
            // j is the ending index of the chain.
            j = i + l - 1
            m[i, j] = infinity // Initialize with a very large value

            // k is the split point. We try all possible splits.
            // (A_i ... A_k) * (A_{k+1} ... A_j)
            for k = i to j - 1:
                // Cost = cost_to_multiply(A_i..A_k) +
                //        cost_to_multiply(A_{k+1}..A_j) +
                //        cost_to_multiply_the_two_resulting_matrices.
                // Dimensions:
                //   Result of (A_i..A_k) is p[i-1] x p[k]
                //   Result of (A_{k+1}..A_j) is p[k] x p[j]
                q = m[i, k] + m[k + 1, j] + p[i - 1] * p[k] * p[j]
               
                if q < m[i, j]:
                    m[i, j] = q
                    s[i, j] = k // Store k as the optimal split point for A_i...A_j
   
    // m[1, n] contains the minimum number of multiplications for the whole chain A_1...A_n.
    // s contains the split points to reconstruct the optimal parenthesization.
    return m and s
```

### Explanation of the Algorithm:

1. **Initialization:**
    * **p:** An array representing the dimensions of the matrices. If you have $n$ matrices $A_1, A_2, \ldots, A_n$, and matrix $A_i$ has dimensions $dim_i \times dim_{i+1}$, then the array p would be $[dim_1, dim_2, \ldots, dim_{n+1}]$. So p[i-1] * p[i] gives the dimensions of matrix $A_i$. (The provided pseudocode uses p[i-1] * p[k] * p[j] where p[i-1] is rows of first matrix in $A_i..A_k$, p[k] is cols of last matrix in $A_i..A_k$ AND rows of first matrix in $A_{k+1}..A_j$, and p[j] is cols of last matrix in $A_{k+1}..A_j$).
    * **m[i][i] = 0 for all $i$:** The cost of multiplying a chain of length 1 (a single matrix) is 0, as no multiplications are needed.
    * **s[i][j]:** This table is used to store the value of $k$ (the split point) that yields the optimal cost for multiplying matrices $A_i$ through $A_j$. This table helps in reconstructing the actual order of multiplications.

2. **Iterating by Chain Length (`l`):** The algorithm computes $m[i, j]$ for chains of increasing length $l$, starting from $l=2$ (e.g., $A_1A_2$) up to $l=n$ (the entire chain $A_1 \ldots A_n$).

3. **Iterating by Starting Matrix (`i`):** For each chain length l, it iterates through all possible starting matrices `i`. The ending matrix `j` is then `i + l - 1`.

4. **Finding the Optimal Split (`k`):**
    * For each subchain $A_i \ldots A_j$, it tries every possible split point $k$ (where $i \le k < j$).
    * A split at $k$ means we first compute $(A_i \ldots A_k)$ and $(A_{k+1} \ldots A_j)$ separately, and then multiply these two resulting matrices.
    * The cost q for a given split $k$ is calculated as: cost($A_i$..$A_k$) (which is `m[i, k]`) + cost($A_{k+1}$..$A_j$) (which is `m[k+1, j]`) + cost of multiplying the two results (dimensions are $p[i-1] \times p[k]$ and $p[k] \times p[j]$, so cost is $p[i-1] \cdot p[k] \cdot p[j]$).
    * The algorithm finds the `k` that minimizes this `q` value and stores this minimum cost in `m[i, j]` and the split point `k` in `s[i, j]`.

5. **Result:** The minimum number of scalar multiplications for the entire chain $A_1 \ldots A_n$ is found in $m[1, n]$. The `s` table can be used to reconstruct the optimal parenthesization (the actual order of multiplications).

## Complexity Analysis

* **Time Complexity:** The algorithm uses three nested loops:
    * The outermost loop for chain length `l` runs $n-1$ times (from 2 to $n$).
    * The middle loop for starting index `i` runs approximately $n-l$ times.
    * The innermost loop for split point `k` runs approximately $l-1$ times. This gives a time complexity of roughly $O(n \cdot n \cdot n) = \textbf{O(n³)}$, where $n$ is the number of matrices in the chain.

* **Space Complexity:** The algorithm uses two 2D tables, `m` and `s`, both of size approximately $n \times n$. Therefore, the space complexity is **O(n²)**.

## Proof of Correctness (Intuition)

The correctness of the Matrix Chain Order algorithm relies on the principle of optimality inherent in dynamic programming:

1. **Optimal Substructure:** An optimal parenthesization of the product $A_i \ldots A_j$ must involve splitting this product at some $k$ into $(A_i \ldots A_k) \times (A_{k+1} \ldots A_j)$. The way these two sub-chains $(A_i \ldots A_k)$ and $(A_{k+1} \ldots A_j)$ are themselves parenthesized must also be optimal. If they weren't, we could substitute a better parenthesization for a sub-chain and get a better overall solution, contradicting the optimality of the original parenthesization.
2. **Overlapping Subproblems:** When computing the optimal cost for a chain, the optimal costs for its sub-chains are needed multiple times. The dynamic programming approach calculates the cost for each sub-chain only once and stores it in the m table, reusing it whenever needed.

The algorithm systematically builds up solutions for longer chains by optimally combining solutions for shorter, already computed sub-chains, ensuring that by the time it computes $m[1, n]$, it has considered all possible optimal parenthesizations.

## Key Takeaways

* The **Matrix Chain Multiplication** problem is about finding the optimal order (parenthesization) to multiply a sequence of matrices to minimize the total number of scalar multiplications.
* It's a classic optimization problem solvable efficiently using **Dynamic Programming**.
* The DP approach builds a solution bottom-up, calculating the minimum costs for progressively longer sub-chains of matrices.
* The time complexity is **O(n³)** and space complexity is **O(n²)**, where $n$ is the number of matrices.

While the problem might seem academic, it's a great illustration of how dynamic programming can transform an exponential brute-force search into a polynomial-time solution by exploiting optimal substructure and overlapping subproblems!