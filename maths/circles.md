---
title: Coordinate Geometry of the Circle - Equations, Tangents, and Intersections
description: Delve into the coordinate geometry of the circle, from its fundamental equations to advanced properties. This post explores tangents, normals, chords, families of circles, and the radical axis, providing the algebraic framework for understanding one of geometry's most perfect shapes.
date: 2023-01-05
draft: false
slug: /pensieve/maths/circles
tags:
  - Mathematics
  - Geometry
---

The circle is arguably one of the most fundamental and perfect shapes in geometry. Its perfect symmetry and simple definition give rise to a rich and elegant set of properties that can be powerfully explored using the tools of coordinate geometry. By placing the circle onto the Cartesian plane, we can translate its geometric features into algebraic equations, allowing for rigorous analysis and problem-solving. This exploration covers the essential aspects of the circle, from its various equational forms to its relationships with points, lines, and other circles.

---
## Defining Perfection: The Equations of a Circle

The fundamental definition of a circle is geometric:

> A **circle** is the locus of a point that moves in a plane such that its distance from a fixed point (the center) is always constant. This constant distance is called the radius.

This definition serves as the starting point for deriving its algebraic equations.

### Different Forms of the Equation of a Circle

1.  **Standard Form (Center-Radius Form):**
    This is the most direct translation of the definition. Let the center of the circle be $C(h,k)$ and the radius be $r$. Let $P(x,y)$ be any point on the circle. The distance between $P$ and $C$ must be equal to $r$.
    * **Derivation:** Using the distance formula, we have:
        $\sqrt{(x-h)^2 + (y-k)^2} = r$.
        Squaring both sides gives the standard equation of a circle:
        $$ (x-h)^2 + (y-k)^2 = r^2 $$
    * **Circle with Center at Origin:** A special case where $(h,k) = (0,0)$. The equation simplifies to:
        $$ x^2 + y^2 = r^2 $$

2.  **General Form:**
    The general form of the equation of a circle is a second-degree equation in $x$ and $y$.
    * **Derivation:** By expanding the standard form $(x-h)^2 + (y-k)^2 = r^2$:
        $(x^2 - 2hx + h^2) + (y^2 - 2ky + k^2) = r^2$
        $x^2 + y^2 - 2hx - 2ky + (h^2 + k^2 - r^2) = 0$.
        Letting $g = -h$, $f = -k$, and $c = h^2+k^2-r^2$, we get the general form:
        $$ x^2 + y^2 + 2gx + 2fy + c = 0 $$
    * **Finding Center and Radius from General Form:**
        Comparing the general form with the expanded standard form, we can identify:
        * Center: $(-g, -f)$
        * Radius: $r = \sqrt{g^2 + f^2 - c}$
        For a real circle to exist, the radius must be real and positive, so we require the condition $g^2 + f^2 - c > 0$. If it is zero, the circle is a point circle. If it is negative, the circle is imaginary.

3.  **Diametric Form:**
    The equation of a circle where the endpoints of a diameter are given as $A(x_1, y_1)$ and $B(x_2, y_2)$.
    * **Derivation:** Let $P(x,y)$ be any point on the circle. The angle in a semicircle is a right angle, so $\angle APB = 90^\circ$. This means the line segment $PA$ is perpendicular to the line segment $PB$.
        The slope of $PA$ is $m_1 = \dfrac{y-y_1}{x-x_1}$.
        The slope of $PB$ is $m_2 = \dfrac{y-y_2}{x-x_2}$.
        For perpendicular lines, $m_1 m_2 = -1$.
        $\left(\dfrac{y-y_1}{x-x_1}\right) \left(\dfrac{y-y_2}{x-x_2}\right) = -1$
        $(y-y_1)(y-y_2) = -(x-x_1)(x-x_2)$
        $$ (x-x_1)(x-x_2) + (y-y_1)(y-y_2) = 0 $$

---
## Inside, Outside, or On the Edge? Position of a Point with Respect to a Circle

