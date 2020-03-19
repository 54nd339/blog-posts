---
title: The Circle in Coordinates - Equations, Tangents, Radical Axis
description: One distance condition — a point stays a fixed length from a centre — becomes a second-degree equation, and every geometric question about circles turns into algebra. Equation forms, the point-position test, tangents and normals, chords of contact, and how two circles interact through their radical axis.
date: 2020-03-19
draft: false
slug: /maths/circles
tags:
  - Mathematics
  - Geometry
---

A circle is the simplest curve after a line: the set of points a fixed distance $r$ from a fixed centre $(h, k)$. Feed that one sentence through the distance formula and you get $(x-h)^2 + (y-k)^2 = r^2$, and from there almost every question you can ask — is this point inside? does this line cut the circle or graze it? at what angle do these two circles cross? — has a purely algebraic answer.

This post works through that algebra: the three equation forms and when each is handy, a single expression that tests whether a point is in, on, or out, the tangent and normal in every form you'll need, the chord constructions, and the **radical axis**, the line that governs how two circles relate.

A notation that pays off: for a circle written $x^2 + y^2 + 2gx + 2fy + c = 0$, call the left-hand side $S$. Write $S_1$ for $S$ evaluated at a point $(x_1, y_1)$, and $T$ for the "polarised" form
$$ T \equiv x x_1 + y y_1 + g(x + x_1) + f(y + y_1) + c $$
which is $S$ with $x^2 \to x x_1$, $y^2 \to y y_1$, $2x \to x + x_1$, $2y \to y + y_1$. Tangents, chords of contact, and bisected chords are all one-liners in $S$, $S_1$, $T$.

---
## Three forms of the equation

**Standard (centre–radius).** Straight from $\text{dist}(P, C) = r$, squared:
$$ (x - h)^2 + (y - k)^2 = r^2 $$
Centred at the origin it is just $x^2 + y^2 = r^2$.

**General.** Expand the standard form and rename constants — $g = -h$, $f = -k$, $c = h^2 + k^2 - r^2$:
$$ x^2 + y^2 + 2gx + 2fy + c = 0 $$
Read backwards, the centre is $(-g, -f)$ and the radius $r = \sqrt{g^2 + f^2 - c}$. That radicand decides what you have: positive is a real circle, zero a single point, negative an "imaginary" circle with no real points. Any second-degree equation with equal $x^2$, $y^2$ coefficients and no $xy$ term is a circle in this form.

**Diametric.** Given the ends of a diameter $A(x_1, y_1)$, $B(x_2, y_2)$: any point $P$ on the circle sees the diameter at a right angle (angle in a semicircle), so $PA \perp PB$, so the slopes multiply to $-1$. Clearing denominators:
$$ (x - x_1)(x - x_2) + (y - y_1)(y - y_2) = 0 $$

---
## Where a point sits

Evaluate $S$ at $P(x_1, y_1)$ to get $S_1$. Then

- $S_1 > 0$ — $P$ is **outside**,
- $S_1 = 0$ — $P$ is **on**,
- $S_1 < 0$ — $P$ is **inside**.

Why: the squared distance from the centre $C(-g, -f)$ to $P$ works out to $CP^2 = (x_1 + g)^2 + (y_1 + f)^2 = S_1 + (g^2 + f^2 - c) = S_1 + r^2$. So $S_1 = CP^2 - r^2$, which is positive, zero, or negative exactly as $P$ is farther than, on, or nearer than the radius.

---
## A line meeting a circle

Substitute the line into the circle. Putting $y = mx + c$ into $x^2 + y^2 = r^2$:
$$ (1 + m^2)x^2 + 2mc\,x + (c^2 - r^2) = 0 $$
a quadratic in $x$ whose discriminant $\Delta$ counts the intersections: $\Delta > 0$ two points (**secant**), $\Delta = 0$ one (**tangent**), $\Delta < 0$ none. Equivalently — and usually faster — the line is a tangent exactly when the perpendicular distance from the centre to the line equals $r$.

---
## Tangent at a point on the circle

The radius to the point of contact is perpendicular to the tangent, and that one fact generates every form.

**Point form.** At $(x_1, y_1)$ *on* $S = 0$:
$$ T = 0, \qquad \text{i.e.}\quad x x_1 + y y_1 + g(x + x_1) + f(y + y_1) + c = 0 $$
Derivation: the radius $CP$ has slope $\frac{y_1 + f}{x_1 + g}$, so the tangent has slope $-\frac{x_1 + g}{y_1 + f}$. Point–slope form gives $xx_1 + yy_1 + gx + fy = x_1^2 + y_1^2 + gx_1 + fy_1$. Since $P$ is on the circle, $x_1^2 + y_1^2 = -2gx_1 - 2fy_1 - c$; substitute and the right side collapses to $-gx_1 - fy_1 - c$, which rearranges to $T = 0$.

**Slope form.** Tangents to $x^2 + y^2 = r^2$ with slope $m$: force the distance from the origin to $y = mx + c$ to equal $r$, giving $|c| = r\sqrt{1 + m^2}$, hence
$$ y = mx \pm r\sqrt{1 + m^2} $$
For the general circle, shift by the centre: $(y + f) = m(x + g) \pm r\sqrt{1 + m^2}$ with $r = \sqrt{g^2 + f^2 - c}$.

