---
title: An Exploration of Geometry - From Lines and Planes to Topology and Fractals
description: Journey through the worlds of geometry, from the algebraic precision of 3D coordinate systems to the abstract, flexible spaces of topology and the infinite complexity of fractal geometry. This post covers the essential equations and concepts that describe the structure of space.
date: 2023-01-08
draft: false
slug: /pensieve/maths/3d-geometry
tags:
  - Mathematics
  - Coordinate Geometry
---

Our physical world exists in three dimensions, a space of height, width, and depth that we navigate every day. To describe, analyze, and manipulate objects within this space, we rely on the powerful framework of three-dimensional coordinate geometry. By extending the Cartesian plane to include a third axis, we can assign a unique address to every point in space and represent geometric objects like lines, planes, and spheres with the precision of algebraic equations.

This exploration delves into the fundamental principles of 3D coordinate geometry. We will then venture beyond these classical structures into the more abstract realms of **topology**, the study of spatial properties preserved under continuous deformation, and **fractal geometry**, the science of infinite complexity and self-similarity.

---
## The Foundation: 3D Space and Direction

### Coordinate Axes and Coordinate Planes in 3D Space
The three-dimensional Cartesian system is defined by three mutually perpendicular axes, typically labeled x, y, and z.
* **Origin:** The point where the three axes intersect, denoted $(0,0,0)$.
* **Coordinate Planes:** The axes define three coordinate planes: the xy-plane (where $z=0$), the yz-plane (where $x=0$), and the xz-plane (where $y=0$). These planes divide space into eight regions called **octants**.
* **Coordinates of a Point:** Any point $P$ in space is uniquely identified by an ordered triple $(x,y,z)$.

### Direction Cosines & Direction Ratios
To define the orientation of a line or a vector in space, we use direction cosines and direction ratios.
* **Direction Cosines ($l, m, n$):** The direction cosines of a directed line are the cosines of the angles ($\alpha, \beta, \gamma$) that the line makes with the positive x, y, and z axes, respectively.
    $$ l = \cos\alpha, \quad m = \cos\beta, \quad n = \cos\gamma $$
    A fundamental property is that the sum of the squares of the direction cosines is always one:
    $$ l^2 + m^2 + n^2 = 1 $$
    **Derivation:** Consider a position vector $\vec{r} = x\hat{i} + y\hat{j} + z\hat{k}$ from the origin. The projections on the axes are $x, y, z$. We have $x = |\vec{r}|\cos\alpha$, $y=|\vec{r}|\cos\beta$, $z=|\vec{r}|\cos\gamma$. Since $|\vec{r}|^2 = x^2+y^2+z^2$, substituting gives $|\vec{r}|^2 = |\vec{r}|^2(\cos^2\alpha+\cos^2\beta+\cos^2\gamma)$, which simplifies to $l^2+m^2+n^2=1$.

* **Direction Ratios ($a, b, c$):** Any three numbers that are proportional to the direction cosines of a line are called the direction ratios of the line. If $l,m,n$ are the direction cosines and $a,b,c$ are the direction ratios, then:
    $$ \dfrac{l}{a} = \dfrac{m}{b} = \dfrac{n}{c} = k $$
    From this, we can find the direction cosines from direction ratios:
    $l = \pm \dfrac{a}{\sqrt{a^2+b^2+c^2}}$, $m = \pm \dfrac{b}{\sqrt{a^2+b^2+c^2}}$, $n = \pm \dfrac{c}{\sqrt{a^2+b^2+c^2}}$.

---
## The Straight and Narrow: The Straight Line in 3D

A straight line in space can be uniquely determined if we know a point on the line and its direction, or if we know two points on the line.

