---
title: Polygon Area - The Shoelace Formula and Pick's Theorem
description: The signed area of any simple polygon as a sum of cross products around its vertices, what the sign tells you about winding, and Pick's theorem relating area to interior and boundary lattice points.
date: 2024-08-15
draft: false
slug: /cp/polygon-area
tags:
  - Competitive Programming
  - Geometry
  - Polygons
---

The area of a polygon given its vertices — convex or not, as long as it does not self-intersect — is one pass of cross products. The **signed** version also tells you the winding direction, and for lattice polygons **Pick's theorem** ties the area to the count of grid points inside and on the boundary.

## The problem

Given the vertices $P_0, P_1, \dots, P_{n-1}$ of a simple polygon in order (CW or CCW), compute its area. For lattice polygons, also relate area to $I$ = interior lattice points and $B$ = boundary lattice points.

Example: the triangle $(0,0), (4,0), (0,3)$ has area $6$.

## The shoelace formula

$$2 \cdot \text{signed area} = \sum_{i=0}^{n-1} \big(x_i\, y_{i+1} - x_{i+1}\, y_i\big) = \sum_{i=0}^{n-1} P_i \times P_{i+1}$$

(indices mod $n$). Each term is twice the signed area of the triangle $O P_i P_{i+1}$; summed around the polygon, the parts outside cancel and only the interior remains — regardless of where the origin sits or whether the polygon is convex.

- **Sign:** positive if the vertices are in counter-clockwise order, negative if clockwise. Take the absolute value for the unsigned area.
- Fixing one vertex $P_0$ and summing $\text{orient}(P_0, P_i, P_{i+1})$ over triangles is the same sum rearranged — sometimes numerically nicer.

## How it works

Triangle $(0,0), (4,0), (0,3)$:

$$\sum = (0\cdot 0 - 4\cdot 0) + (4\cdot 3 - 0\cdot 0) + (0\cdot 0 - 0\cdot 3) = 0 + 12 + 0 = 12,$$

so area $= 12 / 2 = 6$, and the sign is positive → the vertices are listed counter-clockwise.

## Algorithm

```python
def signed_area2(poly) -> int:
    """Twice the signed area (integer if input is integer). >0 CCW, <0 CW."""
    s = 0
    n = len(poly)
    for i in range(n):
        x1, y1 = poly[i]
        x2, y2 = poly[(i + 1) % n]
        s += x1 * y2 - x2 * y1
    return s

def area(poly) -> float:
    return abs(signed_area2(poly)) / 2

def is_ccw(poly) -> bool:
    return signed_area2(poly) > 0
```

Keep `signed_area2` as an integer for lattice input — exact, and it is what Pick's theorem needs.

## Pick's theorem

For a simple polygon whose vertices are all lattice points,

$$A = I + \frac{B}{2} - 1,$$

where $A$ is the area, $I$ the number of lattice points strictly inside, and $B$ the number on the boundary.

- **Boundary points on edge $P_i P_{i+1}$:** $\gcd(|x_{i+1} - x_i|,\ |y_{i+1} - y_i|)$; sum over edges to get $B$ (each vertex counted once).
- **Interior points:** rearrange Pick, $I = A - B/2 + 1$. Since $2A$ is the integer shoelace sum, $I = \dfrac{2A - B + 2}{2}$ is an integer.

```python
from math import gcd

def boundary_points(poly) -> int:
    n = len(poly)
    total = 0
    for i in range(n):
        x1, y1 = poly[i]
        x2, y2 = poly[(i + 1) % n]
        total += gcd(abs(x2 - x1), abs(y2 - y1))
    return total                                  # == number of lattice points on the boundary

def interior_points(poly) -> int:
    A2 = abs(signed_area2(poly))                  # == 2A
    B = boundary_points(poly)
    return (A2 - B + 2) // 2
```

## Common pitfalls

- **Non-simple polygons.** The shoelace formula assumes no self-intersection. A figure-eight gives a meaningless "signed area" (the lobes cancel). Split or reject.
- **Forgetting to close the polygon.** The last term wraps $P_{n-1} \to P_0$; index `(i + 1) % n`.
- **Odd shoelace sum.** $\sum (x_i y_{i+1} - x_{i+1} y_i)$ is always even for a lattice polygon (it equals $2A$ and $A$ can be a half-integer only if... no — for lattice polygons $2A$ is an integer, and Pick forces $A \in \frac12\mathbb{Z}$). Divide by $2$ *after* taking absolute value.
- **`gcd(0, 0)` for a repeated vertex.** A degenerate edge of length $0$ — filter duplicate consecutive vertices first.
- **Pick with non-lattice vertices.** The theorem needs integer coordinates. For a scaled polygon, scale back or use a generalisation.

## The keystone

The shoelace sum $\sum P_i \times P_{i+1}$ is twice the signed area of any simple polygon — sign gives the winding, absolute value the area. For lattice polygons, Pick's $A = I + B/2 - 1$ turns that area into a lattice-point count, with $B$ read off as a sum of edge $\gcd$s.