**Parametric form.** The point $(r\cos\theta, r\sin\theta)$ on $x^2 + y^2 = r^2$ has tangent $x x_1 + y y_1 = r^2$; divide by $r$:
$$ x\cos\theta + y\sin\theta = r $$

---
## Tangent from an external point

**Length.** From an outside point $P(x_1, y_1)$, the contact point $T'$, centre $C$, and $P$ form a right angle at $T'$, so $PT'^2 = CP^2 - r^2 = S_1$:
$$ \text{length of tangent} = \sqrt{S_1} $$

**The pair of tangents** from $P$ — both lines at once — is the combined equation
$$ S\,S_1 = T^2 $$

**Chord of contact.** The line joining the two contact points, for $P(x_1, y_1)$ external, is again
$$ T = 0 $$
identical in form to the point-form tangent — the difference is only whether $(x_1, y_1)$ lies on the circle or outside it.

---
## Normal, and a chord with a given midpoint

The **normal** at a point is perpendicular to the tangent there — and since the radius is already perpendicular to the tangent, *the normal is the radius produced*. It passes through the centre, so its equation is just the line through $(x_1, y_1)$ and $(-g, -f)$:
$$ y - y_1 = \frac{y_1 + f}{x_1 + g}\,(x - x_1) $$

The **chord of $S = 0$ with midpoint $(x_1, y_1)$** (an interior point): the line from the centre to the midpoint bisects the chord, hence is perpendicular to it, which after simplification is
$$ T = S_1 $$

---
## Director circle

The **director circle** is the locus of points from which the two tangents to a given circle are perpendicular. Impose "sum of the $x^2$ and $y^2$ coefficients is zero" on the pair-of-tangents equation and it reduces to
$$ x^2 + y^2 = 2r^2 $$
(concentric, radius $\sqrt{2}\,r$) for $x^2 + y^2 = r^2$, and $(x - h)^2 + (y - k)^2 = 2r^2$ in general. It makes sense from the picture: the contact points and centre form a square of side $r$, whose diagonal $\sqrt{2}\,r$ reaches the viewing point.

---
## Two circles

**Common points.** Subtracting $S_2$ from $S_1$ kills the $x^2$ and $y^2$ terms, leaving a *line* — the radical axis. Solve that against either circle.

**Common tangents**, by comparing the centre distance $d$ with the radii:

| Configuration | Condition | Common tangents |
| --- | --- | --- |
| separate | $d > r_1 + r_2$ | 4 |
| touching externally | $d = r_1 + r_2$ | 3 |
| intersecting | $\lvert r_1 - r_2 \rvert < d < r_1 + r_2$ | 2 |
| touching internally | $d = \lvert r_1 - r_2 \rvert$ | 1 |
| one inside the other | $d < \lvert r_1 - r_2 \rvert$ | 0 |

**Angle of intersection** — the angle between the two tangents at a crossing point, equivalently between the radii there. The triangle of the two centres and a crossing point has sides $r_1, r_2, d$, so the law of cosines gives
$$ \cos\theta = \frac{r_1^2 + r_2^2 - d^2}{2 r_1 r_2} $$
The circles are **orthogonal** ($\theta = 90°$) exactly when $d^2 = r_1^2 + r_2^2$.

---
## Radical axis

The **radical axis** of two circles is the locus of points from which the tangent lengths to both are equal. Setting $\sqrt{S_1^{(1)}} = \sqrt{S_1^{(2)}}$ and squaring gives $S_1 = S_2$, i.e.
$$ S_1 - S_2 = 0 $$
which is linear — the $x^2, y^2$ terms cancel — so it is a straight line, and it is perpendicular to the line of centres. Its role depends on the configuration:

- circles intersect → the radical axis is the line through both crossing points (their **common chord**),
- circles touch → it is the common tangent at the contact point,
- three circles with non-collinear centres → the three pairwise radical axes meet at one point, the **radical centre**.

---
## Families of circles

One parameter sweeps out an infinite set that shares a property:

- through the intersection of a circle $S = 0$ and a line $L = 0$: $\;S + \lambda L = 0$,
- through the intersection of two circles $S_1 = 0$, $S_2 = 0$: $\;S_1 + \lambda S_2 = 0$ (for $\lambda \neq -1$; at $\lambda = -1$ it degenerates to the radical axis).

Each works because any point killing the constituent equations kills the combination for every $\lambda$, so the whole family threads the same fixed points — and picking $\lambda$ to satisfy one extra condition names the specific circle you want.

---
## The one idea to keep

The circle is where "geometry as algebra" is cleanest: a shape defined by one distance, an equation of degree two, and a small dictionary — $S_1$ for point position, $\sqrt{S_1}$ for tangent length, $T = 0$ for tangents and chords of contact, $S_1 - S_2 = 0$ for how two circles relate. Loosen the defining condition from "fixed distance from a point" to "fixed ratio of distances to a point and a line" and the same coordinate machinery produces [the parabola, ellipse, and hyperbola](/citadel/maths/conic-section). All of it rests on the [coordinate plane](/citadel/maths/2d-geometry) and the distance formula.
