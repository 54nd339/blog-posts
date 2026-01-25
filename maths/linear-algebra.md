---
title: An Introduction to Linear Algebra - From Vector Spaces to Advanced Algorithms
description: Delve into the foundational concepts of linear algebra, the branch of mathematics that studies vector spaces, linear transformations, and the systems of linear equations that define them. This post provides a detailed exploration with examples and derivations, covering vector space axioms, diagonalization, numerical methods like LU and QR factorization, and the power of matrices and determinants.
date: 2023-01-02
draft: false
slug: /pensieve/maths/linear-algebra
tags:
  - Mathematics
  - Algebra
---

Linear algebra is a cornerstone of modern mathematics, providing the essential language and tools for understanding systems of linear equations, geometric transformations, and the abstract structures known as vector spaces. Its principles are indispensable in nearly every quantitative field, including physics, engineering, computer science, data analysis, and economics. This exploration provides a comprehensive overview of the core concepts of linear algebra, beginning with its foundational abstract structures and connecting them to the concrete computational tools of matrices and determinants, complete with detailed examples and derivations.

---
## The Foundation - Vector Spaces and Linear Transformations

At its heart, linear algebra is the study of vector spaces and the functions that map between them while preserving their linear structure.

### Vector Spaces
A **vector space** is the fundamental setting for linear algebra. It is a set $V$ of objects called vectors, for which two operations—vector addition and scalar multiplication—are defined. These operations must satisfy ten specific axioms to ensure the space behaves in a consistent and structured manner. For a set $V$ to be a vector space over a field of scalars $F$ (typically the real numbers $\mathbb{R}$), for all vectors $\mathbf{u}, \mathbf{v}, \mathbf{w} \in V$ and all scalars $c, d \in F$:
1.  $\mathbf{u} + \mathbf{v} \in V$ (Closure under addition)
2.  $\mathbf{u} + \mathbf{v} = \mathbf{v} + \mathbf{u}$ (Commutativity of addition)
3.  $(\mathbf{u} + \mathbf{v}) + \mathbf{w} = \mathbf{u} + (\mathbf{v} + \mathbf{w})$ (Associativity of addition)
4.  There exists a zero vector $\mathbf{0} \in V$ such that $\mathbf{u} + \mathbf{0} = \mathbf{u}$. (Additive identity)
5.  For every $\mathbf{u} \in V$, there exists an additive inverse $-\mathbf{u} \in V$ such that $\mathbf{u} + (-\mathbf{u}) = \mathbf{0}$.
6.  $c\mathbf{u} \in V$ (Closure under scalar multiplication)
7.  $c(\mathbf{u} + \mathbf{v}) = c\mathbf{u} + c\mathbf{v}$ (Distributivity)
8.  $(c+d)\mathbf{u} = c\mathbf{u} + d\mathbf{u}$ (Distributivity)
9.  $c(d\mathbf{u}) = (cd)\mathbf{u}$ (Associativity of scalar multiplication)
10. $1\mathbf{u} = \mathbf{u}$ (Scalar identity)

A primary example is $\mathbb{R}^n$, the set of all $n$-tuples of real numbers, with standard component-wise addition and scalar multiplication. For instance, in $\mathbb{R}^2$, for $\mathbf{u}=(u_1, u_2)$ and $\mathbf{v}=(v_1, v_2)$, addition is $\mathbf{u}+\mathbf{v}=(u_1+v_1, u_2+v_2)$, and scalar multiplication is $c\mathbf{u}=(cu_1, cu_2)$. These operations can be shown to satisfy all ten axioms.

### Linear Independence, Basis, and Dimension
* **Linear Independence:** A set of vectors $\{\vec{v_1}, \dots, \vec{v_k}\}$ is **linearly independent** if the only solution to the equation $c_1\vec{v_1} + \dots + c_k\vec{v_k} = \vec{0}$ is the trivial solution $c_1=c_2=\dots=c_k=0$. This means no vector in the set can be expressed as a linear combination of the others.
* **Basis:** A **basis** for a vector space is a set of linearly independent vectors that spans the entire space.
* **Dimension:** The number of vectors in any basis for a vector space is its **dimension**.

