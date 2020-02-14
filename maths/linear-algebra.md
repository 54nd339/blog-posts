---
title: Linear Algebra - Vector Spaces, Systems, and Eigenvalues
description: Linear algebra is the study of maps that respect addition and scaling, and the matrices that encode them. Vector spaces and bases, Gaussian elimination and least squares, the eigenvalue problem, diagonalisation, and the numerical methods that make it all run at scale.
date: 2020-02-14
draft: false
slug: /maths/linear-algebra
tags:
  - Mathematics
  - Linear Algebra
---

A transformation that maps grid lines to grid lines and leaves the origin fixed — a rotation, a shear, a projection — is completely determined by where it sends a handful of basis vectors. Stack those images as columns and you have a matrix; applying the transformation is then just multiplication. That correspondence, between geometric maps and rectangular arrays of numbers, is what makes linear algebra both abstract and utterly computational.

This post runs both tracks. The structure: vector spaces, linear independence, bases, inner products, and linear maps. The computation: Gaussian elimination and LU for solving $Ax = b$, least squares for when there is no exact solution, the eigenvalue problem and diagonalisation, and the numerical methods (power iteration, QR) that scale these to large matrices.

---
## Vector spaces

A **vector space** $V$ over a field $F$ (usually $\mathbb{R}$) is a set with addition and scalar multiplication obeying ten axioms: closure under both operations, commutativity and associativity of addition, a zero vector, additive inverses, the two distributive laws $c(\mathbf{u} + \mathbf{v}) = c\mathbf{u} + c\mathbf{v}$ and $(c + d)\mathbf{u} = c\mathbf{u} + d\mathbf{u}$, associativity $c(d\mathbf{u}) = (cd)\mathbf{u}$, and the identity $1\mathbf{u} = \mathbf{u}$. The model is $\mathbb{R}^n$ with componentwise operations, but polynomials, matrices, and function spaces all qualify.

- A set $\{\mathbf{v}_1, \ldots, \mathbf{v}_k\}$ is **linearly independent** if $c_1\mathbf{v}_1 + \cdots + c_k\mathbf{v}_k = \mathbf{0}$ forces every $c_i = 0$ — no vector is a combination of the others.
- A **basis** is a linearly independent set that spans $V$.
- The **dimension** is the number of vectors in any basis (all bases have the same size).

An **inner product** $\langle \mathbf{u}, \mathbf{v}\rangle$ generalises the dot product and brings geometry: length $\|\mathbf{v}\| = \sqrt{\langle\mathbf{v}, \mathbf{v}\rangle}$, angle via $\cos\theta = \dfrac{\langle\mathbf{u}, \mathbf{v}\rangle}{\|\mathbf{u}\|\,\|\mathbf{v}\|}$, and **orthogonality** when the inner product is zero.

A **linear transformation** $T: V \to W$ satisfies $T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v})$ and $T(c\mathbf{u}) = cT(\mathbf{u})$ — the two conditions that make "maps lines to lines, fixes the origin" precise.

---
## Matrices as maps

Every linear $T: \mathbb{R}^n \to \mathbb{R}^m$ is $T(\mathbf{x}) = A\mathbf{x}$ for an $m \times n$ matrix $A$ whose columns are the images $T(\mathbf{e}_j)$ of the standard basis vectors. Then matrix **addition** is addition of maps, and matrix **multiplication** is **composition**: if $A$ does $T_A$ and $B$ does $T_B$, the product $BA$ does $T_B \circ T_A$ (apply $A$ first).

Two families matter geometrically: an **orthogonal** matrix ($A^{-1} = A^\top$) is an isometry — a rotation or reflection, distances and angles preserved; a **symmetric** matrix ($A^\top = A$) has a full set of mutually orthogonal eigenvectors, its principal axes.

The **rank** of $A$ is the dimension of its column space (equal to that of its row space) — the dimension of the image of $T$. And **norms** measure size: for vectors, $\|\mathbf{x}\|_1 = \sum|x_i|$, $\|\mathbf{x}\|_2 = \sqrt{\sum x_i^2}$, $\|\mathbf{x}\|_\infty = \max_i|x_i|$; matrix norms are the compatible versions used to reason about convergence and sensitivity.

---
## Solving $Ax = b$

**Gaussian elimination** reduces the augmented matrix $[A \mid b]$ to upper-triangular (row echelon) form by elementary row operations, then back-substitutes.

Solve $x + y + 2z = 9$, $2x + 4y - 3z = 1$, $3x + 6y - 5z = 0$:
$$ \left[\begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 2 & 4 & -3 & 1 \\ 3 & 6 & -5 & 0 \end{array}\right] \xrightarrow[R_3 - 3R_1]{R_2 - 2R_1} \left[\begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 0 & 2 & -7 & -17 \\ 0 & 3 & -11 & -27 \end{array}\right] \xrightarrow{R_3 - \frac32 R_2} \left[\begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 0 & 2 & -7 & -17 \\ 0 & 0 & -\frac12 & -\frac32 \end{array}\right] $$
Back-substitute: $z = 3$, then $2y - 21 = -17$ so $y = 2$, then $x + 2 + 6 = 9$ so $x = 1$. The solution is $(1, 2, 3)$. The zeros accumulating below the diagonal are the point — once the matrix is this upper-triangular "staircase" of leading entries (**pivots**), the system unwinds one variable at a time from the bottom.

**LU factorisation** writes $A = LU$ ($L$ lower-triangular, $U$ upper), a byproduct of the elimination. Then $Ax = b$ is two triangular solves — $Ly = b$ forward, $Ux = y$ backward — which pays off when solving for many right-hand sides with the same $A$.