Let the equation of a circle be given by $S \equiv x^2 + y^2 + 2gx + 2fy + c = 0$. To determine the position of a point $P(x_1, y_1)$ relative to this circle, we can evaluate the expression $S$ at the point's coordinates.
Let $S_1 = x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c$.
* If $S_1 > 0$, the point $P(x_1, y_1)$ lies **outside** the circle.
* If $S_1 = 0$, the point $P(x_1, y_1)$ lies **on** the circle.
* If $S_1 < 0$, the point $P(x_1, y_1)$ lies **inside** the circle.

**Derivation:** The center of the circle is $C(-g, -f)$ and its radius is $r = \sqrt{g^2+f^2-c}$. Let's compare the squared distance from the center to the point, $CP^2$, with the squared radius, $r^2$.
$CP^2 = (x_1 - (-g))^2 + (y_1 - (-f))^2 = (x_1+g)^2 + (y_1+f)^2 = x_1^2 + 2gx_1 + g^2 + y_1^2 + 2fy_1 + f^2$.
$CP^2 = (x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c) + g^2 + f^2 - c = S_1 + r^2$.
So, $S_1 = CP^2 - r^2$.
* If $P$ is outside, $CP > r \implies CP^2 > r^2 \implies S_1 > 0$.
* If $P$ is on, $CP = r \implies CP^2 = r^2 \implies S_1 = 0$.
* If $P$ is inside, $CP < r \implies CP^2 < r^2 \implies S_1 < 0$.

---
## Lines and Circles: Intersections and Tangency

The intersection of a line and a circle can result in two points (secant), one point (tangent), or no points.
**Method:** To find the points of intersection between a line (e.g., $y=mx+c$) and a circle (e.g., $x^2+y^2=r^2$), we solve their equations simultaneously.
Substitute $y=mx+c$ into the circle's equation:
$x^2 + (mx+c)^2 = r^2 \implies x^2 + m^2x^2 + 2mcx + c^2 = r^2$.
$(1+m^2)x^2 + (2mc)x + (c^2-r^2) = 0$.
This is a quadratic equation in $x$. The nature of its roots (and thus the intersection) depends on its discriminant, $\Delta = B^2 - 4AC$.
* If $\Delta > 0$: Two distinct real roots for $x$. The line is a **secant** and intersects the circle at two distinct points.
* If $\Delta = 0$: One real repeated root for $x$. The line is a **tangent** and touches the circle at one point.
* If $\Delta < 0$: No real roots for $x$. The line does not intersect the circle.

Alternatively, the condition for tangency can be found by setting the perpendicular distance from the center of the circle to the line equal to the radius.

---
## The Gentle Touch: Tangent to a Circle at a Given Point

A **tangent** is a line that touches a circle at exactly one point, the **point of contact**.

### Forms of the Equation of a Tangent

1.  **Point Form:** The equation of the tangent to the circle $x^2+y^2+2gx+2fy+c=0$ at the point $P(x_1, y_1)$ which lies on the circle is:
    $$ xx_1 + yy_1 + g(x+x_1) + f(y+y_1) + c = 0 $$
    This is often remembered by the substitution rule: $x^2 \to xx_1$, $y^2 \to yy_1$, $2x \to (x+x_1)$, $2y \to (y+y_1)$.
    **Derivation:** The radius from the center $C(-g,-f)$ to the point of contact $P(x_1,y_1)$ is perpendicular to the tangent line.
    The slope of the radius $CP$ is $m_{radius} = \dfrac{y_1 - (-f)}{x_1 - (-g)} = \dfrac{y_1+f}{x_1+g}$.
    The slope of the tangent line is $m_{tangent} = -1/m_{radius} = -\dfrac{x_1+g}{y_1+f}$.
    Using the point-slope form $y-y_1 = m_{tangent}(x-x_1)$:
    $y-y_1 = -\dfrac{x_1+g}{y_1+f}(x-x_1)$.
    $(y-y_1)(y_1+f) = -(x-x_1)(x_1+g)$.
    $yy_1 + yf - y_1^2 - y_1f = -xx_1 - gx + x_1^2 + gx_1$.
    $xx_1 + yy_1 + gx + fy = x_1^2 + y_1^2 + gx_1 + fy_1$.
    Since $(x_1,y_1)$ lies on the circle, $x_1^2+y_1^2+2gx_1+2fy_1+c=0$.
    From this, $x_1^2+y_1^2 = -2gx_1 - 2fy_1 - c$.
    Substitute this into the right side:
    $xx_1 + yy_1 + gx + fy = (-2gx_1 - 2fy_1 - c) + gx_1 + fy_1 = -gx_1 - fy_1 - c$.
    Rearranging gives the final form: $xx_1+yy_1+g(x+x_1)+f(y+y_1)+c=0$.