### Equation of a Line Passing through a Given Point and Parallel to a Given Vector
Let the line pass through a point $A$ with position vector $\vec{a}$, and be parallel to a vector $\vec{b}$. Let $\vec{r}$ be the position vector of any general point $P$ on the line.
* **Vector Form:**
    The vector $\vec{AP}$ must be parallel to $\vec{b}$. Therefore, $\vec{AP} = \lambda\vec{b}$ for some scalar parameter $\lambda$.
    Since $\vec{AP} = \vec{r} - \vec{a}$, we have $\vec{r} - \vec{a} = \lambda\vec{b}$.
    $$ \vec{r} = \vec{a} + \lambda\vec{b} $$
* **Cartesian Form:**
    Let $A=(x_1, y_1, z_1)$, $P=(x,y,z)$, and the direction ratios of $\vec{b}$ be $(a,b,c)$.
    Then $\vec{a} = x_1\hat{i}+y_1\hat{j}+z_1\hat{k}$, $\vec{r} = x\hat{i}+y\hat{j}+z\hat{k}$, and $\vec{b} = a\hat{i}+b\hat{j}+c\hat{k}$.
    From the vector form: $(x\hat{i}+y\hat{j}+z\hat{k}) = (x_1\hat{i}+y_1\hat{j}+z_1\hat{k}) + \lambda(a\hat{i}+b\hat{j}+c\hat{k})$.
    Equating components: $x=x_1+\lambda a$, $y=y_1+\lambda b$, $z=z_1+\lambda c$.
    Solving for $\lambda$ gives the symmetric Cartesian form:
    $$ \dfrac{x-x_1}{a} = \dfrac{y-y_1}{b} = \dfrac{z-z_1}{c} $$

### Equation of a Line Passing through Two Given Points
Let the line pass through two points $A$ and $B$ with position vectors $\vec{a}$ and $\vec{b}$ respectively.
* **Vector Form:**
    The line passes through point $A$ and is parallel to the vector $\vec{AB} = \vec{b} - \vec{a}$. Using the previous form:
    $$ \vec{r} = \vec{a} + \lambda(\vec{b}-\vec{a}) $$
* **Cartesian Form:**
    Let $A=(x_1, y_1, z_1)$ and $B=(x_2, y_2, z_2)$. The direction ratios of the line are $(x_2-x_1, y_2-y_1, z_2-z_1)$. Using the point-direction form:
    $$ \dfrac{x-x_1}{x_2-x_1} = \dfrac{y-y_1}{y_2-y_1} = \dfrac{z-z_1}{z_2-z_1} $$

---
## When Lines Meet (or Don't): Relationships Between Two Lines

### Angle between Two Lines
The angle between two lines is the angle between their direction vectors. Let the lines have direction vectors $\vec{b_1}$ and $\vec{b_2}$ (or direction ratios $a_1,b_1,c_1$ and $a_2,b_2,c_2$).
* **Formula:** The angle $\theta$ between the lines is given by the dot product:
    $$ \cos\theta = \dfrac{|\vec{b_1} \cdot \vec{b_2}|}{|\vec{b_1}||\vec{b_2}|} = \dfrac{|a_1a_2+b_1b_2+c_1c_2|}{\sqrt{a_1^2+b_1^2+c_1^2}\sqrt{a_2^2+b_2^2+c_2^2}} $$
* **Condition for Perpendicularity:** The lines are perpendicular if $\theta=90^\circ$, so $\cos\theta=0$.
    $$ \vec{b_1} \cdot \vec{b_2} = 0 \quad \text{or} \quad a_1a_2+b_1b_2+c_1c_2 = 0 $$
* **Condition for Parallelism:** The direction vectors are parallel, so $\vec{b_1} = k\vec{b_2}$ for some scalar $k$.
    $$ \dfrac{a_1}{a_2} = \dfrac{b_1}{b_2} = \dfrac{c_1}{c_2} $$

### Skew Lines
Two lines in 3D space that are neither parallel nor intersecting are called **skew lines**.

---
## Distances Involving Lines: Points and Pairs

