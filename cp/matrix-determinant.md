---
title: Matrix Determinant - Gauss and Crout in O(n^3)
description: The determinant as the product of pivots after Gaussian elimination with a sign per row swap, the exact version modulo a prime, and the fraction-free Bareiss/Crout method that stays in integers.
date: 2024-06-06
draft: false
slug: /cp/matrix-determinant
tags:
  - Competitive Programming
  - Linear Algebra
  - Determinants
---

The determinant of an $n \times n$ matrix takes $O(n!)$ by cofactor expansion and $O(n^3)$ by row reduction — the same [Gaussian elimination](/citadel/cp/gauss-linear-system) as solving a system, since row operations change the determinant predictably. Contest uses: counting spanning trees ([Kirchhoff](/citadel/cp/kirchhoff-theorem)), non-crossing lattice paths (Lindström-Gessel-Viennot), checking linear independence, and area/volume in [geometry](/citadel/cp/geometry).

## The problem

Compute $\det(A)$ for $A \in F^{n \times n}$, where $F$ is $\mathbb{R}$, $\mathbb{Z}/p\mathbb{Z}$ (prime $p$), or $\mathbb{Q}$ / $\mathbb{Z}$ when an *exact integer* answer is required.

Example: $\det\begin{pmatrix} 2 & 1 \\ 1 & 3 \end{pmatrix} = 6 - 1 = 5$.

## The idea

Row operations and the determinant:

- **Add a multiple of one row to another** — determinant unchanged.
- **Swap two rows** — determinant negated.
- **Scale a row by $c$** — determinant multiplied by $c$.

So: reduce $A$ to upper triangular using only "add a multiple" and "swap" (tracking a sign for swaps). An upper-triangular matrix's determinant is the product of its diagonal. Therefore

$$\det(A) = (-1)^{\#\text{swaps}} \cdot \prod_{i} (\text{pivot}_i).$$

If at some column no non-zero pivot exists, the matrix is singular and $\det(A) = 0$.

## How it works

$A = \begin{pmatrix} 2 & 1 \\ 1 & 3 \end{pmatrix}$. Pivot $2$ in column $0$. Eliminate row $1$: subtract $\tfrac12 \times$ row $0$, giving $\begin{pmatrix} 2 & 1 \\ 0 & 5/2 \end{pmatrix}$. No swaps. $\det = 2 \cdot \tfrac52 = 5$.

## Approach 1: Gaussian elimination (reals / mod prime)

### Algorithm

1. `sign = 1`, `det = 1`.
2. For each column $c$: find a row $\ge c$ with a non-zero entry in column $c$. If none, return $0$.
3. Swap it to row $c$; if it moved, `sign = -sign`.
4. Multiply `det` by `A[c][c]`.
5. For each row $r > c$: subtract `A[r][c] / A[c][c]` times row $c$ from row $r$.
6. Return `sign * det`.

```python
def determinant_real(a: list[list[float]]) -> float:
    a = [row[:] for row in a]
    n = len(a)
    det = 1.0
    for c in range(n):
        piv = max(range(c, n), key=lambda r: abs(a[r][c]))
        if abs(a[piv][c]) < 1e-12:
            return 0.0
        if piv != c:
            a[c], a[piv] = a[piv], a[c]
            det = -det
        det *= a[c][c]
        for r in range(c + 1, n):
            f = a[r][c] / a[c][c]
            for k in range(c, n):
                a[r][k] -= f * a[c][k]
    return det

def determinant_mod(a: list[list[int]], mod: int) -> int:
    a = [[x % mod for x in row] for row in a]
    n = len(a)
    det = 1
    for c in range(n):
        piv = next((r for r in range(c, n) if a[r][c]), -1)
        if piv == -1:
            return 0
        if piv != c:
            a[c], a[piv] = a[piv], a[c]
            det = -det % mod
        inv = pow(a[c][c], mod - 2, mod)
        det = det * a[c][c] % mod
        for r in range(c + 1, n):
            f = a[r][c] * inv % mod
            if f:
                a[r] = [(a[r][k] - f * a[c][k]) % mod for k in range(n)]
    return det % mod
```

### Complexity

$O(n^3)$ time, $O(n^2)$ space. `determinant_mod` is exact; `determinant_real` accumulates round-off — fine for small $n$ and integer inputs, risky otherwise.

## Approach 2: Bareiss / Crout — fraction-free, exact integer

### Intuition

Ordinary elimination introduces fractions. Bareiss's algorithm divides each update by the *previous* pivot, and a theorem guarantees that division is exact — every intermediate stays an integer, and the final diagonal product (divided appropriately) is $\det(A)$ exactly, with no modulus and no floating point.

### Algorithm

Maintain `prev = 1`. For each pivot step $k$:

$$a_{ij} \leftarrow \frac{a_{ij}\, a_{kk} - a_{ik}\, a_{kj}}{\text{prev}} \quad (i, j > k),$$

then `prev = a[k][k]` (before it is overwritten). After processing all rows, $\det(A) = a_{n-1,n-1}$ (times the swap sign).

```python
def determinant_bareiss(a: list[list[int]]) -> int:
    a = [row[:] for row in a]
    n = len(a)
    sign = 1
    prev = 1
    for k in range(n - 1):
        if a[k][k] == 0:
            swap = next((r for r in range(k + 1, n) if a[r][k]), -1)
            if swap == -1:
                return 0
            a[k], a[swap] = a[swap], a[k]
            sign = -sign
        for i in range(k + 1, n):
            for j in range(k + 1, n):
                a[i][j] = (a[i][j] * a[k][k] - a[i][k] * a[k][j]) // prev
        prev = a[k][k]
    return sign * a[n - 1][n - 1]
```

### Complexity

$O(n^3)$ arithmetic operations, but on integers that can grow to $\sim n$ times the bit length of the entries — Python handles the big integers transparently; in C++ you would need `__int128` or a bignum for large $n$.

## Common pitfalls

- **Sign from swaps.** Each row swap flips the sign. Track it; forgetting gives $\pm\det$.
- **Real determinant of an integer matrix.** Round-off can turn an exact $0$ into $10^{-9}$ or an exact $5$ into $4.9999997$. Use `determinant_mod` (or round if you know the answer is an integer and $n$ is tiny).
- **Bareiss division.** The `// prev` is *exact* by the theorem — if you get a non-integer, there is an indexing bug, not a math surprise.
- **Singular detection.** In `determinant_real`, a pivot below the epsilon means singular → return $0$; do not divide by it.
- **`det` under a prime and negative intermediates.** Keep reducing `% mod` and normalise the final result into $[0, p)$.

## The keystone

Row-reduce to upper triangular with "add a multiple" (determinant-preserving) and "swap" (sign-flipping); the determinant is then the pivot product times $(-1)^{\#\text{swaps}}$ — $O(n^3)$. Use the modular version for exact answers over a prime, and Bareiss when you need an exact integer determinant with no modulus, as [Kirchhoff's spanning-tree count](/citadel/cp/kirchhoff-theorem) does.
