---
title: 3D Coordinate Geometry - Lines, Planes, and the Sphere
description: Adding a third axis lets every point in space carry an address and every line, plane, and sphere become an equation. Direction cosines, the vector and Cartesian forms of a line, the shortest distance between skew lines, the four forms of a plane, and the sphere.
date: 2020-03-30
draft: false
slug: /maths/3d-geometry
tags:
  - Mathematics
  - Coordinate Geometry
---

The plane gets you distance, slope, and the conic sections. Add a third axis and the same algebra describes real space — the orientation of a beam, whether two pipes clash, the distance from a point to a wall. The new ingredients are a way to write down a *direction* in three dimensions, and the [dot and cross products](/citadel/maths/vectors) that turn "angle" and "common perpendicular" into formulas.

This post covers 3D coordinate geometry: direction cosines and ratios, the vector and Cartesian equations of a line, the relationships between two lines (including the shortest distance between skew ones), the four forms of a plane and how lines and planes meet, and the sphere. The more exotic geometries — topology and fractals — are [their own post](/citadel/maths/topology-and-fractals).

---
## Space and direction

Three perpendicular axes meet at the origin; the three coordinate planes ($z = 0$, $x = 0$, $y = 0$) cut space into eight **octants**. A point is the triple $(x, y, z)$.

**Direction cosines** $l, m, n$ of a directed line are the cosines of the angles $\alpha, \beta, \gamma$ it makes with the positive axes. For a vector $\vec r$ with components $(x, y, z)$, $x = |\vec r|\cos\alpha$ and so on, and since $|\vec r|^2 = x^2 + y^2 + z^2$,
$$ l^2 + m^2 + n^2 = 1 $$

**Direction ratios** $a, b, c$ are any triple proportional to $(l, m, n)$ — usually easier to read off. Recover the direction cosines by normalising:
$$ l = \pm\frac{a}{\sqrt{a^2 + b^2 + c^2}}, \quad m = \pm\frac{b}{\sqrt{a^2 + b^2 + c^2}}, \quad n = \pm\frac{c}{\sqrt{a^2 + b^2 + c^2}} $$

---
## The straight line

A line is fixed by a point on it and a direction. Through $A$ (position vector $\vec a$), parallel to $\vec b$: a general point $P$ has $\vec{AP} = \lambda\vec b$, so
$$ \vec r = \vec a + \lambda\vec b $$
Splitting into components with $A = (x_1, y_1, z_1)$ and $\vec b = (a, b, c)$ and eliminating $\lambda$ gives the **symmetric Cartesian form**
$$ \frac{x - x_1}{a} = \frac{y - y_1}{b} = \frac{z - z_1}{c} $$
Through **two points** $A, B$: use direction $\vec b - \vec a$, giving $\vec r = \vec a + \lambda(\vec b - \vec a)$, or in Cartesian form $\dfrac{x - x_1}{x_2 - x_1} = \dfrac{y - y_1}{y_2 - y_1} = \dfrac{z - z_1}{z_2 - z_1}$.

---
## Two lines

**Angle** between lines of direction $\vec b_1, \vec b_2$:
$$ \cos\theta = \frac{|\vec b_1 \cdot \vec b_2|}{|\vec b_1|\,|\vec b_2|} = \frac{|a_1 a_2 + b_1 b_2 + c_1 c_2|}{\sqrt{a_1^2 + b_1^2 + c_1^2}\,\sqrt{a_2^2 + b_2^2 + c_2^2}} $$
Perpendicular iff $\vec b_1 \cdot \vec b_2 = 0$; parallel iff $\tfrac{a_1}{a_2} = \tfrac{b_1}{b_2} = \tfrac{c_1}{c_2}$.

In 3D two lines can also be **skew** — neither parallel nor intersecting, missing each other in space.

---
## Distances involving lines

**Point to line.** For the line $\vec r = \vec a + \lambda\vec b$ and external point $\vec p$: the cross product $|(\vec p - \vec a) \times \vec b|$ is the area of the parallelogram on $\vec{AP}$ and $\vec b$, which is also base $\times$ height $= |\vec b|\,d$. So
$$ d = \frac{|(\vec p - \vec a) \times \vec b|}{|\vec b|} $$
The **foot of the perpendicular** $M = \vec a + \lambda\vec b$ is found from $\vec{PM} \cdot \vec b = 0$, and the **image** $P'$ is the reflection of $P$ across $M$.

**Between two skew lines** $\vec r = \vec a_1 + \lambda\vec b_1$ and $\vec r = \vec a_2 + \mu\vec b_2$: project the connector $\vec a_2 - \vec a_1$ onto the common perpendicular direction $\vec b_1 \times \vec b_2$:
$$ d = \frac{|(\vec a_2 - \vec a_1) \cdot (\vec b_1 \times \vec b_2)|}{|\vec b_1 \times \vec b_2|} $$
The lines **intersect** iff this is zero — iff $(\vec a_2 - \vec a_1)$, $\vec b_1$, $\vec b_2$ are coplanar, i.e. their scalar triple product vanishes.

