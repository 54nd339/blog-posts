---
title: Strassen's Algorithm - A Faster Way to Multiply Matrices with Divide and Conquer
description: Discover Strassen's matrix multiplication algorithm! Learn how this clever divide and conquer technique multiplies two n x n matrices in approximately O(n^{2.81}) time, beating the standard O(n^3) method.
date: 2023-02-08
draft: false
slug: /pensieve/algorithms/strassen-matrix-multiplication
tags:
  - DSA
  - Divide and Conquer
---

Hey everyone, and welcome back to the blog! Multiplying matrices is a fundamental operation in many fields, from computer graphics and scientific computing to machine learning and data analysis. The standard "schoolbook" method for multiplying two $n \times n$ matrices involves $n^3$ scalar multiplications, giving it an O(n³) time complexity. This is perfectly fine for small matrices, but for large matrices, this cubic complexity can become a significant performance bottleneck.

Enter **Strassen's Algorithm**! Developed by Volker Strassen in 1969, this ingenious algorithm was one of the first to demonstrate that matrix multiplication could be done faster than O(n³). It uses a **divide and conquer** approach to reduce the number of required multiplications, achieving a better asymptotic complexity. Let's explore how it works its magic.

## The Problem: Speeding Up Matrix Multiplication
Given two square matrices, $A$ and $B$, both of size $n \times n$, our goal is to compute their product, $C = A \times B$, also an $n \times n$ matrix. The challenge is to do this with fewer than the $O(n^3)$ scalar multiplications required by the standard algorithm, especially when $n$ is large.

Strassen's algorithm achieves this by cleverly reducing the multiplication of two $n \times n$ matrices to **seven** multiplications of $n/2 \times n/2$ submatrices, along with a fixed number of matrix additions and subtractions (which are $O(n^2)$ operations). This reduction from eight (which the naive divide and conquer approach would use) to seven recursive multiplications is the key to its improved time complexity.

## The Strassen Algorithm: A Divide and Conquer Approach

Let's assume, for simplicity, that $n$ is a power of 2, so we can always divide matrices evenly. (If not, padding can be used, though this adds some complexity in practice).

1.  **Divide (Partitioning the Matrices):**
    Divide the input matrices $A$ and $B$ (and the result matrix $C$) into four equal-sized $n/2 \times n/2$ submatrices:

    $A = \begin{pmatrix} A_{11} & A_{12} \\ A_{21} & A_{22} \end{pmatrix}$,
    $B = \begin{pmatrix} B_{11} & B_{12} \\ B_{21} & B_{22} \end{pmatrix}$,
    $C = \begin{pmatrix} C_{11} & C_{12} \\ C_{21} & C_{22} \end{pmatrix}$

    The standard multiplication would then be:
    $C_{11} = A_{11}B_{11} + A_{12}B_{21}$
    $C_{12} = A_{11}B_{12} + A_{12}B_{22}$
    $C_{21} = A_{21}B_{11} + A_{22}B_{21}$
    $C_{22} = A_{21}B_{12} + A_{22}B_{22}$
    This involves 8 recursive multiplications of $n/2 \times n/2$ matrices.

