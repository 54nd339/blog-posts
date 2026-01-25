---
title: Vectors and Vector Algebra - The Mathematics of Direction and Magnitude
description: Explore the fundamental world of vectors, the mathematical objects that describe quantities with both magnitude and direction. This post delves into coordinate systems, vector algebra, dot and cross products, and their applications in geometry and physics, providing a comprehensive guide to their properties and operations.
date: 2023-01-07
draft: false
slug: /pensieve/maths/vectors
tags:
  - Mathematics
  - Algebra
---

In our scientific and mathematical descriptions of the world, we encounter quantities of two distinct types. Some, like mass, temperature, and time, can be fully described by a single number—a magnitude. These are scalars. Others, however, such as force, velocity, and displacement, require not only a magnitude but also a direction to be fully understood. These are **vectors**. Vector algebra provides the powerful framework for manipulating these directional quantities, forming a cornerstone of geometry, physics, engineering, and computer graphics. This exploration delves into the essential concepts of vectors, from their representation in space to the rich algebra that governs their interactions.

---
## Setting the Stage: 3D Space and Coordinates – Our Geometric Playground

To work with vectors, we first need a coordinate system to define position and direction.

* **Coordinate Axes and Planes in 3D Space:** The standard three-dimensional Cartesian system consists of three mutually perpendicular axes, typically labeled x, y, and z. They intersect at a point called the origin ($O$). These axes define three coordinate planes: the xy-plane, the yz-plane, and the xz-plane. The orientation usually follows the **right-hand rule**: if you curl the fingers of your right hand from the positive x-axis to the positive y-axis, your thumb points in the direction of the positive z-axis.

* **Coordinates of a Point in Space:** Any point $P$ in this space is uniquely identified by an ordered triple of real numbers $(x,y,z)$, representing its perpendicular distances from the yz, xz, and xy planes, respectively.

### Distance between Two Points
The distance $d$ between two points $P_1(x_1, y_1, z_1)$ and $P_2(x_2, y_2, z_2)$ is given by the distance formula.
$$ d = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2} $$
**Derivation:** This formula is an extension of the Pythagorean theorem. Consider a rectangular box with opposite vertices at $P_1$ and $P_2$. The lengths of the sides of the box are $|x_2-x_1|$, $|y_2-y_1|$, and $|z_2-z_1|$. The length of the diagonal on the base of the box is $d_{xy} = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$. Now, consider the right triangle formed by this base diagonal, the vertical edge of length $|z_2-z_1|$, and the main diagonal $d$. Applying the Pythagorean theorem again gives $d^2 = d_{xy}^2 + (z_2-z_1)^2$, which leads directly to the formula.

### Section Formula
This formula determines the coordinates of a point that divides the line segment joining two points $P_1(x_1,y_1,z_1)$ and $P_2(x_2,y_2,z_2)$ in a given ratio $m:n$.
* **Internal Division:** If the point $P(x,y,z)$ divides the segment internally:
    $$ P(x,y,z) = \left(\dfrac{mx_2+nx_1}{m+n}, \dfrac{my_2+ny_1}{m+n}, \dfrac{mz_2+nz_1}{m+n}\right) $$
* **External Division:** If the point $P(x,y,z)$ divides the segment externally:
    $$ P(x,y,z) = \left(\dfrac{mx_2-nx_1}{m-n}, \dfrac{my_2-ny_1}{m-n}, \dfrac{mz_2-nz_1}{m-n}\right) $$
**Derivation (Internal Division):** This is derived using similar triangles. By dropping perpendiculars from $P_1$, $P$, and $P_2$ to one of the coordinate planes (e.g., the xy-plane), and then to one of the axes (e.g., the x-axis), one can form two similar right-angled triangles. The ratio of their corresponding sides will be $m:n$. For the x-coordinate, this gives $\dfrac{x-x_1}{x_2-x} = \dfrac{m}{n}$. Solving for $x$ yields $x = \dfrac{mx_2+nx_1}{m+n}$. The same logic applies to the y and z coordinates.