2.  **Slope Form:** The equations of the two tangents to the circle $x^2+y^2=r^2$ with a given slope $m$ are:
    $$ y = mx \pm r\sqrt{1+m^2} $$
    **Derivation:** The perpendicular distance from the center $(0,0)$ to the line $y=mx+c$ (or $mx-y+c=0$) must be equal to the radius $r$ for the line to be a tangent.
    Distance $d = \dfrac{|m(0) - (0) + c|}{\sqrt{m^2+(-1)^2}} = \dfrac{|c|}{\sqrt{m^2+1}}$.
    Setting $d=r$: $r = \dfrac{|c|}{\sqrt{m^2+1}} \implies |c| = r\sqrt{m^2+1} \implies c = \pm r\sqrt{1+m^2}$.
    Substituting $c$ back into $y=mx+c$ gives the two tangent equations.

    **General Form:** For a circle $x^2+y^2+2gx+2fy+c=0$, the equations of the tangents with slope $m$ are:
    $$ (y+f) = m(x+g) \pm r\sqrt{1+m^2} $$
    where $r = \sqrt{g^2 + f^2 - c}$ is the radius.

3.  **Parametric Form:** The equation of the tangent to the circle $x^2+y^2=r^2$ at the point $(r\cos\theta, r\sin\theta)$ on the circle is:
    $$ x\cos\theta + y\sin\theta = r $$
    **Derivation:** This is found by substituting $x_1 = r\cos\theta$ and $y_1 = r\sin\theta$ into the point form tangent equation for this circle, which is $xx_1+yy_1=r^2$.
    $x(r\cos\theta) + y(r\sin\theta) = r^2$. Dividing by $r$ gives the result.

---
## Reaching Out: Tangent from an External Point

* **Length of Tangent:** The length of the tangent from an external point $P(x_1, y_1)$ to the circle $S \equiv x^2+y^2+2gx+2fy+c=0$ is given by $\sqrt{S_1}$.
    **Derivation:** Let $C(-g,-f)$ be the center and $T$ be the point of contact. $\triangle CTP$ is a right-angled triangle with the right angle at $T$. The length of the tangent is $PT$.
    By Pythagoras' theorem, $CP^2 = CT^2 + PT^2$.
    $CP^2 = (x_1+g)^2 + (y_1+f)^2 = x_1^2+y_1^2+2gx_1+2fy_1+g^2+f^2$.
    $CT^2 = r^2 = g^2+f^2-c$.
    So, $PT^2 = CP^2 - CT^2 = (x_1^2+y_1^2+2gx_1+2fy_1+g^2+f^2) - (g^2+f^2-c) = x_1^2+y_1^2+2gx_1+2fy_1+c = S_1$.
    Thus, length $PT = \sqrt{S_1}$.

* **Pair of Tangents:** The combined equation of the pair of tangents drawn from an external point $P(x_1, y_1)$ to the circle $S=0$ is given by the relation:
    $$ S S_1 = T^2 $$
    where $S=x^2+y^2+2gx+2fy+c$, $S_1=x_1^2+y_1^2+2gx_1+2fy_1+c$, and $T=xx_1+yy_1+g(x+x_1)+f(y+y_1)+c$.

---
## The Perpendicular Line: Normal to a Circle at a Given Point

The **normal** to a circle at a given point is the line perpendicular to the tangent at that same point.
A key property is that **the normal at any point on a circle always passes through the center of the circle.**
Therefore, the equation of the normal to a circle at point $(x_1,y_1)$ on it is simply the equation of the line passing through $(x_1,y_1)$ and the center of the circle $(-g,-f)$.

