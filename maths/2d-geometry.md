---
title: Coordinate Geometry - Turning Shapes into Equations
description: Pin two number lines to the plane and every geometric question becomes an algebra question. Distance and section formulas, the six forms of a line, angles and distances, triangle centres, and how one second-degree equation hides a pair of lines.
date: 2020-03-18
draft: false
slug: /maths/2d-geometry
tags:
  - Mathematics
  - Coordinate Geometry
---

"Is the point $(4, 1)$ on the line through $(0, -1)$ and $(2, 0)$?" You could draw it and squint, or you could check whether $x - 2y = 2$ holds when $x = 4$, $y = 1$. It does. Descartes' move, around 1637, was to make that second option always available: pin two perpendicular number lines to the plane, and every point gets an address $(x, y)$, every curve gets an equation, and every geometric question — distance, intersection, "are these three points in a line?" — becomes arithmetic you can just do.

This post is the 2D toolkit: how to measure and divide with coordinates, the standard forms of a straight line and when each is the natural one to reach for, the formulas for angles and perpendicular distances, the five classical centres of a triangle, and the fact that a single quadratic in $x$ and $y$ can secretly be two lines.

---
## The Cartesian plane

Two perpendicular number lines — the **$x$-axis** horizontal, the **$y$-axis** vertical — cross at the **origin** $(0,0)$. A point's address is the ordered pair $(x, y)$: $x$ (the **abscissa**) is its signed distance from the $y$-axis, $y$ (the **ordinate**) its signed distance from the $x$-axis. The axes cut the plane into four **quadrants**, numbered I–IV anticlockwise from the top right, where the sign pattern of $(x, y)$ runs $(+,+)$, $(-,+)$, $(-,-)$, $(+,-)$.

---
## Distance and section

**Distance.** Between $P(x_1, y_1)$ and $Q(x_2, y_2)$, take $PQ$ as the hypotenuse of a right triangle with horizontal leg $|x_2 - x_1|$ and vertical leg $|y_2 - y_1|$. Pythagoras gives
$$ d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2} $$

**Section.** The point dividing segment $AB$ from $A(x_1,y_1)$ to $B(x_2,y_2)$ in ratio $m : n$. For **internal** division, drop perpendiculars from $A$, $P$, $B$ to the $x$-axis; similar triangles give $\frac{x - x_1}{x_2 - x} = \frac{m}{n}$, which rearranges to $(m+n)x = m x_2 + n x_1$, and likewise for $y$:
$$ P = \left(\frac{m x_2 + n x_1}{m + n},\; \frac{m y_2 + n y_1}{m + n}\right) $$
Setting $m = n$ gives the **midpoint** $\big(\tfrac{x_1 + x_2}{2}, \tfrac{y_1 + y_2}{2}\big)$. **External** division (the point lies on the line $AB$ but outside the segment) flips a sign:
$$ P = \left(\frac{m x_2 - n x_1}{m - n},\; \frac{m y_2 - n y_1}{m - n}\right) $$

---
## Locus

A **locus** is the set of all points satisfying a stated condition, and its **equation** is the algebraic relation every such $(x, y)$ obeys. Translating the condition into coordinates *is* the method. "Equidistant from $A$ and $B$" means $PA = PB$; square both sides of $\sqrt{(x-x_1)^2 + (y-y_1)^2} = \sqrt{(x-x_2)^2 + (y-y_2)^2}$, the quadratic terms cancel, and what survives is a linear equation — the perpendicular bisector of $AB$.

---
## Changing coordinates

A hard problem often turns easy in better-placed axes.

**Shift of origin** to $(h, k)$, axes still parallel:
$$ x = x' + h, \qquad y = y' + k $$

**Rotation of axes** by angle $\theta$ anticlockwise about the origin:
$$ x = x'\cos\theta - y'\sin\theta, \qquad y = x'\sin\theta + y'\cos\theta $$
with inverse $x' = x\cos\theta + y\sin\theta$, $y' = -x\sin\theta + y\cos\theta$. Rotation is what removes the $xy$ cross-term from a conic's equation.

---
## The straight line

The **slope** $m = \tan\theta$, where $\theta$ is the angle the line makes with the positive $x$-axis; through two points it is $\dfrac{y_2 - y_1}{x_2 - x_1}$ ("rise over run"). Two non-vertical lines are **parallel** iff $m_1 = m_2$ and **perpendicular** iff $m_1 m_2 = -1$ (a quarter-turn sends slope $m$ to $-1/m$).

Six forms, each suited to a different set of givens:

| Form | Equation | Use when you know |
| --- | --- | --- |
| Slope–intercept | $y = mx + c$ | slope and $y$-intercept $c$ |
| Point–slope | $y - y_1 = m(x - x_1)$ | slope and one point |
| Two-point | $y - y_1 = \frac{y_2 - y_1}{x_2 - x_1}(x - x_1)$ | two points |
| Intercept | $\frac{x}{a} + \frac{y}{b} = 1$ | both axis intercepts $a$, $b$ |
| Normal | $x\cos\alpha + y\sin\alpha = p$ | perpendicular distance $p$ from origin, at angle $\alpha$ |
| General | $Ax + By + C = 0$ | anything ($m = -A/B$) |

---
## Lines meeting lines

**Intersection**: solve the two equations simultaneously.

**Angle** between lines of slope $m_1, m_2$:
$$ \tan\theta = \left|\frac{m_2 - m_1}{1 + m_1 m_2}\right| $$
(the $1 + m_1 m_2$ in the denominator vanishes exactly when they are perpendicular).