---
## The Stars of the Show: Vectors and Scalars – More than Just a Number

* **Scalar:** A quantity that is fully described by a magnitude (a single real number) and a unit. Examples: mass, distance, speed, temperature.
* **Vector:** A quantity that is described by both a magnitude and a direction. Examples: displacement, velocity, force, acceleration. Vectors are typically represented by an arrow ($\vec{a}$) whose length indicates magnitude and whose orientation indicates direction.

* **Position Vector:** A vector that originates from the origin $(0,0,0)$ and terminates at a point $P(x,y,z)$ is called the position vector of that point, denoted $\vec{r} = \vec{OP}$.
* **Direction Cosines:** The **direction cosines** of a non-zero vector are the cosines of the angles ($\alpha, \beta, \gamma$) that the vector makes with the positive x, y, and z axes, respectively. They are usually denoted by $l, m, n$.
    $$ l = \cos\alpha, \quad m = \cos\beta, \quad n = \cos\gamma $$
    **Property:** For any vector, the sum of the squares of its direction cosines is always 1.
    $$ l^2 + m^2 + n^2 = 1 $$
    **Derivation:** If a vector $\vec{a}$ has components $(a_x, a_y, a_z)$, then from the geometry of the vector and its projections on the axes, we have $a_x = |\vec{a}|\cos\alpha$, $a_y = |\vec{a}|\cos\beta$, and $a_z = |\vec{a}|\cos\gamma$.
    We know that $|\vec{a}|^2 = a_x^2 + a_y^2 + a_z^2$.
    $|\vec{a}|^2 = (|\vec{a}|\cos\alpha)^2 + (|\vec{a}|\cos\beta)^2 + (|\vec{a}|\cos\gamma)^2$.
    $|\vec{a}|^2 = |\vec{a}|^2 (\cos^2\alpha + \cos^2\beta + \cos^2\gamma)$.
    Dividing by $|\vec{a}|^2$ gives $\cos^2\alpha + \cos^2\beta + \cos^2\gamma = 1$.

---
## The Vector Zoo: A Menagerie of Vector Types

* **Zero (Null) Vector ($\vec{0}$):** A vector with zero magnitude and an indeterminate direction.
* **Unit Vector ($\hat{a}$):** A vector with a magnitude of one. A unit vector in the direction of a non-zero vector $\vec{a}$ is given by $\hat{a} = \dfrac{\vec{a}}{|\vec{a}|}$. The standard basis vectors are $\hat{i}, \hat{j}, \hat{k}$ for the x, y, z axes.
* **Coinitial Vectors:** Two or more vectors having the same initial point.
* **Equal Vectors:** Two vectors are equal if they have the same magnitude and the same direction.
* **Negative of a Vector:** A vector having the same magnitude as a given vector $\vec{a}$ but the opposite direction, denoted $-\vec{a}$.
* **Free Vectors:** Vectors whose value depends only on length and direction, and are not fixed to a specific position in space.
* **Localised Vectors:** Vectors whose value depends on their line of action or point of application (e.g., force).
* **Parallel Vectors:** Two vectors are parallel if they have the same or opposite direction. For two non-zero vectors $\vec{a}$ and $\vec{b}$, they are parallel if $\vec{a} = k\vec{b}$ for some non-zero scalar $k$.
    * **Like Vectors:** Parallel vectors having the same direction ($k>0$).
    * **Unlike Vectors:** Parallel vectors having opposite directions ($k<0$).
* **Collinear Vectors:** Vectors that lie along the same line or parallel lines. This is another term for parallel vectors.
* **Coplanar Vectors:** Three or more vectors are coplanar if they lie in the same plane or are parallel to the same plane.

---
## Vector Algebra: The Rules of Engagement

* **Vector Joining Two Points:** The vector from point $A(x_1,y_1,z_1)$ to point $B(x_2,y_2,z_2)$ is given by:
    $$ \vec{AB} = \vec{OB} - \vec{OA} = (x_2-x_1)\hat{i} + (y_2-y_1)\hat{j} + (z_2-z_1)\hat{k} $$