### Inner Product Spaces
An **inner product space** is a vector space equipped with an **inner product**, an operation that generalizes the dot product. It associates a scalar, denoted $\langle \mathbf{u}, \mathbf{v} \rangle$, with every pair of vectors $\mathbf{u}, \mathbf{v}$. This allows the introduction of geometric notions like:
* **Norm (Length):** $||\mathbf{v}|| = \sqrt{\langle \mathbf{v}, \mathbf{v} \rangle}$
* **Angle:** The angle $\theta$ between two vectors is defined by $\cos\theta = \dfrac{\langle \mathbf{u}, \mathbf{v} \rangle}{||\mathbf{u}|| ||\mathbf{v}||}$.
* **Orthogonality:** Two vectors are orthogonal if their inner product is zero.

### Linear Transformations
A **linear transformation** is a function $T$ between two vector spaces, $T: V \to W$, that preserves the operations of vector addition and scalar multiplication. For any vectors $\mathbf{u}, \mathbf{v} \in V$ and any scalar $c$:
1.  $T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v})$
2.  $T(c\mathbf{u}) = cT(\mathbf{u})$
Linear transformations are the structure-preserving maps of linear algebra; they map lines to lines and keep the origin fixed.

---
## The Concrete Representation - Matrices

Matrices provide a concrete way to represent and work with linear transformations between finite-dimensional vector spaces.

### Matrices as Representations of Linear Maps
Any linear transformation $T: \mathbb{R}^n \to \mathbb{R}^m$ can be represented by an $m \times n$ matrix $A$. The action of the transformation on a vector $\mathbf{x}$ is given by matrix multiplication: $T(\mathbf{x}) = A\mathbf{x}$. The columns of the matrix $A$ are the images of the standard basis vectors of $\mathbb{R}^n$ under the transformation $T$.

### Matrix Algebra as Algebra of Transformations
* **Matrix Addition** corresponds to the addition of two linear transformations.
* **Matrix Multiplication** corresponds to the **composition** of two linear transformations. If $T_A: U \to V$ and $T_B: V \to W$ are represented by matrices $A$ and $B$, then the composite transformation $T_B \circ T_A$ is represented by the matrix product $BA$.

### Special Matrices and Transformations
* **Orthogonal Matrices ($A^{-1} = A^T$):** Represent **isometries**, transformations that preserve distances and angles, such as rotations and reflections.
* **Symmetric Matrices ($A^T=A$):** Represent transformations that have a special set of orthogonal principal axes (their eigenvectors).

### Rank of a Matrix
The **rank** of a matrix $A$ is the dimension of its column space (the span of its column vectors), which is equal to the dimension of its row space. In the context of a linear transformation $T(\mathbf{x})=A\mathbf{x}$, the rank is the dimension of the **image** or **range** of the transformation.

### Measuring Size: Vector and Matrix Norms
A **norm** is a function that assigns a strictly positive length or size to each vector in a vector space, except for the zero vector. For a vector $\mathbf{x}$:
* **$L_1$ Norm:** $||\mathbf{x}||_1 = \sum_i |x_i|$.
* **$L_2$ Norm (Euclidean Norm):** $||\mathbf{x}||_2 = \sqrt{\sum_i |x_i|^2}$.
* **$L_\infty$ Norm (Max Norm):** $||\mathbf{x}||_\infty = \max_i |x_i|$.
A **matrix norm** is a function that assigns a size to a matrix, compatible with the vector norms. Norms are essential for analyzing the convergence of iterative methods and the sensitivity of linear systems.

---
## Solving Linear Systems - The Core Problem

A central problem in linear algebra is solving a system of linear equations, written as $AX=B$. This is equivalent to asking: "What vector $X$, when acted upon by the linear transformation $A$, produces the vector $B$?"

### Gaussian Elimination
**Gaussian elimination** is the primary algorithm for solving $AX=B$. It uses elementary row operations to reduce the **augmented matrix** $[A|B]$ to **row echelon form**, an upper-triangular form. The solution is then found by **back-substitution**.

**Example:** Solve the system $x+y+2z=9$, $2x+4y-3z=1$, $3x+6y-5z=0$.
1.  **Augmented Matrix:**
    $$ \left[ \begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 2 & 4 & -3 & 1 \\ 3 & 6 & -5 & 0 \end{array} \right] $$
