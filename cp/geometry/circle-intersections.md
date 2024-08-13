---
title: Circle Intersections - Lines, Circles, and Common Tangents
description: Intersecting a circle with a line by dropping a perpendicular from the center, reducing circle-circle to circle-line via the radical axis, and enumerating the up-to-four common tangents of two circles.
date: 2024-08-13
draft: false
slug: /cp/circle-intersections
tags:
  - Competitive Programming
  - Geometry
  - Computational Geometry
---

Circle intersection problems all reduce to one construction: the foot of the perpendicular from the circle's centre to a line, plus a right-triangle half-chord. Circle-circle intersection becomes circle-line intersection with the **radical axis**, and common tangents are handled by shrinking/growing radii to turn them into point-to-circle tangents.

## The problem

- **Circle-line:** intersection points of circle $(C, r)$ and a line.
- **Circle-circle:** intersection points of $(C_1, r_1)$ and $(C_2, r_2)$ — 0, 1, 2 points, or infinitely many (identical circles).
- **Common tangents:** the lines tangent to both of two circles — 4 (disjoint), 3 (externally tangent), 2 (overlapping), 1 (internally tangent), 0 (one inside the other).

These need floating point (square roots), so epsilon comparisons throughout.

## Circle-line

Line as $\{P : \vec n \cdot P = c\}$ with unit normal $\vec n$. Distance from centre $C$ to the line is $d = |\vec n \cdot C - c|$. The foot of the perpendicular is $F = C - (\vec n \cdot C - c)\,\vec n$.

- $d > r$: no intersection.
- $d = r$: tangent, one point $F$.
- $d < r$: two points $F \pm \sqrt{r^2 - d^2}\;\vec t$, where $\vec t$ is the line's unit direction ($\vec n$ rotated $90°$).

```python
import math

def circle_line(cx, cy, r, x1, y1, x2, y2, eps=1e-9):
    # line through (x1,y1)-(x2,y2)
    dx, dy = x2 - x1, y2 - y1
    L = math.hypot(dx, dy)
    dx, dy = dx / L, dy / L                        # unit direction
    # foot of perpendicular from center
    t = (cx - x1) * dx + (cy - y1) * dy
    fx, fy = x1 + t * dx, y1 + t * dy
    d2 = (fx - cx) ** 2 + (fy - cy) ** 2
    if d2 > r * r + eps:
        return []
    h = math.sqrt(max(0.0, r * r - d2))
    if h < eps:
        return [(fx, fy)]
    return [(fx - h * dx, fy - h * dy), (fx + h * dx, fy + h * dy)]
```

## Circle-circle via the radical axis

Two circles $x^2 + y^2 + D_i x + E_i y + F_i = 0$. Subtracting the equations cancels the quadratic terms, leaving a **line** — the radical axis — that contains both intersection points. So: compute the radical axis, then intersect it with either circle.

Concretely, with $\vec c = C_2 - C_1$ and $d = |\vec c|$:

- $d > r_1 + r_2$ or $d < |r_1 - r_2|$: no intersection ($0$ points).
- $d = 0$ and $r_1 = r_2$: identical circles (infinite).
- else: along $\vec c$, the intersection points are at distance $a = \dfrac{d^2 + r_1^2 - r_2^2}{2d}$ from $C_1$, offset by $h = \sqrt{r_1^2 - a^2}$ perpendicular to $\vec c$.

```python
def circle_circle(x1, y1, r1, x2, y2, r2, eps=1e-9):
    dx, dy = x2 - x1, y2 - y1
    d2 = dx * dx + dy * dy
    d = math.sqrt(d2)
    if d < eps and abs(r1 - r2) < eps:
        return "identical"
    if d > r1 + r2 + eps or d < abs(r1 - r2) - eps or d < eps:
        return []
    a = (d2 + r1 * r1 - r2 * r2) / (2 * d)
    h2 = r1 * r1 - a * a
    h = math.sqrt(max(0.0, h2))
    xm, ym = x1 + a * dx / d, y1 + a * dy / d
    if h < eps:
        return [(xm, ym)]
    ox, oy = -dy / d * h, dx / d * h
    return [(xm + ox, ym + oy), (xm - ox, ym - oy)]
```

## Common tangents

Reduce to "lines through the origin tangent to a circle" by a shift trick. For each sign choice $\varepsilon_1, \varepsilon_2 \in \{-1, +1\}$, seek a line $\vec n \cdot P + c = 0$ with $|\vec n| = 1$, $\vec n \cdot C_1 + c = \varepsilon_1 r_1$, $\vec n \cdot C_2 + c = \varepsilon_2 r_2$. Subtracting: $\vec n \cdot (C_2 - C_1) = \varepsilon_2 r_2 - \varepsilon_1 r_1$. With $\vec c = C_2 - C_1$ and $\delta = \varepsilon_2 r_2 - \varepsilon_1 r_1$, solve $\vec n \cdot \vec c = \delta$, $|\vec n| = 1$ — a quadratic giving the tangent's normal, then $c = \varepsilon_1 r_1 - \vec n \cdot C_1$.

- $\varepsilon_1 = \varepsilon_2$: the two **external** tangents.
- $\varepsilon_1 \ne \varepsilon_2$: the two **internal** tangents (only real when the circles are disjoint).

```python
def common_tangents(x1, y1, r1, x2, y2, r2, eps=1e-9):
    cx, cy = x2 - x1, y2 - y1
    d2 = cx * cx + cy * cy
    tangents = []
    for s1 in (1, -1):
        for s2 in (1, -1):
            dr = s2 * r2 - s1 * r1
            disc = d2 - dr * dr
            if disc < -eps:
                continue
            disc = math.sqrt(max(0.0, disc))
            nx = (cx * dr + cy * disc) / d2
            ny = (cy * dr - cx * disc) / d2
            c = s1 * r1 - (nx * x1 + ny * y1)
            tangents.append((nx, ny, c))          # line nx*x + ny*y + c = 0, (nx,ny) unit
    return tangents
```

## Common pitfalls

- **Epsilon everywhere.** Square roots and divisions mean nothing is exact. Compare $d$ to $r_1 \pm r_2$ with a tolerance, and clamp negatives before `sqrt`.
- **Identical circles.** $d = 0$ and $r_1 = r_2$ → infinitely many intersection points; special-case it before dividing by $d$.
- **One circle inside the other.** $d < |r_1 - r_2|$ → no intersection and no common tangents; the tangent quadratic has negative discriminant.
- **Degenerate line.** A "line" given by two equal points has zero direction — guard `L > eps`.
- **Tangent count at the boundary.** Externally tangent circles have $3$ common tangents (the internal one degenerates to the touch point's shared tangent); the code returns near-duplicate lines — dedupe if the problem wants the count.

## The keystone

Circle-line intersection is the foot of the perpendicular plus a half-chord $\sqrt{r^2 - d^2}$. Circle-circle reduces to that via the radical axis (subtract the circle equations to get a line). Common tangents come from solving $\vec n \cdot (C_2 - C_1) = \pm r_2 \pm r_1$ with $|\vec n| = 1$ for each of the four sign choices.