* **Components of a Vector:** A vector $\vec{a}$ can be expressed as a sum of its components along the coordinate axes:
    * In 2D: $\vec{a} = a_x\hat{i} + a_y\hat{j}$. Magnitude $|\vec{a}| = \sqrt{a_x^2 + a_y^2}$.
    * In 3D: $\vec{a} = a_x\hat{i} + a_y\hat{j} + a_z\hat{k}$. Magnitude $|\vec{a}| = \sqrt{a_x^2 + a_y^2 + a_z^2}$.
* **Addition of Vectors:**
    * Geometrically, vectors are added using the **Triangle Law** (placing them head-to-tail) or the **Parallelogram Law** (placing them tail-to-tail and taking the diagonal).
    * Algebraically, addition is done component-wise:
        If $\vec{a} = a_x\hat{i} + a_y\hat{j} + a_z\hat{k}$ and $\vec{b} = b_x\hat{i} + b_y\hat{j} + b_z\hat{k}$, then
        $$ \vec{a} + \vec{b} = (a_x+b_x)\hat{i} + (a_y+b_y)\hat{j} + (a_z+b_z)\hat{k} $$
    * Properties: Commutative ($\vec{a}+\vec{b}=\vec{b}+\vec{a}$), Associative ($(\vec{a}+\vec{b})+\vec{c}=\vec{a}+(\vec{b}+\vec{c})$), has an additive identity ($\vec{0}$), and has an additive inverse ($-\vec{a}$).
    * $ | \vec{a} + \vec{b} | \leq | \vec{a} | + | \vec{b} | $ (Triangle Inequality).
    * $ | \vec{a} - \vec{b} | \geq | \vec{a} | - | \vec{b} | $ (Reverse Triangle Inequality).
* **Multiplication of a Vector by a Scalar:**
    If $k$ is a scalar and $\vec{a}$ is a vector, $k\vec{a}$ is a vector with magnitude $|k||\vec{a}|$ and direction the same as $\vec{a}$ if $k>0$, and opposite to $\vec{a}$ if $k<0$.
    $$ k\vec{a} = k(a_x\hat{i} + a_y\hat{j} + a_z\hat{k}) = (ka_x)\hat{i} + (ka_y)\hat{j} + (ka_z)\hat{k} $$

---
## Building with Vectors: Linear Combinations and Dependence

* **Linear Combination:** A vector $\vec{r}$ is a linear combination of a set of vectors $\{\vec{v_1}, \vec{v_2}, \ldots, \vec{v_n}\}$ if it can be expressed as a sum of scalar multiples of those vectors:
    $$ \vec{r} = c_1\vec{v_1} + c_2\vec{v_2} + \dots + c_n\vec{v_n} $$
    for some scalars $c_1, c_2, \ldots, c_n$.
* **Linear Independence:** A set of vectors $\{\vec{v_1}, \ldots, \vec{v_n}\}$ is **linearly independent** if the only solution to the equation $c_1\vec{v_1} + \dots + c_n\vec{v_n} = \vec{0}$ is $c_1=c_2=\dots=c_n=0$. This means no vector in the set can be written as a linear combination of the others.
* **Linear Dependence:** A set of vectors is **linearly dependent** if it is not linearly independent. This means at least one vector in the set can be expressed as a linear combination of the others.
    * A pair of collinear vectors is linearly dependent.
    * A triad of coplanar vectors is linearly dependent.
