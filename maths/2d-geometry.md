---
title: Coordinate Geometry - Bridging Algebra and Geometry
description: Explore the foundational principles of coordinate geometry, the powerful system that translates geometric shapes into algebraic equations. This post covers the Cartesian system, distance and section formulas, various forms of the straight line, transformations, and key geometric properties of triangles, providing a comprehensive overview of 2D analytical geometry.
date: 2023-01-04
draft: false
slug: /pensieve/maths/coordinate-geometry
tags:
  - Mathematics
  - Coordinate Geometry
---

For centuries, geometry and algebra were studied as largely separate disciplines. Geometry dealt with shapes, lines, and angles, while algebra dealt with equations and variables. The revolutionary idea of coordinate geometry, primarily developed by René Descartes in the 17th century, created a powerful bridge between these two worlds. By representing geometric figures on a coordinate plane, it became possible to analyze shapes using algebraic equations and, conversely, to visualize equations as geometric curves. This exploration delves into the fundamental principles of two-dimensional coordinate geometry, from basic formulas to the intricate properties of lines and triangles.

---
## Mapping the Plane: The Cartesian Coordinate System

The foundation of 2D coordinate geometry is the **Cartesian system of rectangular coordinates**. This system consists of:
* Two perpendicular number lines, called **axes**. The horizontal line is the **x-axis**, and the vertical line is the **y-axis**.
* The point where the axes intersect is called the **origin**, denoted as $(0,0)$.
* The position of any point in the plane is uniquely described by an **ordered pair of coordinates $(x, y)$**. The first value, $x$, is the **abscissa** (horizontal distance from the y-axis), and the second value, $y$, is the **ordinate** (vertical distance from the x-axis).
* The two axes divide the plane into four **quadrants**, numbered I, II, III, and IV, starting from the upper right and moving counter-clockwise.

---
## Measuring and Dividing: The Core Formulas – Distance and Section

With the coordinate system established, we can derive formulas to measure distances and divide line segments.

### Distance Formula
The distance $d$ between two points $P(x_1, y_1)$ and $Q(x_2, y_2)$ in the plane can be found using the Pythagorean theorem.

**Derivation:**
Construct a right-angled triangle with the line segment $PQ$ as the hypotenuse. The horizontal side will have length $|x_2 - x_1|$, and the vertical side will have length $|y_2 - y_1|$.
By the Pythagorean theorem:
$d^2 = (|x_2 - x_1|)^2 + (|y_2 - y_1|)^2$
$d^2 = (x_2 - x_1)^2 + (y_2 - y_1)^2$
Taking the square root gives the **distance formula**:
$$ d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2} $$

### Section Formula
The section formula provides the coordinates of a point that divides a line segment joining two given points in a specified ratio.

* **Internal Division:**
    Let point $P(x,y)$ divide the line segment joining $A(x_1, y_1)$ and $B(x_2, y_2)$ internally in the ratio $m:n$.
    **Derivation:**
    Draw perpendiculars from $A, P, B$ to the x-axis. Using similar triangles, the ratio of horizontal intercepts is equal to the ratio of division:
    $\dfrac{AP}{PB} = \dfrac{m}{n} \implies \dfrac{x - x_1}{x_2 - x} = \dfrac{m}{n}$
    $n(x - x_1) = m(x_2 - x) \implies nx - nx_1 = mx_2 - mx \implies (m+n)x = mx_2 + nx_1$.
    Similarly for the y-coordinates. This gives the coordinates of $P$:
    $$ P(x,y) = \left(\dfrac{mx_2 + nx_1}{m+n}, \dfrac{my_2 + ny_1}{m+n}\right) $$
    A special case is the **midpoint formula**, where $m=n=1$:
    $P_{mid} = \left(\dfrac{x_1+x_2}{2}, \dfrac{y_1+y_2}{2}\right)$.

* **External Division:**
    If point $P(x,y)$ divides the line segment joining $A(x_1, y_1)$ and $B(x_2, y_2)$ externally in the ratio $m:n$, the formula is:
    $$ P(x,y) = \left(\dfrac{mx_2 - nx_1}{m-n}, \dfrac{my_2 - ny_1}{m-n}\right) $$

---
## The Path of a Point: Locus and Its Equation

A **locus** is a set of all points (and only those points) in a plane that satisfy a given geometric condition. The **equation of the locus** is the algebraic relation between the coordinates $(x,y)$ of every point on the path.

