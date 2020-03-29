---
title: Vectors - Algebra of Magnitude and Direction
description: A vector carries size and direction, so it describes force and velocity where a bare number can't. Components and the two products, what each one means geometrically, the triple products for volume, linear dependence, and the reciprocal system.
date: 2020-03-29
draft: false
slug: /maths/vectors
tags:
  - Mathematics
  - Vector Algebra
---

Temperature is one number. Velocity is not: "$30$ km/h" leaves out which way, and for anything that moves, turns, or pushes, the direction is half the information. A **vector** packages both — a magnitude and a direction — into one object, drawn as an arrow, and the point of vector algebra is that you can add, scale, and multiply these arrows with rules that respect the geometry.

This post builds that algebra: components and how vectors add, the two ways to multiply them (the dot product, which returns a number, and the cross product, which returns a perpendicular vector), what each product measures, the triple products that compute volumes, the notion of linear dependence that underlies "collinear" and "coplanar", and the reciprocal system used in crystallography.

---
## Space and coordinates

Three mutually perpendicular axes $x, y, z$ meet at the origin $O$, oriented by the **right-hand rule**: curl the right hand's fingers from $+x$ to $+y$ and the thumb points along $+z$. A point $P$ is the triple $(x, y, z)$ of its distances from the three coordinate planes.

**Distance** between $P_1$ and $P_2$ is Pythagoras applied twice — once across the base of the bounding box, once up to the far corner:
$$ d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2 + (z_2 - z_1)^2} $$

**Section formula** for the point dividing $P_1P_2$ in ratio $m : n$, from similar triangles on each axis:
$$ P = \left(\frac{m x_2 + n x_1}{m + n},\; \frac{m y_2 + n y_1}{m + n},\; \frac{m z_2 + n z_1}{m + n}\right) \quad\text{(internal; flip the } n \text{ signs for external)} $$

---
## Scalars, vectors, and direction cosines

A **scalar** is a magnitude alone (mass, speed, temperature); a **vector** has magnitude *and* direction (displacement, velocity, force). The **position vector** of $P(x, y, z)$ is $\vec{r} = \overrightarrow{OP}$.

The **direction cosines** $l, m, n$ of a vector are the cosines of the angles $\alpha, \beta, \gamma$ it makes with the positive axes. Since a vector's components are $a_x = |\vec a|\cos\alpha$ and so on, and $|\vec a|^2 = a_x^2 + a_y^2 + a_z^2$, dividing through gives
$$ l^2 + m^2 + n^2 = 1 $$

---
## Types of vector

- **Zero vector** $\vec 0$ — magnitude zero, direction undefined.
- **Unit vector** $\hat a = \vec a / |\vec a|$ — magnitude $1$; the axis unit vectors are $\hat i, \hat j, \hat k$.
- **Equal vectors** — same magnitude and direction (position irrelevant).
- **Negative** $-\vec a$ — same magnitude, opposite direction.
- **Free vs localised** — a free vector is fixed only by length and direction; a localised one (a force) also depends on its line of action.
- **Parallel / collinear** — $\vec a = k\vec b$ for a scalar $k$; *like* if $k > 0$, *unlike* if $k < 0$.
- **Coplanar** — three or more vectors lying in (or parallel to) one plane.

---
## Vector algebra

The vector from $A$ to $B$ is $\overrightarrow{AB} = \overrightarrow{OB} - \overrightarrow{OA} = (x_2 - x_1)\hat i + (y_2 - y_1)\hat j + (z_2 - z_1)\hat k$, with magnitude $|\vec a| = \sqrt{a_x^2 + a_y^2 + a_z^2}$.

**Addition** is head-to-tail (triangle law) or the diagonal of the parallelogram — algebraically, componentwise: $(\vec a + \vec b)_i = a_i + b_i$. It is commutative and associative, with identity $\vec 0$ and inverse $-\vec a$, and obeys
$$ |\vec a + \vec b| \le |\vec a| + |\vec b|, \qquad |\vec a - \vec b| \ge \big||\vec a| - |\vec b|\big| $$

**Scalar multiplication** $k\vec a$ has magnitude $|k||\vec a|$, same direction for $k > 0$ and opposite for $k < 0$, componentwise $(k\vec a)_i = k a_i$.

---
## Linear combinations and dependence

$\vec r = c_1\vec v_1 + \cdots + c_n\vec v_n$ is a **linear combination**. A set is **linearly independent** if $c_1\vec v_1 + \cdots + c_n\vec v_n = \vec 0$ forces every $c_i = 0$; otherwise **linearly dependent** — some vector is a combination of the rest. In $\mathbb{R}^3$:

- two non-collinear vectors are independent, and every vector in their plane is $x\vec a + y\vec b$ for a *unique* $(x, y)$;
- three non-coplanar vectors are independent and span space: every $\vec r = x\vec a + y\vec b + z\vec c$ uniquely;
- three vectors are coplanar iff $\det[a_i\ b_i\ c_i] = 0$;
- any four (or more) vectors in $\mathbb{R}^3$ are dependent.

Two position-vector conditions worth memorising: points $\vec a, \vec b, \vec c$ are **collinear** iff scalars not all zero give $x\vec a + y\vec b + z\vec c = \vec 0$ *with* $x + y + z = 0$; four points are **coplanar** under the same statement with a fourth term and $x + y + z + w = 0$.

---
## The dot product

$$ \vec a \cdot \vec b = |\vec a|\,|\vec b|\cos\theta = a_x b_x + a_y b_y + a_z b_z $$
It measures how much of one vector lies along the other. Zero means perpendicular; $\vec a \cdot \vec a = |\vec a|^2$. It is commutative and distributes over addition, and it gives:

- the **angle**: $\cos\theta = \dfrac{\vec a \cdot \vec b}{|\vec a||\vec b|}$ — expand $|\vec a - \vec b|^2 = (\vec a - \vec b)\cdot(\vec a - \vec b)$ and out falls the law of cosines $c^2 = a^2 + b^2 - 2ab\cos\theta$;
- **work** $W = \vec F \cdot \vec d$ and **power** $P = \vec F \cdot \vec v$ in physics;
- the **Cauchy–Schwarz** inequality $|\vec a \cdot \vec b| \le |\vec a||\vec b|$ (equality iff parallel), and the **parallelogram law** $|\vec a + \vec b|^2 + |\vec a - \vec b|^2 = 2(|\vec a|^2 + |\vec b|^2)$.

---
## The cross product

$$ \vec a \times \vec b = \det\begin{pmatrix} \hat i & \hat j & \hat k \\ a_x & a_y & a_z \\ b_x & b_y & b_z \end{pmatrix} = (a_y b_z - a_z b_y)\hat i - (a_x b_z - a_z b_x)\hat j + (a_x b_y - a_y b_x)\hat k $$
The result is a vector: magnitude $|\vec a||\vec b|\sin\theta$ — the **area of the parallelogram** on $\vec a$ and $\vec b$ — and direction perpendicular to both, fixed by the **right-hand rule** (point the fingers along $\vec a$, curl toward $\vec b$, thumb gives $\vec a \times \vec b$).

![A right hand with the index finger pointing along vector a, middle finger along vector b, and thumb pointing up along the cross product a cross b, perpendicular to both.](../images/cross-product-right-hand-rule.png "The right-hand rule fixes the direction of a × b: perpendicular to the plane of a and b, on the side your thumb points. Source: Wikimedia Commons.")

It is **anti-commutative** ($\vec a \times \vec b = -\vec b \times \vec a$), distributes over addition, gives $\vec a \times \vec a = \vec 0$, and is zero exactly when the vectors are parallel. It is **not associative**. In physics it is torque $\vec\tau = \vec r \times \vec F$, angular momentum $\vec L = \vec r \times \vec p$, and the magnetic force $\vec F = q(\vec v \times \vec B)$ — see [rotational dynamics](/citadel/physics/rotations).

**Resolving a vector.** The part of $\vec b$ along $\vec a$ is the projection
$$ \vec b_\parallel = \left(\frac{\vec a \cdot \vec b}{|\vec a|^2}\right)\vec a, \qquad \vec b_\perp = \vec b - \vec b_\parallel $$
and against an orthonormal pair $\hat u, \hat v$ any planar $\vec A = (\vec A \cdot \hat u)\hat u + (\vec A \cdot \hat v)\hat v$ — in 3D, the same with $\hat i, \hat j, \hat k$.

---
## Triple products

**Scalar triple product** $[\vec a\ \vec b\ \vec c] = \vec a \cdot (\vec b \times \vec c)$. Its absolute value is the **volume of the parallelepiped** on the three vectors, and
$$ [\vec a\ \vec b\ \vec c] = \det\begin{pmatrix} a_x & a_y & a_z \\ b_x & b_y & b_z \\ c_x & c_y & c_z \end{pmatrix} $$
The dot and cross can be swapped without changing it, it is linear in each slot, and it is **zero iff the three vectors are coplanar** (no volume). Its sign tells handedness: positive for a right-handed triple.

**Vector triple product** $\vec a \times (\vec b \times \vec c)$ expands by the **BAC–CAB rule**:
$$ \vec a \times (\vec b \times \vec c) = \vec b\,(\vec a \cdot \vec c) - \vec c\,(\vec a \cdot \vec b) $$
The result lies in the plane of $\vec b$ and $\vec c$, and the bracketing matters: $\vec a \times (\vec b \times \vec c) \neq (\vec a \times \vec b) \times \vec c$ in general.

---
## The reciprocal system

Given non-coplanar $\vec a, \vec b, \vec c$, their **reciprocal system** is
$$ \vec a\,' = \frac{\vec b \times \vec c}{[\vec a\ \vec b\ \vec c]}, \qquad \vec b\,' = \frac{\vec c \times \vec a}{[\vec a\ \vec b\ \vec c]}, \qquad \vec c\,' = \frac{\vec a \times \vec b}{[\vec a\ \vec b\ \vec c]} $$
built so that $\vec a \cdot \vec a\,' = \vec b \cdot \vec b\,' = \vec c \cdot \vec c\,' = 1$ while every mismatched pair dots to $0$ — a "dual basis". Then $[\vec a\,'\ \vec b\,'\ \vec c\,'] = 1/[\vec a\ \vec b\ \vec c]$, the orthonormal triad $\hat i, \hat j, \hat k$ is its own reciprocal, and taking the reciprocal twice returns the original. Crystallographers use exactly this to build the reciprocal lattice from a crystal's unit cell.

---
## The one idea to keep

Two products, two meanings: the dot product projects and gives angles, the cross product builds a perpendicular and gives areas; chain them and the scalar triple product gives volume. That is the whole toolkit for spatial reasoning, and it carries directly into [3D analytic geometry](/citadel/maths/3d-geometry) (lines and planes as vector equations), into [linear algebra](/citadel/maths/linear-algebra) (these are the first examples of a vector space), and into [vector calculus](/citadel/maths/vector-calculus), where the vectors become fields that vary from point to point.