* **Theorems:**
    * If $\vec{a}$ and $\vec{b}$ be two non-collinear vectors, then every vector $\vec{r}$ coplanar with $\vec{a}$ and $\vec{b}$ can be expressed in one and only one way as a linear combination $x\vec{a} + y\vec{b}$; $x$ and $y$ being scalars.
    * If $\vec{a}$, $\vec{b}$ and $\vec{c}$ are non-coplanar vectors, then any vector $\vec{r}$ can be uniquely expressed as a linear combination $x\vec{a}+y\vec{b}+z\vec{c}$; $\vec{x}$, $\vec{y}$ and $\vec{z}$ being scalars.
    * If $\vec{a}$, $\vec{b}$ and $\vec{c}$ are coplanar vectors, then:
        $$ \begin{pmatrix} a_1 & a_2 & a_3 \\ b_1 & b_2 & b_3 \\ c_1 & c_2 & c_3 \\ \end{pmatrix} = 0 $$
* **Notes:**
    1. Two collinear vectors are always linearly dependent.
    2. Two non-collinear non-zero vectors are always linearly independent.
    3. Three coplanar vectors are always linearly dependent.
    4. Three non-coplanar non-zero vectors are always linearly independent.
    5. More than three vectors are always linearly dependent.
    6. Three points with position vectors $\vec{a}$, $\vec{b}$ and $\vec{c}$ are collinear if and only if there exist scalars $\vec{x}$, $\vec{y}$ and $\vec{z}$ not all zero such that:
        1. $x\vec{a}+y\vec{b}+z\vec{c} = 0$ and 
        2. $x + y + z = 0$.
    7. Four points with position vectors $\vec{a}$, $\vec{b}$, $\vec{c}$ and $\vec{d}$ are coplanar if there exist scalars $x$, $y$, $z$ and $w$ (sum of any two is not zero) such that $x\vec{a} + y\vec{b} + z\vec{c} + w\vec{d} = \vec{0}$ with $x + y + z + w = 0$.

---
## Multiplying Vectors: Dot and Cross Products

There are two primary ways to multiply vectors, yielding either a scalar or another vector.

### Scalar (Dot) Product ($\vec{a} \cdot \vec{b}$)
The dot product of two vectors results in a scalar quantity.
* **Definition:** $\vec{a} \cdot \vec{b} = |\vec{a}| |\vec{b}| \cos\theta$, where $\theta$ is the angle between the vectors.
* **Component Form:** $\vec{a} \cdot \vec{b} = a_x b_x + a_y b_y + a_z b_z$.
* **Physical Interpretation:** It represents the product of the magnitude of one vector and the magnitude of the projection of the other vector onto it. Widely used to calculate work done ($W = \vec{F}\cdot\vec{d}$) and power ($P = \vec{F}\cdot\vec{v}$).
* **Geometric Interpretation:** The dot product gives a measure of how much one vector extends in the direction of another. If $\vec{a}$ and $\vec{b}$ are orthogonal (perpendicular), then $\theta = 90^\circ$ and $\vec{a} \cdot \vec{b} = 0$.
* **Properties:**
    * **Commutative:** $\vec{a}\cdot\vec{b} = \vec{b}\cdot\vec{a}$
    * **Distributive over addition:** $\vec{a}\cdot(\vec{b}+\vec{c}) = \vec{a}\cdot\vec{b}+\vec{a}\cdot\vec{c}$
    * **Positive-definiteness:** $\vec{a}\cdot\vec{a} = |\vec{a}|^2$
    * Two non-zero vectors are orthogonal (perpendicular) if and only if $\vec{a}\cdot\vec{b}=0$.
    * **Cauchy-Schwarz Inequality:** For any vectors $\vec{a}$ and $\vec{b}$, $|\vec{a}\cdot\vec{b}| \leq |\vec{a}||\vec{b}|$. Equality holds if and only if $\vec{a}$ and $\vec{b}$ are parallel (or one of them is the zero vector).
    * **Triangle Inequality:** For any vectors $\vec{a}$ and $\vec{b}$, $|\vec{a} + \vec{b}| \leq |\vec{a}| + |\vec{b}|$.
    * **Parallelogram Law:** For any vectors $\vec{a}$ and $\vec{b}$, $|\vec{a} + \vec{b}|^2 + |\vec{a} - \vec{b}|^2 = 2(|\vec{a}|^2 + |\vec{b}|^2)$.