**Perpendicular distance** from $(x_1, y_1)$ to $Ax + By + C = 0$:
$$ d = \frac{|A x_1 + B y_1 + C|}{\sqrt{A^2 + B^2}} $$

**Angle bisectors** of two lines are the locus of points equidistant from both, so from the distance formula:
$$ \frac{A_1 x + B_1 y + C_1}{\sqrt{A_1^2 + B_1^2}} = \pm\, \frac{A_2 x + B_2 y + C_2}{\sqrt{A_2^2 + B_2^2}} $$

**Concurrency** of three lines: the coefficient determinant $\begin{vmatrix} A_1 & B_1 & C_1 \\ A_2 & B_2 & C_2 \\ A_3 & B_3 & C_3 \end{vmatrix} = 0$ — necessary but not sufficient, since it also vanishes when the lines are all parallel, so check that separately.

---
## Area, and collinearity as a special case

For a triangle with vertices $A(x_1,y_1)$, $B(x_2,y_2)$, $C(x_3,y_3)$:
$$ \text{Area} = \tfrac{1}{2}\,\big|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)\big| = \tfrac{1}{2}\left|\det\!\begin{pmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{pmatrix}\right| $$
Set the area to zero and you have the test for **three points collinear** — no triangle means they fall on one line.

The same idea scales to any simple polygon with vertices listed in order, via the **shoelace formula**:
$$ \text{Area} = \tfrac{1}{2}\,\big|(x_1 y_2 + x_2 y_3 + \cdots + x_n y_1) - (y_1 x_2 + y_2 x_3 + \cdots + y_n x_1)\big| $$

---
## The centres of a triangle

Take vertices $A(x_1,y_1)$, $B(x_2,y_2)$, $C(x_3,y_3)$ with opposite side lengths $a, b, c$ and angles $A, B, C$.

- **Centroid** $G$ — medians meet here; it is the centre of mass:
  $$ G = \left(\frac{x_1 + x_2 + x_3}{3},\; \frac{y_1 + y_2 + y_3}{3}\right) $$
- **Incentre** $I$ — internal angle bisectors meet here; centre of the inscribed circle. It is the side-length-weighted average of the vertices:
  $$ I = \left(\frac{a x_1 + b x_2 + c x_3}{a + b + c},\; \frac{a y_1 + b y_2 + c y_3}{a + b + c}\right) $$
- **Excentre** $I_a$ — one internal and two external bisectors; centre of the escribed circle opposite $A$. Flip the sign of $a$: $I_a = \big(\frac{-a x_1 + b x_2 + c x_3}{-a + b + c},\, \frac{-a y_1 + b y_2 + c y_3}{-a + b + c}\big)$, and similarly for $I_b, I_c$.
- **Orthocentre** $H$ — altitudes meet here; a weighted average with weights $\tan A, \tan B, \tan C$.
- **Circumcentre** $O$ — perpendicular bisectors meet here; equidistant from all three vertices; weights $\sin 2A, \sin 2B, \sin 2C$.

$G$, $H$, and $O$ are collinear on the **Euler line**, with $G$ dividing $HO$ in ratio $2 : 1$. Their midpoint $N = \tfrac12(H + O)$ is the **nine-point centre** — the centre of the circle through the three side-midpoints, the three altitude feet, and the three midpoints of the segments from $H$ to each vertex. In an equilateral triangle all of these centres collapse to one point.

---
## Family of lines

Every line through the intersection of $L_1 = 0$ and $L_2 = 0$ can be written
$$ L_1 + \lambda L_2 = 0 $$
for a real parameter $\lambda$ — because any $(x, y)$ that kills both $L_1$ and $L_2$ kills the combination for every $\lambda$, and varying $\lambda$ sweeps out the whole pencil. It is the quick way to name "the line through that intersection *and* one more condition" without ever computing the intersection point.

---
## One equation, two lines

A second-degree equation can factor into two linear ones. The **homogeneous** case
$$ ax^2 + 2hxy + by^2 = 0 $$
always represents two lines through the origin, $y = m_1 x$ and $y = m_2 x$, where $m_1, m_2$ solve $bm^2 + 2hm + a = 0$. So $m_1 + m_2 = -2h/b$, $m_1 m_2 = a/b$, and the angle between them is
$$ \tan\theta = \frac{2\sqrt{h^2 - ab}}{|a + b|} $$
The pair is **perpendicular** when $a + b = 0$, **coincident** when $h^2 = ab$. Their bisectors satisfy $\dfrac{x^2 - y^2}{a - b} = \dfrac{xy}{h}$.

The **general** second-degree equation $ax^2 + 2hxy + by^2 + 2gx + 2fy + c = 0$ splits into two lines iff
$$ \Delta = abc + 2fgh - af^2 - bg^2 - ch^2 = 0 \quad\text{and}\quad h^2 \ge ab $$
Their intersection $(x_0, y_0)$ solves the pair $\partial F/\partial x = 0$, $\partial F/\partial y = 0$; the angle and bisector formulas carry over unchanged from the homogeneous case. When $\Delta \neq 0$ the same equation is instead an ellipse, parabola, or hyperbola — degree two, one algebraic family, [every conic section](/citadel/maths/conic-section).

---
## The one idea to keep

Coordinate geometry is a dictionary: "distance" ↔ a square root, "collinear" ↔ a determinant that vanishes, "perpendicular" ↔ a product that equals $-1$. Once a shape is an equation you can differentiate it, intersect it with another, or hand it to a computer. The first curve worth taking through this machinery is the [circle](/citadel/maths/circles); after that, the same coordinates carry straight into [vectors](/citadel/maths/vectors) and into [three dimensions](/citadel/maths/3d-geometry).