For example, the locus of a point that is always equidistant from two fixed points $A(x_1, y_1)$ and $B(x_2, y_2)$ is the perpendicular bisector of the line segment $AB$. Its equation can be found by setting $PA = PB$:
$\sqrt{(x-x_1)^2 + (y-y_1)^2} = \sqrt{(x-x_2)^2 + (y-y_2)^2}$.
Squaring both sides and simplifying yields a linear equation in $x$ and $y$.

---
## Changing Perspectives: Transformation of Coordinates

Sometimes, a geometric problem can be simplified by changing the coordinate system. This can be done by shifting the origin or rotating the axes.

### Shifting of Origin
If the origin $(0,0)$ is shifted to a new point $(h,k)$ without changing the orientation of the axes, the relationship between the old coordinates $(x,y)$ and the new coordinates $(x',y')$ of a point is:
$$ x = x' + h \quad \text{and} \quad y = y' + k $$
or equivalently:
$$ x' = x - h \quad \text{and} \quad y' = y - k $$

### Rotation of Axes
If the coordinate axes are rotated counter-clockwise by an angle $\theta$ about the origin, a point with old coordinates $(x,y)$ will have new coordinates $(x',y')$. The relationship is derived using trigonometry:
$$ x = x'\cos\theta - y'\sin\theta $$
$$ y = x'\sin\theta + y'\cos\theta $$
The inverse transformation (finding $(x',y')$ from $(x,y)$) is:
$$ x' = x\cos\theta + y\sin\theta $$
$$ y' = -x\sin\theta + y\cos\theta $$

---
## The Straight and Narrow: The Line – Fundamentals and Forms

A straight line is one of the most fundamental geometric figures.

* **Slope (or Gradient) of a Line ($m$):**
    The slope measures the steepness of a line. It is the tangent of the angle $\theta$ the line makes with the positive x-axis: $m = \tan\theta$.
    For a line passing through two points $(x_1, y_1)$ and $(x_2, y_2)$:
    $$ m = \dfrac{\text{rise}}{\text{run}} = \dfrac{y_2 - y_1}{x_2 - x_1} $$

* **Conditions for Parallel and Perpendicular Lines:**
    Given two non-vertical lines with slopes $m_1$ and $m_2$:
    * They are **parallel** if and only if their slopes are equal: $m_1 = m_2$.
    * They are **perpendicular** if and only if the product of their slopes is -1: $m_1 m_2 = -1$.

* **Intercepts of a Line:**
    * **x-intercept:** The x-coordinate of the point where the line crosses the x-axis (where $y=0$).
    * **y-intercept:** The y-coordinate of the point where the line crosses the y-axis (where $x=0$).

### Various Forms of Equations of a Line
1.  **Slope-Intercept Form:** $y = mx + c$, where $m$ is the slope and $c$ is the y-intercept.
2.  **Point-Slope Form:** $y - y_1 = m(x - x_1)$, for a line with slope $m$ passing through point $(x_1, y_1)$.
3.  **Two-Point Form:** For a line passing through $(x_1, y_1)$ and $(x_2, y_2)$:
    $$ y - y_1 = \left(\dfrac{y_2 - y_1}{x_2 - x_1}\right)(x - x_1) $$
4.  **Intercept Form:** For a line with x-intercept $a$ and y-intercept $b$:
    $$ \dfrac{x}{a} + \dfrac{y}{b} = 1 $$
5.  **Normal (or Perpendicular) Form:** $x\cos\alpha + y\sin\alpha = p$, where $p$ is the length of the perpendicular from the origin to the line, and $\alpha$ is the angle this perpendicular makes with the positive x-axis.
6.  **General Form:** $Ax + By + C = 0$, where $A, B, C$ are constants. The slope is $m = -A/B$.

---
## Lines in Congress: Intersections, Angles, and More

* **Intersection of Lines:** The point of intersection of two non-parallel lines is found by solving their equations simultaneously.

* **Angle Between Two Lines:** If $\theta$ is the acute angle between two lines with slopes $m_1$ and $m_2$:
    $$ \tan\theta = \left|\dfrac{m_2 - m_1}{1 + m_1 m_2}\right| $$

* **Angle Bisectors:** The locus of points equidistant from two intersecting lines gives the angle bisectors. For lines $A_1x+B_1y+C_1=0$ and $A_2x+B_2y+C_2=0$, the equations of the bisectors are:
    $$ \dfrac{A_1x+B_1y+C_1}{\sqrt{A_1^2+B_1^2}} = \pm \dfrac{A_2x+B_2y+C_2}{\sqrt{A_2^2+B_2^2}} $$