2.  **Row Operations to Echelon Form:**
    * $R_2 \to R_2 - 2R_1$
    * $R_3 \to R_3 - 3R_1$
    $$ \left[ \begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 0 & 2 & -7 & -17 \\ 0 & 3 & -11 & -27 \end{array} \right] $$
    * $R_3 \to R_3 - \dfrac{3}{2}R_2$
    $$ \left[ \begin{array}{ccc|c} 1 & 1 & 2 & 9 \\ 0 & 2 & -7 & -17 \\ 0 & 0 & -1/2 & -3/2 \end{array} \right] $$
3.  **Back-Substitution:**
    * From $R_3$: $-\dfrac{1}{2}z = -\dfrac{3}{2} \implies z=3$.
    * From $R_2$: $2y - 7z = -17 \implies 2y - 7(3) = -17 \implies 2y = 4 \implies y=2$.
    * From $R_1$: $x + y + 2z = 9 \implies x + 2 + 2(3) = 9 \implies x = 1$.
    The unique solution is $(x,y,z) = (1,2,3)$.

### LU-Factorization
For a square matrix $A$, an **LU-factorization** is a decomposition of $A$ into the product of a lower triangular matrix $L$ and an upper triangular matrix $U$:
$$ A = LU $$
* **Solving Systems with LU-Factorization:** Once $L$ and $U$ are known, solving $AX=B$ becomes a two-step process:
    1.  Solve $L\mathbf{y}=B$ for $\mathbf{y}$ using **forward substitution**.
    2.  Solve $U\mathbf{x}=\mathbf{y}$ for $\mathbf{x}$ using **back substitution**.
This is highly efficient when solving for multiple different $B$ vectors with the same matrix $A$. The factorization can often be found as a byproduct of the Gaussian elimination process.

### Existence, Uniqueness and Stability of Solutions
The nature of the solution is determined by the ranks of the coefficient matrix $A$ and the augmented matrix $[A|B]$ (Rouché–Capelli theorem):
1.  **Unique Solution:** If $rank(A) = rank([A|B]) = n$ (where $n$ is the number of variables).
2.  **Infinitely Many Solutions:** If $rank(A) = rank([A|B]) < n$.
3.  **No Solution (Inconsistent):** If $rank(A) < rank([A|B])$.

* **Ill-Conditioning and Norms:** A system is **ill-conditioned** if small changes in the coefficient matrix $A$ or the vector $B$ can lead to large changes in the solution vector $X$. The **condition number** of a matrix, $\kappa(A) = ||A|| \cdot ||A^{-1}||$, quantifies this sensitivity. A large condition number signifies an ill-conditioned system.

### The Inverse of a Matrix
For a square matrix $A$, its inverse $A^{-1}$ satisfies $AA^{-1}=A^{-1}A=I$. An inverse exists if and only if $A$ is non-singular ($\det(A) \neq 0$).

* **Gauss-Jordan Elimination for Inverse:** This algorithm finds the inverse by reducing $[A|I]$ to $[I|A^{-1}]$ via elementary row operations.

### Determinants and Cramer’s Rule
The **determinant** of a square matrix, $\det(A)$, is a scalar that determines if the matrix is invertible. **Cramer's Rule** provides a formula for the solution of $AX=B$ when $\det(A) \neq 0$:
$$ x_i = \dfrac{\det(A_i)}{\det(A)} $$
where $A_i$ is the matrix formed by replacing the $i^{th}$ column of $A$ with the vector $B$. It is generally computationally less efficient than Gaussian elimination.

### Iterative Methods for Solving Linear Systems
For very large and sparse systems, direct methods like Gaussian elimination can be too computationally expensive. **Iterative methods** start with an initial guess and generate a sequence of increasingly accurate approximations to the solution.
* **Jacobi Method:** Solves the $i$-th equation for the $i$-th variable and uses values from the *previous* iteration for all other variables.
* **Gauss-Seidel Method:** Similar to Jacobi, but as soon as a new value for a variable is computed, it is used immediately in the subsequent equations within the same iteration, often leading to faster convergence.

### The Least Squares Method
When a system $AX=B$ is **overdetermined** (more equations than unknowns) and has no exact solution, the **least squares method** finds the "best fit" solution $\hat{\mathbf{x}}$. This $\hat{\mathbf{x}}$ is the vector that minimizes the squared Euclidean norm of the residual error, $||\mathbf{b} - A\hat{\mathbf{x}}||^2$.
* **Derivation:** The vector representing the error, $\mathbf{b} - A\hat{\mathbf{x}}$, must be orthogonal to the column space of $A$. This means it must be in the null space of $A^T$. So, $A^T(\mathbf{b} - A\hat{\mathbf{x}}) = 0$. This simplifies to the **Normal Equations**:
    $$ A^T A \hat{\mathbf{x}} = A^T \mathbf{b} $$
