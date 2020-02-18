---
title: Matrices and Determinants - Operations, Types, and What Det Means
description: A matrix is a linear transformation written as a grid; its determinant is the factor by which it scales area or volume. Matrix algebra and the special types, the adjoint and inverse, determinant properties and cofactor expansion, and how rank decides whether a linear system has a solution.
date: 2020-02-18
draft: false
slug: /maths/matrices
tags:
  - Mathematics
  - Linear Algebra
---

A matrix is two things at once: a table for organising numbers, and a linear transformation frozen into a grid. Multiplying a vector by $\begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix}$ rotates it a quarter turn; the determinant of that matrix, $1$, says the rotation preserves area. The determinant of $\begin{pmatrix} 2 & 0 \\ 0 & 3 \end{pmatrix}$ is $6$ because it stretches every region to six times its area — and a determinant of $0$ means the transformation flattens space onto a line, losing a dimension, which is exactly when it cannot be undone.

This post is the mechanics: matrix algebra and the zoo of special matrices, the adjoint and the inverse, the determinant with its cofactor expansion and its ten working properties, the eigenvalue basics, and how the **rank** of a matrix decides the fate of a linear system. The vector-space theory and the numerical algorithms are in the companion post on [linear algebra](/citadel/maths/linear-algebra).

---
## Matrices and the trace

An $m \times n$ matrix $A = [a_{ij}]$ has $m$ rows, $n$ columns, entry $a_{ij}$ in row $i$, column $j$. Two matrices are **equal** when they have the same order and every corresponding entry matches. The basic shapes: **row** ($1 \times n$), **column** ($m \times 1$), **square** ($m = n$), **rectangular** ($m \neq n$).

The **trace** of a square matrix is the sum of its diagonal, $\operatorname{tr}(A) = \sum_i a_{ii}$. It is linear ($\operatorname{tr}(A + B) = \operatorname{tr}(A) + \operatorname{tr}(B)$, $\operatorname{tr}(kA) = k\operatorname{tr}(A)$), unchanged by transpose, and — the property that earns its keep — cyclic: $\operatorname{tr}(AB) = \operatorname{tr}(BA)$ even when $AB \neq BA$.

---
## Matrix algebra

**Addition** and **scalar multiplication** act entrywise (same order required), and behave like ordinary arithmetic: commutative and associative addition, a zero matrix, additive inverses, distributive scalars.

**Multiplication** $AB$ is defined when $A$ is $m \times n$ and $B$ is $n \times p$, giving an $m \times p$ result with
$$ c_{ij} = \sum_{k=1}^{n} a_{ik}\, b_{kj} $$
— row $i$ of $A$ dotted with column $j$ of $B$. It is associative and distributive, has identity $I$, but is **not commutative**: $AB$ and $BA$ usually differ, and may not both be defined. This reflects that $AB$ means "do $B$, then $A$", and order matters for transformations.

Four unary operations:

- **Transpose** $A^\top$ — swap rows and columns, $[a_{ji}]$. $(A^\top)^\top = A$, $(A + B)^\top = A^\top + B^\top$, and $(AB)^\top = B^\top A^\top$ (order reverses).
- **Conjugate** $\bar{A}$ — conjugate every entry (complex matrices).
- **Conjugate transpose** $A^\dagger = (\bar A)^\top$ — the complex generalisation of transpose; $(AB)^\dagger = B^\dagger A^\dagger$, $(kA)^\dagger = \bar{k}A^\dagger$.
- **Matrix polynomial** — for $P(x) = a_k x^k + \cdots + a_0$, $P(A) = a_k A^k + \cdots + a_1 A + a_0 I$.

---
## Special matrices

| Name | Defining condition | Meaning |
| --- | --- | --- |
| diagonal | $a_{ij} = 0$ for $i \neq j$ | scales each axis independently |
| scalar | diagonal with equal entries | uniform scaling |
| identity $I$ | scalar with entries $1$ | does nothing |
| symmetric | $A^\top = A$ | real orthogonal eigenvectors |
| skew-symmetric | $A^\top = -A$ | zero diagonal; $2\times2$ block = rotation generator |
| orthogonal | $A^\top A = I$ | rotation/reflection; columns orthonormal |
| Hermitian | $A^\dagger = A$ | real diagonal, real eigenvalues |
| skew-Hermitian | $A^\dagger = -A$ | imaginary diagonal |
| unitary | $A^\dagger A = I$ | complex analogue of orthogonal |
| idempotent | $A^2 = A$ | a projection |
| involutory | $A^2 = I$ | its own inverse (a reflection) |
| nilpotent | $A^k = O$ for some $k$ | shifts, eventually annihilates |