**Existence and uniqueness** (Rouché–Capelli) turn on ranks: with $n$ unknowns, $\operatorname{rank}(A) = \operatorname{rank}([A \mid b]) = n$ gives a unique solution; equal ranks below $n$ give infinitely many; $\operatorname{rank}(A) < \operatorname{rank}([A \mid b])$ means no solution. A system is **ill-conditioned** when tiny changes in $A$ or $b$ swing the solution wildly; the **condition number** $\kappa(A) = \|A\|\,\|A^{-1}\|$ measures it, large meaning fragile.

Related tools: the **inverse** $A^{-1}$ (exists iff $\det A \neq 0$) via reducing $[A \mid I]$ to $[I \mid A^{-1}]$; **Cramer's rule** $x_i = \det(A_i)/\det(A)$, where $A_i$ is $A$ with column $i$ replaced by $b$ (elegant, slow); and **iterative methods** — Jacobi (update every variable from the previous iterate) and Gauss–Seidel (reuse new values immediately, usually faster) — for large sparse systems where elimination is too costly.

**Least squares.** When $Ax = b$ is overdetermined and inconsistent, find $\hat{x}$ minimising $\|b - A\hat{x}\|^2$. The residual must be orthogonal to the column space of $A$, i.e. $A^\top(b - A\hat{x}) = 0$, which gives the **normal equations**
$$ A^\top A\,\hat{x} = A^\top b \quad\Longrightarrow\quad \hat{x} = (A^\top A)^{-1} A^\top b $$
when $A^\top A$ is invertible. This is linear regression.

---
## Eigenvalues and diagonalisation

An **eigenvector** of a square $A$ is a nonzero $x$ whose direction $A$ leaves alone, only scaling it by the **eigenvalue** $\lambda$:
$$ Ax = \lambda x \;\Longleftrightarrow\; (A - \lambda I)x = 0 $$
A nonzero $x$ exists only if $A - \lambda I$ is singular, so the eigenvalues are the roots of the **characteristic equation** $\det(A - \lambda I) = 0$.

For $A = \begin{pmatrix} 4 & -2 \\ 1 & 1 \end{pmatrix}$: $\det(A - \lambda I) = (4 - \lambda)(1 - \lambda) + 2 = \lambda^2 - 5\lambda + 6 = (\lambda - 2)(\lambda - 3)$, so $\lambda = 2, 3$. Solving $(A - 2I)x = 0$ gives $x_1 = x_2$, eigenvector $\binom{1}{1}$; $(A - 3I)x = 0$ gives $x_1 = 2x_2$, eigenvector $\binom{2}{1}$.

If an $n \times n$ matrix has $n$ independent eigenvectors, it is **diagonalisable**: put the eigenvectors in the columns of $P$ and the eigenvalues on the diagonal of $D$, and
$$ A = PDP^{-1} $$
For the example, $P = \begin{pmatrix} 1 & 2 \\ 1 & 1 \end{pmatrix}$, $D = \begin{pmatrix} 2 & 0 \\ 0 & 3 \end{pmatrix}$. The payoff: powers are trivial, $A^k = PD^kP^{-1}$, and any function of $A$ reduces to that function of the eigenvalues.

A **quadratic form** $Q(\mathbf{x}) = \mathbf{x}^\top A\mathbf{x}$ (with $A$ symmetric) diagonalises through an orthogonal $P$ — a rotation of axes — into $\lambda_1 y_1^2 + \lambda_2 y_2^2 + \cdots$ with no cross terms. Those axes are the principal axes of the conic or quadric the form defines.

---
## Numerical eigenvalue methods

Root-finding on a degree-$n$ characteristic polynomial is unstable, so real computation avoids it.

- **Power method** — iterate $\mathbf{x}_{k+1} = A\mathbf{x}_k / \|A\mathbf{x}_k\|$; $\mathbf{x}_k$ converges to the eigenvector of the largest-magnitude eigenvalue, and the Rayleigh quotient $\dfrac{\mathbf{x}_k^\top A\mathbf{x}_k}{\mathbf{x}_k^\top\mathbf{x}_k}$ converges to the eigenvalue.
- **Gerschgorin's theorem** — every eigenvalue lies in some disk centred at a diagonal entry $a_{ii}$ with radius $\sum_{j \neq i}|a_{ij}|$; a cheap bound on where the spectrum is.
- **QR algorithm** — factor $A = QR$ ($Q$ orthogonal, $R$ upper-triangular), form $A' = RQ$, repeat; the iterates converge to a triangular matrix whose diagonal holds all the eigenvalues. For symmetric $A$, a Householder **tridiagonalisation** first makes each step much cheaper.

---
## Complex spaces

With complex scalars, the transpose becomes the conjugate transpose $A^\dagger$. A **Hermitian** matrix ($A^\dagger = A$) is the analogue of symmetric and has *real* eigenvalues — which is exactly why quantum observables are Hermitian operators. A **unitary** matrix ($A^{-1} = A^\dagger$) is the analogue of orthogonal and preserves the complex inner product. The **Hermitian form** $\mathbf{x}^\dagger A\mathbf{x}$ is always real.

---
## The one idea to keep

The two halves stay locked together: an abstract statement ("this map has a one-dimensional invariant subspace") is a concrete computation ("this matrix has a repeated eigenvalue"), and vice versa. Matrices and determinants as *objects* — their algebra, inverses, and properties — get their own treatment in [matrices and determinants](/citadel/maths/matrices); the geometry of the underlying arrows is in [vectors](/citadel/maths/vectors); and the finite-precision reality of running these algorithms is [computational science](/citadel/maths/numerical-analysis).
