---
title: Basic Geometry - Integer Vectors, Dot and Cross Products
description: The point/vector primitives every geometry solution starts from — dot product for projections and angles, cross product for orientation and area, and the sign tests that stay exact in integer arithmetic.
date: 2024-08-13
draft: false
slug: /cp/basic-geometry
tags:
  - Competitive Programming
  - Geometry
  - Vectors
---

Before any convex hull or sweep line, you need the primitives: represent a point, add and scale vectors, and compute the two products — dot and cross — that answer "what angle", "which side", "how much area". Kept in integer arithmetic (when the input allows), these tests are **exact**, which removes the biggest source of wrong answers in geometry.

## The primitives

A point is a pair $(x, y)$; a vector is the difference of two points. The two products:

$$\vec a \cdot \vec b = a_x b_x + a_y b_y = |\vec a|\,|\vec b|\cos\theta, \qquad \vec a \times \vec b = a_x b_y - a_y b_x = |\vec a|\,|\vec b|\sin\theta.$$

- **Dot product** — sign tells you the angle is acute ($+$), right ($0$), or obtuse ($-$); magnitude gives projections and $|\vec a|^2 = \vec a \cdot \vec a$.
- **Cross product** (a scalar in 2-D) — sign is the turn direction; magnitude is twice the triangle area and equals $|\vec a||\vec b|\sin\theta$.

## Orientation: the workhorse test

Given three points $P, Q, R$:

$$\text{orient}(P, Q, R) = (Q - P) \times (R - P).$$

- $> 0$: $P \to Q \to R$ is a **left turn** (counter-clockwise).
- $< 0$: a **right turn** (clockwise).
- $= 0$: collinear.

Almost every geometry algorithm is orientation tests glued together: convex hull, "is this polygon convex", "do these segments intersect", "is the point inside", polygon area sign.

## Code

```python
class P:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __sub__(a, b): return P(a.x - b.x, a.y - b.y)
    def __add__(a, b): return P(a.x + b.x, a.y + b.y)
    def __mul__(a, k): return P(a.x * k, a.y * k)

def dot(a: P, b: P):   return a.x * b.x + a.y * b.y
def cross(a: P, b: P): return a.x * b.y - a.y * b.x

def orient(p: P, q: P, r: P):
    return cross(q - p, r - p)                    # >0 ccw, <0 cw, 0 collinear

def sgn(v) -> int:
    return (v > 0) - (v < 0)

def dist2(a: P, b: P):                            # squared distance -- stays exact
    d = a - b
    return d.x * d.x + d.y * d.y

def angle_cmp(a: P, b: P) -> int:
    # sort vectors by polar angle in [0, 2pi), starting from the +x axis
    def half(v): return int(v.y < 0 or (v.y == 0 and v.x < 0))   # 0: [0,pi)  1: [pi,2pi)
    if half(a) != half(b):
        return half(a) - half(b)
    return -sgn(cross(a, b))                      # ccw order within a half-plane
```

## Common tasks from the primitives

- **Triangle area** $= \tfrac12 |\text{orient}(A, B, C)|$; signed area keeps the winding.
- **Point-to-line distance** from $P$ to line $AB$: $\dfrac{|\text{orient}(A, B, P)|}{|B - A|}$.
- **Point-to-segment distance:** project $P$ onto $AB$ via `dot(P - A, B - A) / dist2(A, B)`; clamp the parameter to $[0, 1]$; distance to the clamped point.
- **Is $P$ on segment $AB$:** `orient(A, B, P) == 0` **and** `dot(P - A, P - B) <= 0`.
- **Rotate a vector by $90°$:** $(x, y) \to (-y, x)$ — exact, no trig.
- **Rotate by $\theta$:** $(x\cos\theta - y\sin\theta,\ x\sin\theta + y\cos\theta)$ — now you are in floating point.

## Integer vs floating point

- If input coordinates are integers up to $C$, then `cross` is an integer up to $\sim 4C^2$. For $C = 10^9$ that is $4 \times 10^{18}$ — fits in signed 64-bit, barely. `orient` of differences can double it; use 128-bit or be careful.
- Keep everything squared and integer as long as possible: compare `dist2`, not `dist`; compare `cross`, not `angle`.
- Switch to `double` only for genuine irrationals: rotations by arbitrary angles, circle intersections, line-circle, anything with a `sqrt` or `atan2`.

## Common pitfalls

- **`atan2` for angle sorting.** Slow and imprecise. Use a cross-product comparator that first buckets by half-plane (as `angle_cmp` above).
- **Comparing `cross` to `0.0` with doubles.** Use an epsilon, or better, keep it integer.
- **Overflow in `cross` on large integer inputs.** $10^9$-scale coordinates overflow 64-bit after one multiply-and-subtract on differences; use Python (unbounded) or `__int128`.
- **Collinear as a special case.** Orientation $= 0$ needs its own branch almost everywhere — segment overlap, hull with collinear points, point-on-segment.
- **Winding direction.** Signed area / orient sign depends on CCW vs CW vertex order; normalise your polygons.

## The keystone

Represent points as vectors; the dot product ($a_x b_x + a_y b_y$) gives angles and projections, the cross product ($a_x b_y - a_y b_x$) gives turn direction and area. The orientation test $\text{sign}\big((Q-P)\times(R-P)\big)$ is the atom of nearly every geometry algorithm — and kept in integers, it never lies.