---
## Adjoint and inverse

The **adjoint** $\operatorname{adj}(A)$ is the transpose of the cofactor matrix: $(\operatorname{adj} A)_{ij} = C_{ji}$, where $C_{ij}$ is defined below. Its defining identity is
$$ A\,\operatorname{adj}(A) = \operatorname{adj}(A)\,A = (\det A)\,I $$
which immediately gives the **inverse**, when it exists:
$$ A^{-1} = \frac{1}{\det A}\,\operatorname{adj}(A), \qquad \text{exists} \iff \det A \neq 0 $$
A matrix with $\det A \neq 0$ is **non-singular** (invertible); with $\det A = 0$ it is **singular**. Inverse properties: $(A^{-1})^{-1} = A$, $(AB)^{-1} = B^{-1}A^{-1}$, $(A^\top)^{-1} = (A^{-1})^\top$, $\det(A^{-1}) = 1/\det A$. In practice the inverse comes not from the adjoint formula but from row-reducing $[A \mid I]$ to $[I \mid A^{-1}]$; if a zero row appears on the left, $A$ is singular.

Adjoint identities worth knowing: $\det(\operatorname{adj} A) = (\det A)^{n-1}$ and $\operatorname{adj}(\operatorname{adj} A) = (\det A)^{n-2}A$ for $n \times n$.

---
## Equivalence and elementary operations

Three **elementary operations** — swap two rows (or columns), scale a row by $k \neq 0$, add $k$ times one row to another — are all reversible and all preserve rank. Two matrices related by a sequence of them are **equivalent**, and every nonzero matrix reduces to a canonical form with a block of $1$s on the diagonal and zeros elsewhere. These operations are the engine behind computing inverses, ranks, and determinants.

---
## Geometric transformations

A point becomes a column vector, and a matrix moves it. In 2D:
$$ \text{rotate } \theta:\ \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}, \qquad \text{scale}:\ \begin{pmatrix} s_x & 0 \\ 0 & s_y \end{pmatrix}, \qquad \text{reflect in } y = x:\ \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix} $$
with reflections in the axes and origin given by the sign patterns $\operatorname{diag}(1, -1)$, $\operatorname{diag}(-1, 1)$, $\operatorname{diag}(-1, -1)$. In 3D the rotations about the coordinate axes are
$$ R_x = \begin{pmatrix} 1 & 0 & 0 \\ 0 & \cos\theta & -\sin\theta \\ 0 & \sin\theta & \cos\theta \end{pmatrix},\quad R_y = \begin{pmatrix} \cos\theta & 0 & \sin\theta \\ 0 & 1 & 0 \\ -\sin\theta & 0 & \cos\theta \end{pmatrix},\quad R_z = \begin{pmatrix} \cos\theta & -\sin\theta & 0 \\ \sin\theta & \cos\theta & 0 \\ 0 & 0 & 1 \end{pmatrix} $$
with $\operatorname{diag}(s_x, s_y, s_z)$ for scaling and a sign flip for a plane reflection. Composing transformations is multiplying their matrices; translations need a fourth homogeneous coordinate to become matrices too.

---
## Eigenvalues in brief

For square $A$, the **characteristic equation** $\det(A - \lambda I) = 0$ is a degree-$n$ polynomial whose roots are the **eigenvalues**; each eigenvalue's nonzero solutions of $(A - \lambda I)X = 0$ are its **eigenvectors** — the directions $A$ merely scales. Fast facts:

- $\sum \lambda_i = \operatorname{tr}(A)$ and $\prod \lambda_i = \det A$;
- $A$ and $A^\top$ have the same eigenvalues;
- a symmetric or Hermitian matrix has all-real eigenvalues;
- a triangular matrix's eigenvalues are its diagonal entries;
- **Cayley–Hamilton**: $A$ satisfies its own characteristic polynomial, $P(A) = O$ — which lets you write $A^{-1}$ and any high power of $A$ as a low-degree polynomial in $A$.