**Between parallel lines** (shared direction $\vec b$): $d = \dfrac{|(\vec a_2 - \vec a_1) \times \vec b|}{|\vec b|}$.

**Worked example.** Line 1: through $(1, 0, 0)$ with direction $\vec b_1 = (1, 1, 0)$. Line 2: through $(0, 0, 1)$ with direction $\vec b_2 = (1, 0, 1)$. Then $\vec b_1 \times \vec b_2 = (1{\cdot}1 - 0{\cdot}0,\ 0{\cdot}1 - 1{\cdot}1,\ 1{\cdot}0 - 1{\cdot}1) = (1, -1, -1)$, with magnitude $\sqrt 3$. The connector is $\vec a_2 - \vec a_1 = (-1, 0, 1)$, and $(-1, 0, 1) \cdot (1, -1, -1) = -1 + 0 - 1 = -2$. So $d = |-2|/\sqrt 3 = 2/\sqrt 3 \approx 1.155$. Non-zero, so the lines are genuinely skew — they pass at closest approach $1.155$ units apart.

---
## The plane

Four ways to pin a plane down.

**Normal form.** With unit normal $\hat n$ and origin distance $p$: every point projects onto $\hat n$ by the same amount,
$$ \vec r \cdot \hat n = p, \qquad \text{Cartesian: } lx + my + nz = p $$

**Point and normal.** Through $A$, normal $\vec n$: the in-plane vector $\vec r - \vec a$ is perpendicular to $\vec n$,
$$ (\vec r - \vec a) \cdot \vec n = 0 \;\Longrightarrow\; Ax + By + Cz + D = 0 $$

**Three non-collinear points.** The normal is $(\vec b - \vec a) \times (\vec c - \vec a)$, so
$$ (\vec r - \vec a) \cdot \big[(\vec b - \vec a) \times (\vec c - \vec a)\big] = 0 $$
— the four points are coplanar iff this scalar triple product is zero.

**Intercept form.** Cutting the axes at $a, b, c$:
$$ \frac{x}{a} + \frac{y}{b} + \frac{z}{c} = 1 $$

**Family through an intersection.** Every plane containing the line where $P_1 = 0$ meets $P_2 = 0$ is $P_1 + \lambda P_2 = 0$.

---
## Lines and planes together

**Angle between two planes** — the angle between their normals: $\cos\theta = \dfrac{|\vec n_1 \cdot \vec n_2|}{|\vec n_1|\,|\vec n_2|}$. Perpendicular iff $\vec n_1 \cdot \vec n_2 = 0$, parallel iff the normals are proportional.

**Line of intersection** of two planes runs perpendicular to both normals, so its direction is $\vec n_1 \times \vec n_2$; get a point on it by fixing one coordinate (say $z = 0$) and solving the two plane equations.

**Angle between a line and a plane** — the line makes angle $\theta$ with the *normal*, so with the plane it makes $\phi = 90° - \theta$:
$$ \sin\phi = \frac{|\vec b \cdot \vec n|}{|\vec b|\,|\vec n|} $$

**Distances.** Point $\vec p$ to plane $\vec r \cdot \vec n = d$: $\;D = \dfrac{|\vec p \cdot \vec n - d|}{|\vec n|}$, or in Cartesian form $\dfrac{|Ax_0 + By_0 + Cz_0 + D|}{\sqrt{A^2 + B^2 + C^2}}$. Between parallel planes $Ax + By + Cz + D_i = 0$: $\;\dfrac{|D_1 - D_2|}{\sqrt{A^2 + B^2 + C^2}}$. The **image** of a point is its reflection across the foot of the perpendicular.

---
## The sphere

The locus of points a fixed distance $r$ from a centre $(h, k, l)$:
$$ (x - h)^2 + (y - k)^2 + (z - l)^2 = r^2 $$
Expanded, the **general form** is
$$ x^2 + y^2 + z^2 + 2ux + 2vy + 2wz + d = 0 $$
with centre $(-u, -v, -w)$ and radius $\sqrt{u^2 + v^2 + w^2 - d}$.

**Diameter form.** With the segment from $A$ to $B$ as a diameter, any point $P$ on the sphere sees it at a right angle, so $\vec{PA} \cdot \vec{PB} = 0$:
$$ (x - x_1)(x - x_2) + (y - y_1)(y - y_2) + (z - z_1)(z - z_2) = 0 $$

---
## The one idea to keep

The whole subject is the plane's coordinate geometry with one extra axis and the vector products doing the geometric work — dot for angles and perpendicular distances, cross for the direction of a common perpendicular or a plane's normal, scalar triple product for coplanarity and skew-line distance. It is the setting for [vector calculus](/citadel/maths/vector-calculus) and the classical geometry that [topology](/citadel/maths/topology-and-fractals) then abstracts away from.