2.  **Conquer (The 7 Clever Products - Strassen's Steps):**
    Strassen defined 10 intermediate sum/difference matrices ($S_1$ to $S_{10}$) involving only additions and subtractions of the $n/2 \times n/2$ submatrices of $A$ and $B$. Then, 7 products ($P_1$ to $P_7$) are computed recursively using these $S_i$ matrices or the original submatrices.

    Let's define the 10 sums/differences:
    * $S_1 = B_{12} - B_{22}$
    * $S_2 = A_{11} + A_{12}$
    * $S_3 = A_{21} + A_{22}$
    * $S_4 = B_{21} - B_{11}$
    * $S_5 = A_{11} + A_{22}$
    * $S_6 = B_{11} + B_{22}$
    * $S_7 = A_{12} - A_{22}$
    * $S_8 = B_{21} + B_{22}$
    * $S_9 = A_{11} - A_{21}$
    * $S_{10} = B_{11} + B_{12}$

    Now, compute the 7 products recursively:
    * $P_1 = \text{Strassen}(A_{11}, S_1)$
    * $P_2 = \text{Strassen}(S_2, B_{22})$
    * $P_3 = \text{Strassen}(S_3, B_{11})$
    * $P_4 = \text{Strassen}(A_{22}, S_4)$
    * $P_5 = \text{Strassen}(S_5, S_6)$
    * $P_6 = \text{Strassen}(S_7, S_8)$
    * $P_7 = \text{Strassen}(S_9, S_{10})$

3.  **Combine (Calculating Submatrices of C):**
    Finally, the submatrices of the result $C$ are calculated using additions and subtractions of these 7 products:
    * $C_{11} = P_5 + P_4 - P_2 + P_6$
    * $C_{12} = P_1 + P_2$
    * $C_{21} = P_3 + P_4$
    * $C_{22} = P_5 + P_1 - P_3 - P_7$

    The resulting matrix $C$ is then formed by combining $C_{11}, C_{12}, C_{21}, C_{22}$.

The base case for the recursion is when the matrix size $n$ becomes 1 (or some small threshold where standard multiplication is faster). If $n=1$, $A \times B$ is just a simple scalar multiplication.

### Pseudocode for Strassen's Algorithm

Here's a conceptual pseudocode reflecting these steps:

```pseudocode
// A, B: Input square matrices of size n x n
function strassen(A, B):
    n = number of rows in A // Assuming square matrices, A.rows == A.cols

    // Base Case: If matrices are 1x1 (or small enough for standard multiplication)
    if n == 1:
        C = new matrix of size 1x1
        C[0][0] = A[0][0] * B[0][0]
        return C
   
    // If n is not 1, proceed with partitioning (assuming n is a power of 2 for simplicity)
    // Split A and B into four n/2 x n/2 submatrices
    A11, A12, A21, A22 = split_matrix(A) // Helper function to get submatrices
    B11, B12, B21, B22 = split_matrix(B)

    // Calculate the 10 intermediate S matrices (using matrix addition/subtraction)
    S1 = B12 - B22
    S2 = A11 + A12
    S3 = A21 + A22
    S4 = B21 - B11
    S5 = A11 + A22
    S6 = B11 + B22
    S7 = A12 - A22
    S8 = B21 + B22
    S9 = A11 - A21
    S10 = B11 + B12

    // Recursively compute the 7 products P1 through P7
    P1 = strassen(A11, S1)
    P2 = strassen(S2, B22)
    P3 = strassen(S3, B11)
    P4 = strassen(A22, S4)
    P5 = strassen(S5, S6)
    P6 = strassen(S7, S8)
    P7 = strassen(S9, S10)

    // Calculate the submatrices of the result C
    C11 = P5 + P4 - P2 + P6
    C12 = P1 + P2
    C21 = P3 + P4
    C22 = P5 + P1 - P3 - P7

    // Combine C11, C12, C21, C22 into the final result matrix C
    C = combine_submatrices(C11, C12, C21, C22)
   
    return C

// Helper functions like split_matrix, matrix_add, matrix_subtract, combine_submatrices
// would need to be defined.
```

### Explanation of the Logic:

The `strassen` function is the core recursive function.
1. It first checks for the base case: if the input matrices are very small (e.g., 1x1), it performs standard scalar multiplication and returns.
2. **Divide:** If the matrices are larger, it splits matrix `A` into `A11, A12, A21, A22` and matrix `B` into `B11, B12, B21, B22`. Each of these is an $n/2 \times n/2$ matrix.
3. **Compute S matrices:** It calculates the 10 intermediate matrices $S_1$ through $S_{10}$ using matrix additions and subtractions on the submatrices from step 2. These are $O(n^2)$ operations.
4. **Conquer (Recursive Calls):** It then makes seven recursive calls to the `strassen` function to compute the products $P_1$ through $P_7$. This is the heart of the algorithm's efficiency gain over the naive 8-multiplication divide and conquer.
5. **Combine:** Finally, it computes the four submatrices $C_{11}, C_{12}, C_{21}, C_{22}$ of the result matrix $C$ using a specific set of additions and subtractions involving the $P_i$ products. These are also $O(n^2)$ operations. These submatrices are then combined to form the final $n \times n$ result matrix $C$.

## Proof of Correctness (Intuition)

The correctness of Strassen's algorithm hinges on the algebraic validity of the specific formulas used to compute $C_{11}, C_{12}, C_{21},$ and $C_{22}$ from the seven products $P_1, \ldots, P_7$. These formulas were carefully derived by Strassen to match the results of the standard matrix multiplication but using one fewer recursive matrix multiplication.

* **Base Case:** When $n=1$ (or the chosen small threshold), the algorithm correctly returns the product by direct multiplication.
* **Inductive Step:** Assuming Strassen's algorithm correctly multiplies matrices of size $n/2 \times n/2$, the formulas for combining $P_1$ through $P_7$ ensure that $C_{11}, C_{12}, C_{21}, C_{22}$ are correctly computed as if derived from the standard 8-multiplication approach. This can be verified by expanding out Strassen's formulas for $C_{ij}$ and showing they equate to the standard definitions (e.g., $C_{12} = P_1 + P_2 = A_{11}(B_{12}-B_{22}) + (A_{11}+A_{12})B_{22} = A_{11}B_{12} - A_{11}B_{22} + A_{11}B_{22} + A_{12}B_{22} = A_{11}B_{12} + A_{12}B_{22}$, which is the standard formula).

## Complexity Analysis

* **Time Complexity:**
    The recurrence relation for Strassen's algorithm is:
    $$T(n) = 7T(n/2) + O(n^2)$$
    where:
    - $7T(n/2)$: Seven recursive calls on matrices of size $n/2 \times n/2$.
    - $O(n^2)$: The cost of matrix additions and subtractions at each step (18 additions/subtractions of $n/2 \times n/2$ matrices, each taking $O((n/2)^2) = O(n^2)$ time).

    Using the Master Theorem, this recurrence relation solves to O($n^{log₂7}$), which is approximately **O($n^{2.8074}$)**. This is asymptotically faster than the standard O(n³) algorithm.

* **Space Complexity:**
The recursive calls create a call stack. The depth of this recursion is **O(log n)**. At each level of recursion, new temporary matrices are created for the $S_i$ and $P_i$ subproblems.

## Applications of Strassen's Algorithm

While Strassen's algorithm offers better asymptotic complexity, it has some practical considerations:

* The constant factor hidden in the O-notation is larger than for the naive algorithm. This means for small $n$, the naive algorithm might actually be faster. Implementations of Strassen's usually switch to standard multiplication for matrices smaller than a certain threshold (e.g., $n \lt 64$ or $n \lt 128$).
* It involves more additions and subtractions, which can be a factor.
* It can be less numerically stable than the standard algorithm for certain types of matrices.
* Implementing it with good memory management to actually achieve better performance can be complex.

Despite these, Strassen's algorithm (and its more advanced successors like Coppersmith–Winograd, though those are even more complex) is significant because:

* It proved that O(n³) was not the theoretical lower bound for matrix multiplication.
* It is **useful in practice for multiplying very large matrices** where the $n^{2.81}$ behavior eventually outpaces the $n^3$ behavior despite larger constant factors.
* It has applications in areas requiring high-performance linear algebra computations on large matrices.

## Key Takeaways

* **Strassen's Algorithm** is a divide and conquer algorithm for matrix multiplication that is asymptotically faster than the standard O(n³) method.
* It reduces the multiplication of two $n \times n$ matrices to 7 recursive multiplications of $n/2 \times n/2$ submatrices, plus a fixed number of matrix additions/subtractions.
* Its time complexity is approximately **O($n^{2.81}$)**.
* While theoretically faster, practical benefits are typically seen for large matrices due to higher constant factors and implementation complexity.

Strassen's algorithm is a landmark in algorithmic design, demonstrating how clever restructuring of a problem can lead to surprising breakthroughs in efficiency!