* **Equation of the Normal:** The equation of the normal to the circle $S=0$ at the point $P(x_1,y_1)$ is given by:
    $$ y - y_1 = \dfrac{y_1 + f}{x_1 + g} (x - x_1) $$
    This is derived from the slope of the radius (which is also the slope of the normal) and using the point-slope form of a line.

---
## Lines from Afar: Chords and Circles

* **Chord of Contact:** If two tangents are drawn from an external point $P(x_1, y_1)$ to a circle, the line segment joining their points of contact is called the **chord of contact**.
    * **Equation:** The equation of the chord of contact from $P(x_1, y_1)$ to the circle $S=0$ is given by $T=0$.
        $$ xx_1 + yy_1 + g(x+x_1) + f(y+y_1) + c = 0 $$
        Note that this has the same form as the equation of the tangent at a point on the circle, but here $(x_1,y_1)$ is an external point.

* **Equation of the Chord Bisected at a Given Point:**
    The equation of the chord of the circle $S=0$ whose midpoint is $(x_1, y_1)$ (an internal point).
    * **Equation:** The equation is given by the relation $T=S_1$.
    * **Derivation:** The line joining the center $C(-g,-f)$ to the midpoint $M(x_1,y_1)$ is perpendicular to the chord. The slope of $CM$ is $m_{CM} = (y_1+f)/(x_1+g)$. The slope of the chord is $m_{chord} = -1/m_{CM} = -(x_1+g)/(y_1+f)$. Using point-slope form for the chord passing through $M(x_1,y_1)$:
        $y-y_1 = -\dfrac{x_1+g}{y_1+f}(x-x_1)$.
        After algebraic manipulation, this can be shown to be equivalent to $T=S_1$.

---
## The Locus of Right Angles: Director Circle

The **director circle** is the locus of a point from which the two tangents drawn to a given circle are perpendicular to each other.
* **Equation:** For a circle $x^2+y^2=r^2$, the director circle is $x^2+y^2=2r^2$.
    For a circle $(x-h)^2+(y-k)^2=r^2$, the director circle is $(x-h)^2+(y-k)^2=2r^2$.
    It is a concentric circle with radius $\sqrt{2}$ times the original radius.
* **Derivation:** The pair of tangents from $(h,k)$ to $x^2+y^2=r^2$ are given by $(x^2+y^2-r^2)(h^2+k^2-r^2)=(xh+yk-r^2)^2$. For these lines to be perpendicular, the sum of the coefficients of $x^2$ and $y^2$ must be zero. Expanding and collecting terms for $x^2$ and $y^2$ gives this condition, which simplifies to $h^2+k^2=2r^2$. Replacing $(h,k)$ with a general point $(x,y)$ gives the locus.

---
## When Circles Meet: Intersections, Tangents, and Angles

* **Intersection of Two Circles:** To find the intersection points of two circles, $S_1=0$ and $S_2=0$, one solves their equations simultaneously. Subtracting one equation from the other ($S_1-S_2=0$) eliminates the $x^2$ and $y^2$ terms, yielding the linear equation of the **radical axis**. Solving this linear equation with either of the circle equations gives the intersection points.
* **Common Tangents:** Depending on the relative positions of two circles (determined by comparing the distance $d$ between their centers with their radii $r_1, r_2$), they can have:
    * 4 common tangents if $d > r_1+r_2$ (circles are separate).
    * 3 common tangents if $d = r_1+r_2$ (touching externally).
    * 2 common tangents if $|r_1-r_2| < d < r_1+r_2$ (intersecting).
    * 1 common tangent if $d = |r_1-r_2|$ (touching internally).
    * 0 common tangents if $d < |r_1-r_2|$ (one inside another).