### Perpendicular Distance of a Point from a Line
Let the line be $\vec{r} = \vec{a} + \lambda\vec{b}$ and the external point have position vector $\vec{p}$.
* **Formula:** The perpendicular distance $d$ is:
    $$ d = \dfrac{|(\vec{p}-\vec{a}) \times \vec{b}|}{|\vec{b}|} $$
* **Derivation:** Let $A$ be the point on the line with position vector $\vec{a}$, and let $P$ be the external point. The magnitude of the cross product $|(\vec{p}-\vec{a}) \times \vec{b}|$ represents the area of the parallelogram formed by vectors $\vec{AP}$ and $\vec{b}$. The area of this parallelogram is also given by base $\times$ height, which is $|\vec{b}| \times d$. Equating these two expressions for the area gives the formula.
* **Foot of the Perpendicular:** To find the coordinates of the foot of the perpendicular, $M$, from point $P$ to the line, we can represent a general point on the line as $M = \vec{a}+\lambda\vec{b}$. The vector $\vec{PM}$ must be perpendicular to the line's direction vector $\vec{b}$. So, $\vec{PM} \cdot \vec{b} = 0$. This allows us to solve for the specific value of $\lambda$ corresponding to point $M$.
* **Image of a Point in the Line:** Once the foot of the perpendicular $M$ is found, it is the midpoint of the original point $P$ and its image $P'$. The coordinates of $P'$ can be found using the midpoint formula.

### Shortest Distance between Two Lines
* **Shortest Distance between Two Skew Lines:**
    Let the lines be $\vec{r} = \vec{a_1} + \lambda\vec{b_1}$ and $\vec{r} = \vec{a_2} + \mu\vec{b_2}$.
    * **Formula:** The shortest distance is the length of the projection of the vector joining a point on each line ($\vec{a_2}-\vec{a_1}$) onto the direction of the common perpendicular vector ($\vec{b_1} \times \vec{b_2}$).
        $$ d = \dfrac{|(\vec{a_2}-\vec{a_1}) \cdot (\vec{b_1} \times \vec{b_2})|}{|\vec{b_1} \times \vec{b_2}|} $$
* **Condition for Lines to Intersect:** Two lines intersect if the shortest distance between them is zero. This occurs when the vectors $(\vec{a_2}-\vec{a_1})$, $\vec{b_1}$, and $\vec{b_2}$ are coplanar. The condition is that their scalar triple product is zero:
    $$ (\vec{a_2}-\vec{a_1}) \cdot (\vec{b_1} \times \vec{b_2}) = 0 $$
* **Distance between Two Parallel Lines:**
    Let the lines be $\vec{r} = \vec{a_1} + \lambda\vec{b}$ and $\vec{r} = \vec{a_2} + \mu\vec{b}$. The distance is the perpendicular distance of the point $A(\vec{a_1})$ on the first line from the second line.
    $$ d = \dfrac{|(\vec{a_2}-\vec{a_1}) \times \vec{b}|}{|\vec{b}|} $$

---
## The Flat World: The Plane – A 2D Surface in 3D Space

A plane is a flat, two-dimensional surface that extends infinitely.

### Equations of a Plane
1.  **Normal Form:**
    * **Vector Form:** Let $\hat{n}$ be a unit vector from the origin normal to the plane, and let $p$ be the perpendicular distance of the plane from the origin. For any point $P(\vec{r})$ on the plane, the projection of $\vec{r}$ onto $\hat{n}$ is $p$.
        $$ \vec{r} \cdot \hat{n} = p $$
    * **Cartesian Form:** If the direction cosines of the normal are $(l, m, n)$, then $\hat{n} = l\hat{i}+m\hat{j}+n\hat{k}$.
        $$ lx + my + nz = p $$