### Vector (Cross) Product ($\vec{a} \times \vec{b}$)
The cross product of two vectors results in a new vector.
* **Definition:** A vector whose magnitude is $|\vec{a} \times \vec{b}| = |\vec{a}| |\vec{b}| \sin\theta$. Its direction is perpendicular to the plane containing $\vec{a}$ and $\vec{b}$, determined by the right-hand rule.
* **Component Form:**
  $$ \vec{a} \times \vec{b} = \det \begin{pmatrix} \hat{i} & \hat{j} & \hat{k} \\ a_x & a_y & a_z \\ b_x & b_y & b_z \end{pmatrix} = (a_y b_z - a_z b_y)\hat{i} - (a_x b_z - a_z b_x)\hat{j} + (a_x b_y - a_y b_x)\hat{k} $$
* **Physical Interpretation:** Used to calculate torque ($\vec{\tau} = \vec{r}\times\vec{F}$), angular momentum ($\vec{L}=\vec{r}\times\vec{p}$), and magnetic force ($\vec{F}=q(\vec{v}\times\vec{B})$).
* **Geometric Interpretation:** The magnitude $|\vec{a}\times\vec{b}|$ represents the area of the parallelogram formed by vectors $\vec{a}$ and $\vec{b}$.
* **Properties:**
    * **Anti-commutative:** $\vec{a}\times\vec{b} = -\vec{b}\times\vec{a}$
    * **Distributive over addition:** $\vec{a}\times(\vec{b}+\vec{c}) = \vec{a}\times\vec{b}+\vec{a}\times\vec{c}$
    * **Associative with scalar multiplication:** $(k\vec{a})\times\vec{b} = k(\vec{a}\times\vec{b})$ for any scalar $k$.
    * **Anti-associative:** $(\vec{a}\times\vec{b})\times\vec{c} \neq \vec{a}\times(\vec{b}\times\vec{c})$.
    * $\vec{a}\times\vec{a} = \vec{0}$
    * Two non-zero vectors are parallel if and only if $\vec{a}\times\vec{b}=\vec{0}$.
* **Right-Hand Rule:** To find the direction of $\vec{a} \times \vec{b}$, point your right thumb in the direction of $\vec{a}$ and curl your fingers towards $\vec{b}$. Your thumb will point in the direction of the cross product.
* The unit vector in the direction of the cross product is given by:
$$ \hat{n} = \dfrac{\vec{a} \times \vec{b}}{|\vec{a} \times \vec{b}|} $$

### Finding the Angle Between Two Vectors
The dot product provides the easiest way to find the angle $\theta$ between two vectors:
$$ \cos\theta = \dfrac{\vec{a} \cdot \vec{b}}{|\vec{a}||\vec{b}|} $$
**Derivation of the Cosine Rule:** For a triangle with sides represented by vectors $\vec{a}, \vec{b}, \vec{c}$ such that $\vec{c} = \vec{a} - \vec{b}$, we can find the magnitude of $\vec{c}$.
$|\vec{c}|^2 = \vec{c}\cdot\vec{c} = (\vec{a}-\vec{b})\cdot(\vec{a}-\vec{b}) = \vec{a}\cdot\vec{a} - 2(\vec{a}\cdot\vec{b}) + \vec{b}\cdot\vec{b}$.
Replacing the dot products with magnitudes: $c^2 = a^2 - 2ab\cos\theta + b^2$, which is the Law of Cosines.

---
## Vector Decomposition: Resolving Vectors

It is often useful to break down a vector into components along specific directions.