* **Solution:** If $A^TA$ is invertible, the unique least squares solution is:
    $$ \hat{\mathbf{x}} = (A^T A)^{-1} A^T \mathbf{b} $$
This method is fundamental to data fitting, linear regression, and many other statistical applications.

---
## Unveiling the Structure - Eigenvalues and Diagonalization

Eigenvalues and eigenvectors reveal the deep internal structure of a linear transformation.

### The Matrix Eigenvalue Problem
**Eigenvectors** of a square matrix $A$ are non-zero vectors $X$ whose direction is unchanged by the linear transformation $A$; they are only scaled by a factor $\lambda$, the **eigenvalue**.
$$ AX = \lambda X $$
This can be rewritten as $(A - \lambda I)X = 0$. For non-trivial solutions for $X$, the matrix $(A - \lambda I)$ must be singular, meaning its determinant must be zero. This gives the **characteristic equation**:
$$ \det(A - \lambda I) = 0 $$
Solving this polynomial equation for $\lambda$ yields the eigenvalues. For each eigenvalue, the corresponding eigenvectors are found by solving the system $(A - \lambda I)X = 0$.

**Example:** Find eigenvalues and eigenvectors of $A = \begin{pmatrix} 4 & -2 \\ 1 & 1 \end{pmatrix}$.
1.  **Characteristic Equation:**
    $\det(A-\lambda I) = \det\begin{pmatrix} 4-\lambda & -2 \\ 1 & 1-\lambda \end{pmatrix} = (4-\lambda)(1-\lambda) - (-2)(1) = 0$.
    $4 - 5\lambda + \lambda^2 + 2 = 0 \implies \lambda^2 - 5\lambda + 6 = 0$.
    $(\lambda-2)(\lambda-3) = 0$. The eigenvalues are $\lambda_1 = 2$ and $\lambda_2 = 3$.
2.  **Eigenvectors:**
    * For $\lambda_1=2$: $(A-2I)X=0 \implies \begin{pmatrix} 2 & -2 \\ 1 & -1 \end{pmatrix}\begin{pmatrix} x_1 \\ x_2 \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$. This gives $2x_1-2x_2=0$, or $x_1=x_2$. An eigenvector is $X_1 = \begin{pmatrix} 1 \\ 1 \end{pmatrix}$.
    * For $\lambda_2=3$: $(A-3I)X=0 \implies \begin{pmatrix} 1 & -2 \\ 1 & -2 \end{pmatrix}\begin{pmatrix} x_1 \\ x_2 \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$. This gives $x_1-2x_2=0$, or $x_1=2x_2$. An eigenvector is $X_2 = \begin{pmatrix} 2 \\ 1 \end{pmatrix}$.

### Numerical Methods for Eigenvalues
Finding roots of a high-degree characteristic polynomial is difficult. Numerical methods are often used.
* **The Power Method:** An iterative method to find the **dominant eigenvalue** (the one with the largest absolute value) and its corresponding eigenvector.
    **Algorithm:** Start with a non-zero vector $\mathbf{x}_0$. Repeatedly apply the matrix:
    $$ \mathbf{x}_{k+1} = \frac{A\mathbf{x}_k}{||\mathbf{x}_k||} $$
    As $k$ becomes large, $\mathbf{x}_k$ will converge to the eigenvector corresponding to the dominant eigenvalue, and the Rayleigh quotient $\frac{\mathbf{x}_k^T A \mathbf{x}_k}{\mathbf{x}_k^T \mathbf{x}_k}$ converges to the dominant eigenvalue itself.