* **Angle of Intersection:** This is the angle between the tangents to the two circles at a point of intersection.
    * **Formula:** If $\theta$ is the angle of intersection, $d$ is the distance between centers, and $r_1, r_2$ are the radii:
        $$ \cos\theta = \dfrac{r_1^2 + r_2^2 - d^2}{2r_1r_2} $$
    * **Derivation:** This comes from the Law of Cosines applied to the triangle formed by the centers of the two circles and one of their intersection points. The sides of this triangle are $r_1, r_2,$ and $d$. The angle between the radii at the intersection point is $\theta'$ (or $180^\circ-\theta'$), and the angle between the tangents is the supplement of this, leading to the formula.
    * **Orthogonal Intersection:** If the circles intersect at a right angle ($\theta=90^\circ$), they are orthogonal. Then $\cos\theta=0$, which gives the condition: $d^2 = r_1^2 + r_2^2$.

---
## The Power Line: Radical Axis – Equal Tangents

* **Radical Axis:** The locus of points from which the lengths of the tangents drawn to two circles are equal.
    * **Equation:** For two circles $S_1 \equiv x^2+y^2+2g_1x+2f_1y+c_1=0$ and $S_2 \equiv x^2+y^2+2g_2x+2f_2y+c_2=0$, the equation of the radical axis is given by:
        $$ S_1 - S_2 = 0 $$
        $ (2g_1-2g_2)x + (2f_1-2f_2)y + (c_1-c_2) = 0 $, which is the equation of a straight line.
    * **Derivation:** Let $P(h,k)$ be a point on the locus. Length of tangent from $P$ to $S_1$ is $\sqrt{h^2+k^2+2g_1h+2f_1k+c_1}$. Length of tangent from $P$ to $S_2$ is $\sqrt{h^2+k^2+2g_2h+2f_2k+c_2}$. Equating the squares of these lengths and replacing $(h,k)$ with $(x,y)$ gives the equation $S_1=S_2$.
* **Properties:**
    * The radical axis is a straight line.
    * It is perpendicular to the line joining the centers of the circles.
    * If two circles intersect, their radical axis is the line containing their **common chord**.
    * If two circles touch, their radical axis is their common tangent at the point of contact.
    * The radical axes of three circles (with non-collinear centers), taken in pairs, are concurrent at a point called the **radical center**.

---
## Circles in a Group: Family of Circles

A **family of circles** is a set of circles that share a common geometric property. Their equations can be represented by a single equation with one or more parameters.
* **Family of circles passing through the intersection of a circle $S=0$ and a line $L=0$:**
    $$ S + \lambda L = 0 $$
    where $\lambda$ is a parameter. For any $\lambda$, this equation represents a circle passing through the two intersection points.
* **Family of circles passing through the intersection of two circles $S_1=0$ and $S_2=0$:**
    $$ S_1 + \lambda S_2 = 0 \quad (\text{for } \lambda \neq -1) $$
    For $\lambda=-1$, the equation becomes $S_1-S_2=0$, which is the equation of the radical axis (common chord).

---
## Key Takeaways: The Algebraic Essence of the Circle

The study of the circle through coordinate geometry provides a powerful fusion of algebraic manipulation and geometric intuition.

* **Equations as Descriptors:** The circle can be precisely described by various algebraic equations—standard, general, and diametric forms—each useful in different contexts.
* **Algebraic Conditions for Geometric Properties:** The position of a point (inside, on, or outside), the nature of the intersection of a line and a circle (secant, tangent, or none), and the relationship between two circles can all be determined algebraically.
* **Tangents and Normals:** The equations of tangents and normals to a circle can be systematically derived in various forms (point, slope, parametric), forming the basis for many geometric constructions and problems.
* **Loci and Chords:** Concepts like the chord of contact, chords bisected at a point, and the director circle are elegantly described as loci with specific equations derived from geometric properties.
* **Interacting Circles and the Radical Axis:** The relationship between two circles is encapsulated by their radical axis ($S_1-S_2=0$), which serves as their common chord if they intersect and provides a powerful tool for analyzing systems of circles.
* **Families of Circles:** Algebraic equations with parameters (e.g., $S+\lambda L=0$) allow for the study of infinite sets of circles that share common properties, such as passing through the same two points.

The coordinate geometry of the circle is a foundational topic that perfectly illustrates the interplay between algebra and geometry, providing essential tools used throughout mathematics and applied sciences.