### Finding Components of a Vector $\vec{b}$ along and Perpendicular to Vector $\vec{a}$
* **Component along $\vec{a}$ (Vector Projection):** This is a vector parallel to $\vec{a}$ whose magnitude is the scalar projection of $\vec{b}$ onto $\vec{a}$.
    The scalar projection is $|\vec{b}|\cos\theta = \dfrac{\vec{a}\cdot\vec{b}}{|\vec{a}|}$.
    The vector component is this magnitude multiplied by the unit vector $\hat{a} = \vec{a}/|\vec{a}|$.
    $$ \vec{b}_{\parallel} = \text{proj}_{\vec{a}}\vec{b} = \left(\dfrac{\vec{a}\cdot\vec{b}}{|\vec{a}|}\right) \dfrac{\vec{a}}{|\vec{a}|} = \left(\dfrac{\vec{a}\cdot\vec{b}}{|\vec{a}|^2}\right)\vec{a} $$
* **Component Perpendicular to $\vec{a}$:** This is simply the rest of vector $\vec{b}$ after the parallel component is subtracted.
    $$ \vec{b}_{\perp} = \vec{b} - \vec{b}_{\parallel} $$

### Resolving a Given Vector in the Direction of Given Two Orthogonal Vectors
If $\hat{u}$ and $\hat{v}$ are two orthogonal (perpendicular) unit vectors, then any vector $\vec{A}$ that lies in the plane defined by $\hat{u}$ and $\hat{v}$ can be written as a sum of its projections onto them:
$$ \vec{A} = (\vec{A} \cdot \hat{u})\hat{u} + (\vec{A} \cdot \hat{v})\hat{v} $$
In 3D, any vector $\vec{A}$ can be resolved using three orthogonal unit vectors (like $\hat{i}, \hat{j}, \hat{k}$).

---
## 3D Products: Scalar and Vector Triple Products

### Scalar Triple Product (STP)
* **Definition:** The scalar triple product of three vectors $\vec{a}, \vec{b}, \vec{c}$ is defined as $\vec{a} \cdot (\vec{b} \times \vec{c})$. It is denoted as $[\vec{a}\ \vec{b}\ \vec{c}]$.
* **Geometric Interpretation:** The magnitude of the scalar triple product, $|\vec{a} \cdot (\vec{b} \times \vec{c})|$, represents the **volume of the parallelepiped** with adjacent sides given by the vectors $\vec{a}, \vec{b}, \vec{c}$.
* **Determinant Form:**
    $$ \vec{a} \cdot (\vec{b} \times \vec{c}) = \det \begin{pmatrix} a_x & a_y & a_z \\ b_x & b_y & b_z \\ c_x & c_y & c_z \end{pmatrix} $$
* **Properties:**
    1. $(\vec{a} \times \vec{b}) \cdot \vec{c} = \vec{a} \cdot (\vec{b} \times \vec{c})$, i.e., position of the dot and the cross can be interchanged without altering the product.
    2. $[k \vec{a} \vec{b} \vec{c}] = k[\vec{a} \vec{b} \vec{c}]$ (where k is scalar)
    3. $[\vec{a} + \vec{b}\vec{c}\vec{d}] = [\vec{a}\vec{c}\vec{d}]+[\vec{b}\vec{c}\vec{d}]$
    4. $\vec{a}$, $\vec{b}$ and $\vec{c}$ in that order form a right-handed system if $[\vec{a}\vec{b}\vec{c}] > 0$, else they form a left-handed system if $[\vec{a}\vec{b}\vec{c}] < 0$.
    5. The necessary and sufficient condition for three non-zero, non-collinear vectors $\vec{a}$, $\vec{b}$ and $\vec{c}$ to be coplanar is that $[\vec{a}\vec{b}\vec{c}] = 0$.
    6. $[\vec{a}\vec{a}\vec{b}] = 0$ ( $\because$ $a$ is $\perp$ to $\vec{a} \times \vec{b}, \vec{a}\cdot(\vec{a} \times \vec{b}) = 0$)