2.  **Passing through a Point and Normal to a Vector:**
    Let the plane pass through a point $A$ with position vector $\vec{a}$ and be normal to a vector $\vec{n}$. For any point $P(\vec{r})$ on the plane, the vector $\vec{AP} = \vec{r}-\vec{a}$ lies in the plane and must be perpendicular to $\vec{n}$.
    $$ (\vec{r} - \vec{a}) \cdot \vec{n} = 0 \quad \text{or} \quad \vec{r} \cdot \vec{n} = \vec{a} \cdot \vec{n} $$
    In Cartesian form, for point $(x_1,y_1,z_1)$ and normal with direction ratios $(A,B,C)$:
    $A(x-x_1) + B(y-y_1) + C(z-z_1) = 0$. This leads to the general equation $Ax+By+Cz+D=0$.

3.  **Passing through Three Non-collinear Points:**
    Let the points be $A(\vec{a}), B(\vec{b}), C(\vec{c})$. The vectors $\vec{AB} = \vec{b}-\vec{a}$ and $\vec{AC} = \vec{c}-\vec{a}$ lie in the plane. The normal to the plane is perpendicular to both, so $\vec{n} = (\vec{b}-\vec{a}) \times (\vec{c}-\vec{a})$. Using the point-normal form:
    $$ (\vec{r} - \vec{a}) \cdot [(\vec{b}-\vec{a}) \times (\vec{c}-\vec{a})] = 0 $$
    This is the scalar triple product representation.

4.  **Intercept Form:**
    The equation of a plane that makes intercepts $a, b, c$ on the x, y, z axes respectively is:
    $$ \dfrac{x}{a} + \dfrac{y}{b} + \dfrac{z}{c} = 1 $$
    **Derivation:** The plane passes through points $(a,0,0), (0,b,0), (0,c,0)$. Substituting these into the general equation $Ax+By+Cz=D$ allows one to solve for $A,B,C$ in terms of $D,a,b,c$, leading to this form.

5.  **Passing through the Intersection of Two Planes:**
    The equation of any plane passing through the line of intersection of two planes $P_1 \equiv \vec{r}\cdot\vec{n}_1 - d_1 = 0$ and $P_2 \equiv \vec{r}\cdot\vec{n}_2 - d_2 = 0$ is given by the family of planes:
    $$ P_1 + \lambda P_2 = 0 $$
    $$ (\vec{r}\cdot\vec{n}_1 - d_1) + \lambda(\vec{r}\cdot\vec{n}_2 - d_2) = 0 $$

### Relationships between Planes and Lines
* **Angle between Two Planes:** The angle between two planes is the angle between their normal vectors, $\vec{n_1}$ and $\vec{n_2}$.
    $$ \cos\theta = \dfrac{|\vec{n_1} \cdot \vec{n_2}|}{|\vec{n_1}||\vec{n_2}|} $$
    * **Perpendicularity:** Planes are perpendicular if $\vec{n_1} \cdot \vec{n_2} = 0$.
    * **Parallelism:** Planes are parallel if their normal vectors are parallel, $\vec{n_1} = k\vec{n_2}$.

* **Line of Intersection of Two Planes:** The line of intersection is perpendicular to both normal vectors, so its direction vector $\vec{b}$ is given by $\vec{b} = \vec{n_1} \times \vec{n_2}$. To find a point on the line, one can set one coordinate (e.g., $z=0$) in both plane equations and solve the resulting system of two linear equations in two variables.

* **Angle between a Line and a Plane:** Let the line have direction vector $\vec{b}$ and the plane have normal vector $\vec{n}$. The angle $\theta$ between the line and the normal is found using the dot product. The angle $\phi$ between the line and the plane is the complement of $\theta$, so $\phi = 90^\circ - \theta$.
    $$ \sin\phi = \cos\theta = \dfrac{|\vec{b}\cdot\vec{n}|}{|\vec{b}||\vec{n}|} $$

### Distances Involving Planes
* **Distance from a Point to a Plane:**
    The perpendicular distance from a point with position vector $\vec{p}$ to the plane $\vec{r}\cdot\vec{n} = d$ is:
    $$ D = \dfrac{|\vec{p}\cdot\vec{n} - d|}{|\vec{n}|} $$
    In Cartesian form, the distance from point $(x_0,y_0,z_0)$ to the plane $Ax+By+Cz+D=0$ is:
    $$ D = \dfrac{|Ax_0+By_0+Cz_0+D|}{\sqrt{A^2+B^2+C^2}} $$
