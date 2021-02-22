---
title: Strassen's Algorithm - Seven Multiplies for a 2x2 Matrix Block
description: Multiplying two n-by-n matrices the schoolbook way costs n^3 scalar multiplications. Strassen's 1969 divide-and-conquer scheme multiplies 2-by-2 blocks with seven recursive products instead of eight, giving about O(n^2.807).
date: 2021-02-22
draft: false
slug: /algorithms/Strassen
tags:
  - Algorithms
  - Divide and Conquer
---

Multiplying two $n \times n$ matrices by the definition takes $n^3$ scalar multiplications. For decades that was assumed to be the best possible. In 1969 Volker Strassen showed it isn't: by restructuring the multiplication of $2 \times 2$ blocks, he needed **seven** recursive multiplications instead of eight, and recursion turns that saving into an exponent of about 2.807.

## The block view

Split $A$, $B$, and the result $C$ into four $n/2 \times n/2$ blocks (assume $n$ is a power of 2; pad otherwise):

$$C = \begin{pmatrix} C_{11} & C_{12} \\ C_{21} & C_{22} \end{pmatrix}, \quad \text{where } C_{11} = A_{11}B_{11} + A_{12}B_{21}, \text{ etc.}$$

The four formulas together use **eight** block multiplications. Recursing gives $T(n) = 8T(n/2) + O(n^2) = O(n^3)$ — no improvement over schoolbook.

## Seven products

Strassen defined seven products $P_1 \ldots P_7$, each one multiplication of block sums or differences:

$$
\begin{aligned}
P_1 &= A_{11}(B_{12} - B_{22}) & P_2 &= (A_{11} + A_{12})B_{22} \\
P_3 &= (A_{21} + A_{22})B_{11} & P_4 &= A_{22}(B_{21} - B_{11}) \\
P_5 &= (A_{11} + A_{22})(B_{11} + B_{22}) & P_6 &= (A_{12} - A_{22})(B_{21} + B_{22}) \\
P_7 &= (A_{11} - A_{21})(B_{11} + B_{12})
\end{aligned}
$$

Then the result blocks are additions and subtractions of those:

$$C_{11} = P_5 + P_4 - P_2 + P_6, \quad C_{12} = P_1 + P_2, \quad C_{21} = P_3 + P_4, \quad C_{22} = P_5 + P_1 - P_3 - P_7.$$

Each identity checks out by expansion — for instance $C_{12} = P_1 + P_2 = A_{11}(B_{12} - B_{22}) + (A_{11} + A_{12})B_{22} = A_{11}B_{12} + A_{12}B_{22}$, the schoolbook formula. The block additions are $O(n^2)$; the seven multiplications are the recursion.

## The code

```python
def add(X, Y): return [[a + b for a, b in zip(rx, ry)] for rx, ry in zip(X, Y)]
def sub(X, Y): return [[a - b for a, b in zip(rx, ry)] for rx, ry in zip(X, Y)]

def split(M):
    n = len(M) // 2
    a11 = [row[:n] for row in M[:n]]
    a12 = [row[n:] for row in M[:n]]
    a21 = [row[:n] for row in M[n:]]
    a22 = [row[n:] for row in M[n:]]
    return a11, a12, a21, a22

def strassen(A, B):
    n = len(A)
    if n == 1:
        return [[A[0][0] * B[0][0]]]

    a11, a12, a21, a22 = split(A)
    b11, b12, b21, b22 = split(B)

    p1 = strassen(a11, sub(b12, b22))
    p2 = strassen(add(a11, a12), b22)
    p3 = strassen(add(a21, a22), b11)
    p4 = strassen(a22, sub(b21, b11))
    p5 = strassen(add(a11, a22), add(b11, b22))
    p6 = strassen(sub(a12, a22), add(b21, b22))
    p7 = strassen(sub(a11, a21), add(b11, b12))

    c11 = add(sub(add(p5, p4), p2), p6)
    c12 = add(p1, p2)
    c21 = add(p3, p4)
    c22 = sub(sub(add(p5, p1), p3), p7)

    top = [l + r for l, r in zip(c11, c12)]
    bot = [l + r for l, r in zip(c21, c22)]
    return top + bot


A = [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12], [13, 14, 15, 16]]
B = [[17, 18, 19, 20], [21, 22, 23, 24], [25, 26, 27, 28], [29, 30, 31, 32]]
naive = [[sum(A[i][k] * B[k][j] for k in range(4)) for j in range(4)] for i in range(4)]
assert strassen(A, B) == naive
```

Real implementations stop recursing below a threshold (around $n = 64$ to $128$) and finish with schoolbook multiplication, because Strassen's constant factor is larger.

## Cost

$$T(n) = 7T(n/2) + O(n^2)$$

— seven recursive multiplications, plus 18 block additions and subtractions costing $O(n^2)$ total. The master theorem gives $O(n^{\log_2 7}) \approx O(n^{2.807})$. Recursion depth is $O(\log n)$, but each level allocates temporary blocks.

## Why it matters, and its limits

Strassen's headline result was conceptual: $O(n^3)$ is *not* a lower bound for matrix multiplication. Later algorithms (Coppersmith–Winograd and its refinements) push the exponent below 2.373, though they are galactic — only faster for matrices too large to exist. Strassen itself is used in practice for large dense matrices once $n$ is past the crossover, accepting a larger constant, more additions, and slightly worse numerical stability. Note the contrast with [matrix chain multiplication](/citadel/algorithms/MatrixChainMultiplication), which optimises the *order* of a sequence of multiplies rather than speeding up one; and with [Karatsuba](/citadel/algorithms/Karatsuba), which is the same "trade a multiply for some adds" trick on integers.
