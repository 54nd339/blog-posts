---
title: Vector and Tensor Calculus - Fields, Flows, and Curved Space
description: Calculus on functions that return a vector at every point. Gradient, divergence, and curl and what each measures; the line, surface, and volume integrals; the Green–Gauss–Stokes theorems that tie a field's interior to its boundary; and the tensor generalisation that curved spacetime needs.
date: 2020-05-03
draft: false
slug: /maths/vector-calculus
tags:
  - Mathematics
  - Vector Algebra
---

A weather map is a field: at every point it gives a temperature (a scalar) and a wind (a vector). To ask "where is it heating up fastest?", "is air piling up here or draining away?", "is the wind swirling?" you need calculus that acts on the *whole field* — differentiation and integration lifted from the line to two and three dimensions.

Vector calculus is that lift. This post covers the three field derivatives — **gradient**, **divergence**, **curl** — and what each one physically measures; the line, surface, and volume integrals that accumulate a field; the three great theorems (Green, Gauss, Stokes) that each say "the total of a derivative inside a region equals the field's behaviour on the boundary"; and the step up to **tensors**, which is what the maths of general relativity requires.

---
## Differentiating a vector function

For $\vec r(t)$ (a particle's position, say), the derivative is the componentwise limit
$$ \vec r\,'(t) = \lim_{\Delta t \to 0}\frac{\vec r(t + \Delta t) - \vec r(t)}{\Delta t} $$
— the velocity, tangent to the path. It is linear, obeys a product rule with a scalar function ($(f\vec r)' = f'\vec r + f\vec r\,'$), and threads through the vector products: $(\vec u \cdot \vec v)' = \vec u\,' \cdot \vec v + \vec u \cdot \vec v\,'$, likewise for $\times$ and for the scalar triple product.

For a **curve** $\vec r(t)$: an element of arc has $ds = |\vec r\,'(t)|\,dt$ (Pythagoras on $dx, dy, dz$), so the **arc length** is
$$ s(t) = \int_{t_0}^{t} |\vec r\,'(\tau)|\,d\tau $$
**Curvature** $\kappa = \left|\dfrac{d\hat T}{ds}\right| = \dfrac{|\vec r\,' \times \vec r\,''|}{|\vec r\,'|^3}$ measures how fast the unit tangent $\hat T$ turns; **torsion** measures how the curve lifts out of its instantaneous plane.

---
## Gradient, divergence, curl

All three are the **del operator** $\nabla = \hat i\,\partial_x + \hat j\,\partial_y + \hat k\,\partial_z$ acting in different ways.

**Gradient** — on a scalar field, returns a vector field:
$$ \nabla f = \frac{\partial f}{\partial x}\hat i + \frac{\partial f}{\partial y}\hat j + \frac{\partial f}{\partial z}\hat k $$
$\nabla f$ points in the direction of steepest increase of $f$, with $|\nabla f|$ that steepest rate. The **directional derivative** along a unit vector $\mathbf u$ is its projection: parametrise a line as $\vec r(s) = \vec r_0 + s\mathbf u$, apply the multivariable chain rule to $df/ds$, and
$$ D_{\mathbf u}f = \nabla f \cdot \mathbf u $$

**Divergence** — on a vector field, returns a scalar field:
$$ \nabla \cdot \vec F = \frac{\partial F_x}{\partial x} + \frac{\partial F_y}{\partial y} + \frac{\partial F_z}{\partial z} $$
It is the outflow per unit volume: positive at a source, negative at a sink, zero where whatever flows in flows out.

**Curl** — on a vector field, returns a vector field:
$$ \nabla \times \vec F = \det\begin{pmatrix} \hat i & \hat j & \hat k \\ \partial_x & \partial_y & \partial_z \\ F_x & F_y & F_z \end{pmatrix} $$
It measures local circulation: its direction is the axis a tiny paddlewheel would spin about (right-hand rule), its magnitude the spin rate.

The **Laplacian** $\nabla^2 f = \nabla \cdot (\nabla f)$ combines two of them and measures how much $f$ at a point differs from its neighbourhood average. Two identities do heavy lifting later: $\nabla \times (\nabla f) = \vec 0$ (a gradient never curls) and $\nabla \cdot (\nabla \times \vec F) = 0$ (a curl never diverges).

---
## Integrating a field

**Line integral of a scalar** $\displaystyle\int_C f\,ds = \int_a^b f(\vec r(t))\,|\vec r\,'(t)|\,dt$ — accumulate $f$ along a curve.

**Line integral of a vector field** — accumulate the tangential component:
$$ \int_C \vec F \cdot d\vec r = \int_a^b \vec F(\vec r(t)) \cdot \vec r\,'(t)\,dt $$
This is the **work** done by a force $\vec F$ along $C$. In general it depends on the path, not just the endpoints. But for a **conservative** field, four conditions are equivalent on a simply connected domain:

1. $\int_C \vec F \cdot d\vec r$ is path independent;
2. $\oint_C \vec F \cdot d\vec r = 0$ for every closed loop;
3. $\vec F = \nabla f$ for some scalar potential $f$;
4. $\nabla \times \vec F = \vec 0$.

That (3) $\Rightarrow$ (1) is the **Fundamental Theorem for Line Integrals**: $\displaystyle\int_C \nabla f \cdot d\vec r = \int_a^b \frac{d}{dt}f(\vec r(t))\,dt = f(B) - f(A)$ by the chain rule — endpoints only.

**Surface integrals.** Parametrise $S$ by $\vec r(u, v)$; the vector area element is $d\vec S = \left(\dfrac{\partial \vec r}{\partial u} \times \dfrac{\partial \vec r}{\partial v}\right)du\,dv = \hat n\,dS$. Then $\iint_S g\,dS$ sums a scalar over the surface, and
$$ \text{flux} = \iint_S \vec F \cdot d\vec S $$
is the net flow of $\vec F$ through $S$.

---
## The three theorems

Each generalises the Fundamental Theorem of Calculus: the integral of a derivative over a region equals the original function evaluated on the boundary.

**Green's theorem** (plane). For a positively oriented simple closed curve $C$ bounding a region $D$:
$$ \oint_C (P\,dx + Q\,dy) = \iint_D \left(\frac{\partial Q}{\partial x} - \frac{\partial P}{\partial y}\right)dA $$
Proved by establishing $\oint_C P\,dx = -\iint_D \partial_y P\,dA$ (and the $Q$ half) on a simple region via the 1D Fundamental Theorem, then patching general regions from simple pieces. A useful corollary, with $\partial f/\partial n$ the outward normal derivative: $\iint_D \nabla^2 f\,dA = \oint_C \dfrac{\partial f}{\partial n}\,ds$.

**Divergence theorem** (Gauss). For a solid $V$ with outward-oriented closed boundary surface $S$:
$$ \oiint_S \vec F \cdot d\vec S = \iiint_V (\nabla \cdot \vec F)\,dV $$
The net flux out of a closed surface equals the total of the sources inside. Applying it to $f\nabla g$ gives **Green's identities**, e.g. $\iiint_V (f\nabla^2 g - g\nabla^2 f)\,dV = \oiint_S \big(f\,\partial_n g - g\,\partial_n f\big)\,dS$.

**Stokes' theorem.** For an oriented surface $S$ bounded by a closed curve $C$:
$$ \oint_C \vec F \cdot d\vec r = \iint_S (\nabla \times \vec F) \cdot d\vec S $$
The circulation around the rim equals the total curl through any surface spanning it. Green's theorem is the flat special case. Together, these three are why Maxwell's equations have both a differential and an integral form — they are the same statements read through Gauss and Stokes.

---
## Tensors

Curved spaces and relativity need an object more general than a vector: a **tensor**, classified by **rank** — rank 0 a scalar, rank 1 a vector, rank 2 a matrix-like map from vectors to vectors — and defined by how its components transform under a change of coordinates.

**Covariant and contravariant.** In a curvilinear coordinate system a vector has two kinds of components: **contravariant** $V^i$ (upper index), which transform against the basis (a velocity), and **covariant** $V_i$ (lower index), which transform with it (a gradient). In an orthonormal Cartesian basis the two coincide.

**The metric tensor** $g_{ij} = \mathbf e_i \cdot \mathbf e_j$ encodes the geometry: it sets the **line element**
$$ ds^2 = \sum_{i,j} g_{ij}\,dx^i\,dx^j $$
In flat Cartesian space $g_{ij}$ is the identity and this is $dx^2 + dy^2 + dz^2$; in flat Minkowski spacetime it is $\operatorname{diag}(1, -1, -1, -1)$ (up to a $c^2$), giving $ds^2 = c^2\,dt^2 - dx^2 - dy^2 - dz^2$. The metric and its inverse $g^{ij}$ **raise and lower indices**: $V_i = \sum_j g_{ij}V^j$.

**The covariant derivative.** A plain partial derivative $\partial V^i/\partial x^j$ is not a tensor in curved coordinates, because the basis vectors themselves vary from point to point. The fix adds connection terms — the **Christoffel symbols** $\Gamma^k_{ij}$, built from derivatives of the metric:
$$ \nabla_j V^k = \frac{\partial V^k}{\partial x^j} + \sum_i \Gamma^k_{ij}\,V^i $$
(with a minus sign and a lowered index for a covariant vector). In Cartesian coordinates every $\Gamma^k_{ij} = 0$ and this collapses back to the partial derivative.

**General relativity** is written entirely in this language: the **stress–energy tensor** $T_{\mu\nu}$ (energy and momentum density and flux) is the source, the **Einstein tensor** $G_{\mu\nu}$ (spacetime curvature, from the metric and its derivatives) is the response, and
$$ G_{\mu\nu} = \frac{8\pi G}{c^4}\,T_{\mu\nu} $$
says matter tells spacetime how to curve. The [general-relativity post](/citadel/physics/astrodynamics-advanced) unpacks that chain.

---
## The one idea to keep

The pattern to carry away is the boundary principle: a field's total change through a region is fixed by its values on the edge — the FTC, Green, Gauss, and Stokes are one idea at rising dimension. Vector calculus is the language [electromagnetism](/citadel/physics/electromag) and fluid dynamics are written in; tensors are the extension that survives a change to any coordinates, curved ones included. The prerequisites are [vector algebra](/citadel/maths/vectors) and multivariable [differentiation and integration](/citadel/maths/integral-calculus).