* **Image of a Point in a Plane:** Found by determining the foot of the perpendicular from the point to the plane, and then extending that line segment an equal distance on the other side.
* **Distance between Parallel Planes:** For planes $Ax+By+Cz+D_1=0$ and $Ax+By+Cz+D_2=0$, the distance is:
    $$ D = \dfrac{|D_1-D_2|}{\sqrt{A^2+B^2+C^2}} $$

---
## The Perfect Shape: The Sphere

A **sphere** is the locus of a point in space that is at a constant distance (the radius) from a fixed point (the center).

* **Equations:**
    * **Center-Radius Form:** A sphere with center $(h,k,l)$ and radius $r$:
        $$ (x-h)^2 + (y-k)^2 + (z-l)^2 = r^2 $$
    * **General Form:**
        $$ x^2+y^2+z^2+2ux+2vy+2wz+d=0 $$
        Center is $(-u,-v,-w)$ and radius is $r=\sqrt{u^2+v^2+w^2-d}$.
* **Diameter Form:**
    The equation of the sphere with the line segment joining points $(x_1,y_1,z_1)$ and $(x_2,y_2,z_2)$ as a diameter is:
    $$ (x-x_1)(x-x_2) + (y-y_1)(y-y_2) + (z-z_1)(z-z_2) = 0 $$
    **Derivation:** Any point $P(x,y,z)$ on the sphere forms a right angle with the diameter endpoints $A$ and $B$. The vectors $\vec{PA}$ and $\vec{PB}$ are perpendicular, so their dot product is zero: $\vec{PA} \cdot \vec{PB} = 0$. Expanding this gives the formula.

---
## Beyond Euclidean Shapes - Topology and Fractals

While coordinate geometry provides a rigid framework for measurement, other branches of geometry explore different, more abstract properties of space and shape.

### Topology: The Geometry of Stretching and Bending
**Topology** is a major area of mathematics concerned with the properties of space that are preserved under continuous deformations, such as stretching, twisting, and bending, but not tearing or gluing. It is informally known as "rubber sheet geometry."

* **Topological Spaces:** The core object of study is a **topological space**, which is a set of points, along with a set of subsets (called **open sets**) that satisfy three axioms:
    1.  The empty set and the entire space are open sets.
    2.  The union of any collection of open sets is an open set.
    3.  The intersection of any finite collection of open sets is an open set.
    This abstract definition of "open sets" generalizes the concept of open intervals on the real number line and allows us to define continuity and connectedness in very broad settings.

* **Homeomorphism and Topological Invariants:**
    In topology, two objects are considered equivalent if one can be continuously deformed into the other. Such an equivalence is called a **homeomorphism**.
    The properties that remain unchanged by homeomorphisms are called **topological invariants**. These are the properties topologists truly care about.
    * **Example:** A coffee mug and a donut (a torus) are famously homeomorphic. One can be reshaped into the other without tearing. They both have exactly one "hole," which is a topological invariant. A sphere, having no holes, is topologically distinct from a torus.
    * **Key Invariants:**
        * **Connectedness:** Whether the space is a single piece.
        * **Compactness:** A generalization of being closed and bounded.
        * **Genus:** The number of "holes" of a surface.
        * **Euler Characteristic ($\chi$):** For polyhedra, this is $\chi = V - E + F$ (Vertices - Edges + Faces). For a sphere, $\chi=2$; for a torus, $\chi=0$.

### Fractal Geometry: The Beauty of Infinite Complexity
**Fractal geometry** is the study of shapes that are "rough" or fragmented and exhibit intricate detail at all scales of magnification. The term was coined by Benoît Mandelbrot.