The full treatment — diagonalisation, the power method, QR — is in [linear algebra](/citadel/maths/linear-algebra).

---
## Determinants

$\det[a] = a$; $\det\begin{pmatrix} a & b \\ c & d \end{pmatrix} = ad - bc$; larger determinants are defined recursively. Geometrically $|\det A|$ is the factor by which $A$ scales area (2D) or volume (3D), and the sign records whether orientation is flipped.

**Ten properties**, most of them ways to simplify before computing:

1. $\det(A^\top) = \det(A)$.
2. Swapping two rows (or columns) flips the sign.
3. Two equal rows (or columns) $\Rightarrow \det = 0$.
4. A zero row (or column) $\Rightarrow \det = 0$.
5. Scaling one row by $k$ scales the determinant by $k$.
6. Hence $\det(kA) = k^n \det(A)$ for $n \times n$.
7. $\det(AB) = \det(A)\det(B)$.
8. A triangular matrix's determinant is the product of its diagonal.
9. $R_i \to R_i + kR_j$ leaves the determinant unchanged — the key move for hand computation.
10. $A$ is invertible $\iff \det(A) \neq 0$.

**Minors and cofactors.** The minor $M_{ij}$ is the determinant left after deleting row $i$ and column $j$; the cofactor is $C_{ij} = (-1)^{i+j}M_{ij}$. **Laplace (cofactor) expansion** along any row or column:
$$ \det(A) = \sum_{j=1}^{n} a_{ij}\,C_{ij} $$
For $3 \times 3$, **Sarrus's rule** is the shortcut: $\det = aei + bfg + cdh - ceg - bdi - afh$ for $\begin{pmatrix} a & b & c \\ d & e & f \\ g & h & i \end{pmatrix}$.

**Differentiating a determinant** whose entries are functions of $x$: sum $n$ determinants, in each of which one row (or column) has been differentiated and the rest left alone.

**In geometry**, the determinant packages classic conditions ([coordinate geometry](/citadel/maths/2d-geometry) has the derivations):
$$ \text{triangle area} = \tfrac12\left|\det\begin{pmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{pmatrix}\right|, \qquad \text{collinear} \iff \text{that } \det = 0 $$
and three lines $a_i x + b_i y + c_i = 0$ are concurrent (or parallel) iff $\det[a_i\ b_i\ c_i] = 0$.

---
## Systems and rank

Write a linear system as $AX = B$. It is **consistent** if it has a solution. The **rank** of $A$ — the number of linearly independent rows, equivalently columns, equivalently the size of the largest nonzero minor — decides everything. Reduce to echelon form; the rank is the count of nonzero rows.

- **Homogeneous** $AX = O$: always has $X = O$; has other solutions iff $\operatorname{rank}(A) < n$ (for square $A$, iff $\det A = 0$).
- **Non-homogeneous** $AX = B$: consistent iff $\operatorname{rank}(A) = \operatorname{rank}([A \mid B])$. Then a unique solution if that common rank equals $n$, and an $\big(n - \operatorname{rank}(A)\big)$-parameter family of solutions if it is less. If $\operatorname{rank}(A) < \operatorname{rank}([A \mid B])$, there is no solution.

**Cramer's rule** gives the unique solution when $\det A \neq 0$: $x_i = \det(A_i)/\det(A)$, with $A_i$ being $A$ with column $i$ replaced by $B$. Clean to state, too slow for large systems.

---
## The one idea to keep

The two ideas to hold onto: a matrix *is* a linear map, so composing maps is multiplying matrices and undoing one is inverting; and the determinant is the volume-scaling factor, so $\det = 0$ is precisely the collapse that makes a map non-invertible and a system either unsolvable or under-determined. From here, the same objects power [3D geometry](/citadel/maths/3d-geometry), the eigen-analysis of [systems of differential equations](/citadel/maths/differential-equations), and — with real numbers and rounding error — [computational science](/citadel/maths/numerical-analysis).