### Vector Triple Product
* **Definition:** $\vec{a} \times (\vec{b} \times \vec{c})$. The result is a vector.
* **BAC-CAB Rule (Lagrange's Identity):** This identity provides a way to expand the vector triple product:
    $$ \vec{a} \times (\vec{b} \times \vec{c}) = \vec{b}(\vec{a}\cdot\vec{c}) - \vec{c}(\vec{a}\cdot\vec{b}) $$
    It's important to note that the cross product is not associative, i.e., $\vec{a} \times (\vec{b} \times \vec{c}) \neq (\vec{a} \times \vec{b}) \times \vec{c}$.

---
## The Other Side: Reciprocal System of Vectors

Given a set of three non-coplanar vectors $\{\vec{a}, \vec{b}, \vec{c}\}$ (which can form a basis), their **reciprocal system of vectors** $\{\vec{a}', \vec{b}', \vec{c}'\}$ is defined as:
$$ \vec{a}' = \dfrac{\vec{b} \times \vec{c}}{[\vec{a}\ \vec{b}\ \vec{c}]}, \quad \vec{b}' = \dfrac{\vec{c} \times \vec{a}}{[\vec{a}\ \vec{b}\ \vec{c}]}, \quad \vec{c}' = \dfrac{\vec{a} \times \vec{b}}{[\vec{a}\ \vec{b}\ \vec{c}]} $$
where $[\vec{a}\ \vec{b}\ \vec{c}]$ is the scalar triple product.
* **Properties:**
    * A vector from the original set dotted with its corresponding reciprocal vector is 1:
        $\vec{a} \cdot \vec{a}' = \vec{b} \cdot \vec{b}' = \vec{c} \cdot \vec{c}' = 1$.
    * A vector from the original set dotted with a non-corresponding reciprocal vector is 0:
        $\vec{a} \cdot \vec{b}' = \vec{a} \cdot \vec{c}' = \vec{b} \cdot \vec{a}' = \dots = 0$.
    * The scalar triple product of the reciprocal system is the reciprocal of the scalar triple product of the original system: $[\vec{a}'\ \vec{b}'\ \vec{c}'] = 1 / [\vec{a}\ \vec{b}\ \vec{c}]$.
    * The reciprocal of the reciprocal system is the original system.
    * The orthogonal triad of vectors $\hat{i}, \hat{j}, \hat{k}$ is self-reciprocal, meaning $\hat{i}' = \hat{i}$, $\hat{j}' = \hat{j}$, $\hat{k}' = \hat{k}$.
    * $\vec{a}$, $\vec{b}$, and $\vec{c}$ are non-coplanar if and only if their reciprocal vectors $\vec{a}'$, $\vec{b}'$, and $\vec{c}'$ are also non-coplanar.
This system is particularly useful in crystallography for describing crystal lattices and in physics for advanced mechanics and electromagnetism.

---
## Key Takeaways: The Power and Utility of Vector Algebra

Vectors and the algebra that governs them provide an indispensable language for describing the physical world and solving geometric problems.

* **Directional Quantities:** Vectors elegantly capture quantities that possess both magnitude and direction, which are ubiquitous in science and engineering.
* **Geometric Foundation:** The 3D Cartesian coordinate system provides the stage for vector algebra, with fundamental formulas for distance and division of segments.
* **Algebraic Operations:** Vectors can be added, subtracted, and multiplied by scalars in a component-wise manner. Multiplication between vectors is defined through the scalar (dot) product and the vector (cross) product, each with distinct geometric and physical interpretations.
* **Linear Dependence & Basis:** The concepts of linear independence and dependence allow us to understand collinearity, coplanarity, and the idea of a basis for a vector space.
* **Vector Products:** The dot product is used to find angles and projections. The cross product gives a vector perpendicular to the original two and its magnitude relates to area. The scalar and vector triple products extend these concepts to three vectors, representing volume and providing useful algebraic identities.
* **Resolving Vectors:** Any vector can be broken down into components along orthogonal directions, a crucial technique for problem-solving.
* **Broad Applicability:** Vector analysis is fundamental to nearly all areas of physics (mechanics, electromagnetism, fluid dynamics), as well as computer graphics, engineering, and navigation.

Mastering vector algebra opens the door to a more profound and quantitative understanding of the spatial relationships and physical laws that govern our universe.