* **Key Properties of Fractals:**
    1.  **Self-Similarity:** The shape is made of smaller copies of itself. The parts resemble the whole, either exactly or statistically.
    2.  **Infinite Detail:** Zooming in on a fractal reveals more and more complexity, a process that can continue indefinitely.
    3.  **Fractal Dimension (Non-Integer Dimension):** This is a key concept that distinguishes fractals from traditional Euclidean shapes. A fractal's "dimension" measures its complexity and space-filling ability, and it can be a non-integer value.
        * **Derivation/Example (Similarity Dimension):** A line segment (1D) can be made of $N=2$ copies of itself, each scaled down by a factor $S=2$. A square (2D) can be made of $N=4$ copies, each scaled by $S=2$. A cube (3D) can be made of $N=8$ copies, each scaled by $S=2$. Notice that $N = S^D$, where $D$ is the dimension. This gives a general formula for dimension: $D = \frac{\log N}{\log S}$.
        * Let's apply this to the **Koch curve**. To build it, you start with a line segment, replace the middle third with two sides of an equilateral triangle. Each step creates an object made of $N=4$ smaller copies of the previous object, where each copy is scaled down by a factor of $S=3$.
        * The fractal dimension of the Koch curve is therefore:
          $$ D = \frac{\log(4)}{\log(3)} \approx 1.26 $$
          It is more than a 1D line but does not fill a 2D plane.

* **Classic Examples of Fractals:**
    * **The Koch Snowflake:** Formed by applying the Koch curve construction to the sides of an equilateral triangle. It has a finite area but an infinite perimeter.
    * **The Sierpinski Triangle:** An equilateral triangle subdivided recursively into smaller equilateral triangles.
    * **The Mandelbrot Set:** A famous set of complex numbers whose boundary is an intricate and infinitely complex fractal.

* **Applications:** Fractals are not just mathematical curiosities; they provide powerful models for many natural phenomena like coastlines, snowflakes, lightning bolts, and the structure of plants. They are also used in computer graphics, antenna design, and data compression.

---
## Key Takeaways: The Algebraic Structure of 3D Space

The application of vector and Cartesian methods provides a comprehensive and powerful framework for analyzing the geometry of three-dimensional space.

* **Vectors Define Direction and Position:** The concepts of direction cosines, direction ratios, and position vectors allow for the precise algebraic description of orientation and location in 3D space.
* **Lines as Directed Paths:** The equation of a straight line can be elegantly represented in both vector ($\vec{r} = \vec{a} + \lambda\vec{b}$) and Cartesian forms, which are essential for analyzing angles, intersections, and distances.
* **Planes as Flat Surfaces:** Planes are defined by a point and a normal vector, leading to various forms of their equation ($\vec{r}\cdot\vec{n}=d$, $Ax+By+Cz+D=0$). This allows for the analysis of their intersections and relationships with lines and points.
* **Vector Algebra is Key:** Operations like the dot product and cross product are not just abstract tools; they directly calculate geometric quantities like angles ($\cos\theta = \vec{a}\cdot\vec{b} / (|\vec{a}||\vec{b}|)$) and common perpendiculars ($\vec{n}=\vec{b}_1 \times \vec{b}_2$).
* **Unified Geometric Solutions:** This algebraic approach provides systematic methods for solving complex geometric problems, such as finding the shortest distance between skew lines, the image of a point in a plane, or the properties of families of lines and planes.
* **Extension to Surfaces:** The principles extend to surfaces like the sphere, where a simple geometric locus definition translates directly into a manageable algebraic equation.
* **Topology for Qualitative Properties:** Topology moves beyond distance and angle to study the fundamental properties of shape that are preserved under continuous deformation, such as connectedness and the number of holes.
* **Fractal Geometry for Complexity:** Fractals provide a language to describe shapes that exhibit infinite detail and self-similarity at all scales, with the concept of a non-integer fractal dimension capturing their space-filling complexity.

Together, these branches of geometry provide a comprehensive toolkit for analyzing the world around us, from the engineered structures we build to the abstract patterns of nature and the fundamental properties of space itself.