* **Inclusion of Matrix Eigenvalues (Localization):** The **Gerschgorin Circle Theorem** provides a way to estimate the location of eigenvalues in the complex plane. It states that every eigenvalue of a complex $n \times n$ matrix $A$ lies within at least one of the Gerschgorin disks $D(a_{ii}, R_i)$, where $a_{ii}$ is a diagonal entry and $R_i = \sum_{j \neq i} |a_{ij}|$ is the sum of the absolute values of the off-diagonal entries in that row.
* **Advanced Decomposition Methods:** To find all eigenvalues, more robust methods are used.
    * **QR-Factorization and QR-Algorithm:** Any matrix $A$ can be decomposed as $A=QR$, where $Q$ is an orthogonal (or unitary) matrix and $R$ is an upper triangular matrix. The **QR algorithm** uses this factorization iteratively ($A_0=A$; $A_k=Q_kR_k$; $A_{k+1}=R_kQ_k$) to converge to a matrix form where the eigenvalues are easily read (e.g., an upper triangular or diagonal matrix).
    * **Tridiagonalization:** For symmetric matrices, a preparatory step is often to reduce the matrix to a much simpler **tridiagonal form** using Householder transformations. This makes the subsequent QR algorithm significantly faster.

### Eigenbases and Diagonalization
* **Eigenbasis:** If an $n \times n$ matrix has $n$ linearly independent eigenvectors, they form a basis for the vector space called an **eigenbasis**.
* **Diagonalization:** A square matrix $A$ is **diagonalizable** if there exists an invertible matrix $P$ and a diagonal matrix $D$ such that:
    $$ A = PDP^{-1} \quad \text{or} \quad D = P^{-1}AP $$
    This is possible if and only if $A$ has a complete set of $n$ linearly independent eigenvectors.
    * The columns of $P$ are the eigenvectors of $A$.
    * The diagonal entries of $D$ are the corresponding eigenvalues.
* **Continuing the example:**
    $P = \begin{pmatrix} 1 & 2 \\ 1 & 1 \end{pmatrix}$, $D = \begin{pmatrix} 2 & 0 \\ 0 & 3 \end{pmatrix}$. We can verify that $AP = PD$.

* **Significance:** Diagonalization simplifies matrix operations, like computing powers: $A^k = PD^kP^{-1}$.

### Quadratic Forms
A **quadratic form** is a homogeneous polynomial of degree two, represented as $\mathbf{x}^T A \mathbf{x}$ where $A$ is a symmetric matrix.
**Example:** $Q(x_1, x_2) = 5x_1^2 - 4x_1x_2 + 2x_2^2$.
The matrix is $A = \begin{pmatrix} 5 & -2 \\ -2 & 2 \end{pmatrix}$.
Diagonalizing $A$ via an orthogonal matrix $P$ (change of basis by rotation) transforms the quadratic form into one with no cross-product terms: $\lambda_1 y_1^2 + \lambda_2 y_2^2$, where $y_1, y_2$ are coordinates in the new basis of eigenvectors. This identifies the principal axes of the conic section represented by the form.

---
## Extensions to Complex Spaces

Linear algebra concepts extend naturally to **complex vector spaces**, where scalars are complex numbers.
* **Complex Matrices:**
    * **Hermitian Matrix ($A^\dagger = A$):** The complex analogue of a symmetric matrix. Its eigenvalues are always real.
    * **Unitary Matrix ($A^{-1} = A^\dagger$):** The complex analogue of an orthogonal matrix. It preserves the complex inner product.
* **Complex Forms:**
    * **Hermitian Form:** The complex analogue of a quadratic form, given by $\mathbf{x}^\dagger A \mathbf{x}$, where $A$ is a Hermitian matrix. Its value is always real.

---
## Key Takeaways: The Power of Linear Algebra

Linear algebra provides a unifying language for understanding systems, spaces, and transformations that exhibit linearity.

* **Abstract Structures and Concrete Representations:** Its power lies in the interplay between abstract concepts like vector spaces and linear transformations, and their concrete representation through matrices.
* **Solving Systems of Equations:** It provides robust and systematic algorithms like Gaussian elimination and iterative methods for solving linear systems, with rank and determinants offering profound insight into the existence and uniqueness of solutions. The least squares method provides a way to handle inconsistent systems.
* **Eigenvalues as Intrinsic Properties:** Eigenvalues and eigenvectors reveal the fundamental, basis-independent structure of a linear transformation, indicating which directions are simply scaled. Numerical methods like the power method and the QR algorithm allow for their computation.
* **Diagonalization as Simplification:** The process of diagonalization simplifies complex transformations by finding a basis (the eigenbasis) in which the transformation acts as a simple scaling, making problems like matrix exponentiation and the analysis of quadratic forms tractable.
* **A Foundational Tool:** From solving differential equations to performing data analysis in machine learning, and from the state vectors of quantum mechanics to the transformations in computer graphics, linear algebra is an indispensable tool across the scientific and technological landscape.