* **Distance of a Point from a Line:** The perpendicular distance $d$ from a point $(x_1, y_1)$ to the line $Ax+By+C=0$ is:
    $$ d = \dfrac{|Ax_1 + By_1 + C|}{\sqrt{A^2 + B^2}} $$

* **Condition for Concurrency of Three Lines:** Three lines $A_1x+B_1y+C_1=0$, $A_2x+B_2y+C_2=0$, and $A_3x+B_3y+C_3=0$ are **concurrent** (intersect at a single point) if the determinant of their coefficients is zero:
    $$ \begin{vmatrix} A_1 & B_1 & C_1 \\ A_2 & B_2 & C_2 \\ A_3 & B_3 & C_3 \end{vmatrix} = 0 $$
    This condition also holds if the lines are mutually parallel, so one must check for non-parallelism as well.

---
## Geometric Measures: Area of Triangles and Polygons

* **Area of a Triangle:**
    Given vertices $A(x_1, y_1), B(x_2, y_2),$ and $C(x_3, y_3)$, the area can be found using the formula:
    $$ \text{Area} = \dfrac{1}{2} |x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)| $$
    This can be expressed concisely using a determinant:
    $$ \text{Area} = \dfrac{1}{2} \left| \det \begin{pmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{pmatrix} \right| $$
    A consequence of this is the **condition for collinearity** of three points: the area of the triangle formed is zero, so the determinant above is zero.

* **Area of a Polygon:**
    The area of any simple polygon with vertices $(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)$ listed in counter-clockwise or clockwise order can be calculated using the **Shoelace Formula** (or Shoelace Algorithm):
    $$ \text{Area} = \dfrac{1}{2} |(x_1y_2 + x_2y_3 + \dots + x_ny_1) - (y_1x_2 + y_2x_3 + \dots + y_nx_1)| $$

---
## Special Points of a Triangle: The Centers of Power

Several special points, or "centers," are associated with any triangle, each with unique geometric properties. Let the vertices of a triangle be $A(x_1, y_1), B(x_2, y_2),$ and $C(x_3, y_3)$, and let the side lengths opposite these vertices be $a, b,$ and $c$ respectively.

* **Centroid ($G$):** The point of intersection of the **medians** (lines from a vertex to the midpoint of the opposite side). It is the triangle's center of mass.
    $$ G = \left(\dfrac{x_1+x_2+x_3}{3}, \dfrac{y_1+y_2+y_3}{3}\right) $$
* **Incentre ($I$):** The point of intersection of the **internal angle bisectors**. It is the center of the inscribed circle (incircle).
    $$ I = \left(\dfrac{ax_1+bx_2+cx_3}{a+b+c}, \dfrac{ay_1+by_2+cy_3}{a+b+c}\right) $$
* **Excentre ($I_a, I_b, I_c$):** A triangle has three excentres, each being the intersection point of one internal angle bisector and two external angle bisectors. They are the centers of the escribed circles. For the excentre $I_a$ opposite vertex A:
    $$ I_a = \left(\dfrac{-ax_1+bx_2+cx_3}{-a+b+c}, \dfrac{-ay_1+by_2+cy_3}{-a+b+c}\right) $$
* **Orthocentre ($O$ or $H$):** The point of intersection of the **altitudes** (perpendiculars from a vertex to the opposite side). Its coordinates are found by solving the system of equations for two altitude lines.
    $$ H = \left(\dfrac{x_1 \sin(2A) + x_2 \sin(2B) + x_3 \sin(2C)}{\sin(2A) + \sin(2B) + \sin(2C)}, \dfrac{y_1 \sin(2A) + y_2 \sin(2B) + y_3 \sin(2C)}{\sin(2A) + \sin(2B) + \sin(2C)}\right) $$
* **Circumcentre ($C$ or $O$):** The point of intersection of the **perpendicular bisectors** of the sides. It is the center of the circumscribed circle (circumcircle) and is equidistant from all three vertices. Its coordinates are found by solving the system of equations for two perpendicular bisector lines.
    $$ C = \left(\dfrac{x_1 \tan(A) + x_2 \tan(B) + x_3 \tan(C)}{\tan(A) + \tan(B) + \tan(C)}, \dfrac{y_1 \tan(A) + y_2 \tan(B) + y_3 \tan(C)}{\tan(A) + \tan(B) + \tan(C)}\right) $$
* **Nine-Point Circle Centre ($N$):** The center of the nine-point circle, which passes through the midpoints of the sides, the feet of the altitudes, and the midpoints of the segments from each vertex to the orthocentre. Its coordinates can be derived from those of the centroid and circumcentre:
    $$ N = \left(\dfrac{x_G + x_C}{2}, \dfrac{y_G + y_C}{2}\right) $$
    where $G$ is the centroid and $C$ is the circumcentre.
* **Notes on Concurrency and Collinearity:**
    * The centroid, orthocentre, and circumcentre are collinear, lying on the **Euler line** of the triangle.
    * The incenter and excentres are also collinear, lying on the **Gergonne line**.
    * The centroid divides the segment joining the orthocentre and circumcentre in a 2:1 ratio.
* **Special Cases:**
    * In an equilateral triangle, all centers coincide at the same point.
    * In an isosceles triangle, the centroid, orthocentre, and circumcentre lie on the line of symmetry.

---
## Lines in a Group: Family of Straight Lines

A **family of lines** is an infinite set of lines that share a common geometric property. A very useful case is the family of lines passing through the intersection point of two given lines $L_1=A_1x+B_1y+C_1=0$ and $L_2=A_2x+B_2y+C_2=0$. The equation for any line in this family can be written as:
$$ L_1 + \lambda L_2 = 0 \quad \text{or} \quad (A_1x+B_1y+C_1) + \lambda(A_2x+B_2y+C_2) = 0 $$
where $\lambda$ is a parameter. A specific value of $\lambda$ corresponds to a unique line passing through the intersection point.

---
## Two Lines in Disguise: Pair of Straight Lines

A single second-degree equation can represent a pair of straight lines.

* **Homogeneous Equation of Second Degree:**
    The equation $ax^2 + 2hxy + by^2 = 0$ always represents a pair of straight lines passing through the origin.
    The separate lines can be found by factoring the expression. If the lines are $y=m_1x$ and $y=m_2x$, then $m_1$ and $m_2$ are the roots of the quadratic equation $bm^2 + 2hm + a = 0$.
    * Sum of slopes: $m_1+m_2 = -2h/b$.
    * Product of slopes: $m_1m_2 = a/b$.
    * **Angle ($\theta$) between the pair of lines:**
        $$ \tan\theta = \dfrac{|2\sqrt{h^2-ab}|}{|a+b|} $$
        Lines are perpendicular if $a+b=0$. Lines are coincident if $h^2-ab=0$.
    * **Equation of the angle bisectors:**
        $$ \dfrac{x^2-y^2}{a-b} = \dfrac{xy}{h} $$

* **General Equation of Second Degree:**
    The equation $ax^2+2hxy+by^2+2gx+2fy+c=0$ represents a pair of straight lines if it satisfies two conditions:
    1.  The discriminant $\Delta = abc + 2fgh - af^2 - bg^2 - ch^2 = 0$.
    2.  $h^2 \ge ab$, $g^2 \ge ac$, and $f^2 \ge bc$.
    * **Point of Intersection:** The intersection point $(x_0, y_0)$ can be found by solving the system of equations derived from partial derivatives: $\dfrac{\partial F}{\partial x} = 2ax+2hy+2g=0$ and $\dfrac{\partial F}{\partial y} = 2hx+2by+2f=0$, where $F(x,y)$ is the general equation.
    * The angle between the lines and the equation of their bisectors are given by the same formulas as for the homogeneous case.

---
## Key Takeaways: The Power of Analytical Geometry

Coordinate geometry provides a robust and systematic framework for analyzing geometric problems through algebraic methods.

* **Bridging Disciplines:** It establishes a direct correspondence between geometric curves and algebraic equations, allowing for the translation of problems from one domain to the other.
* **Fundamental Formulas:** The distance and section formulas provide the basic tools for calculating lengths and locating points on the Cartesian plane.
* **The Straight Line:** A variety of standard forms allow for the equation of a straight line to be determined from different given conditions (e.g., slope, points, intercepts).
* **Geometric Relationships:** Algebraic methods can be used to find angles between lines, distances from points to lines, and conditions for concurrency and collinearity.
* **Triangles and Polygons:** Coordinate geometry provides direct formulas for calculating the area of triangles and polygons and for locating important centers of a triangle.
* **Higher-Order Equations:** Second-degree equations describe not only pairs of straight lines under specific conditions but also the entire family of conic sections, demonstrating the deep connection between algebraic degree and geometric shape.
* **Foundation for Higher Mathematics:** The principles of coordinate geometry are foundational for the study of conic sections, three-dimensional geometry, vector spaces, calculus, and many other advanced mathematical topics.

By representing shapes and relationships with coordinates and equations, this field offers a powerful analytical perspective that is indispensable in science, engineering, computer graphics, and numerous other areas.