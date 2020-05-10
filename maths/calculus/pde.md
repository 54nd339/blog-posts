---
title: Partial Differential Equations - The Classical Trio
description: An ODE has one independent variable; a PDE has several, and that changes everything. First-order equations move information along characteristic curves. The heat, wave, and Laplace equations behave in three fundamentally different ways - diffusive, propagating, and equilibrium - and separation of variables solves all three on nice domains.
date: 2020-05-10
draft: false
slug: /maths/pde
tags:
  - Mathematics
  - Differential Equations
---

An [ordinary differential equation](/citadel/maths/differential-equations) relates a function of one variable to its derivatives; you solve it and get a family of curves, pinned down by initial conditions. A **partial** differential equation relates a function of several variables — $u(x, t)$, $u(x, y, z)$ — to its partial derivatives. The extra dimensions change the character of the problem completely: instead of "given the state now, find the state later," you often have "given the values on the boundary of a region, find the values inside," and the same-looking equation can be diffusive, wave-like, or an equilibrium condition depending on one sign.

This post covers first-order PDEs via characteristics, the classification into three types, the three canonical second-order equations and how differently they behave, and separation of variables — the one technique that cracks all three on well-shaped domains.

---

## Classification

A general second-order linear PDE in two variables,

$$A\,u_{xx} + B\,u_{xy} + C\,u_{yy} + \text{(lower-order terms)} = 0,$$

is classified by the discriminant $B^2 - 4AC$, exactly like a conic section:

- **Hyperbolic** ($B^2 - 4AC > 0$) — the **wave equation** $u_{tt} = c^2 u_{xx}$. Signals propagate at finite speed; disturbances travel along characteristics; singularities in the data persist.
- **Parabolic** ($B^2 - 4AC = 0$) — the **heat / diffusion equation** $u_t = \alpha u_{xx}$. Infinite propagation speed but instant smoothing; any rough initial data becomes analytic for $t > 0$.
- **Elliptic** ($B^2 - 4AC < 0$) — **Laplace's equation** $u_{xx} + u_{yy} = 0$. No time direction at all; $u$ is determined everywhere by its values on the boundary, and it's as smooth as possible.

These three model behaviours — propagate, diffuse, equilibrate — are the reason the same PDE course keeps returning to the same three equations.

**Well-posedness** (Hadamard): a problem is well-posed if a solution exists, is unique, and depends continuously on the data. Each equation type has *its own* appropriate data:

- Hyperbolic: initial position and velocity on a spatial slice (a Cauchy problem).
- Parabolic: initial data plus boundary conditions for $t > 0$.
- Elliptic: boundary conditions on a closed curve (Dirichlet: $u$ given; Neumann: $\partial u/\partial n$ given).

Pairing the wrong data with an equation gives an ill-posed problem — e.g. solving the heat equation *backward* in time amplifies every high-frequency wiggle without bound.

---

## First-order equations and characteristics

Consider $a(x, t)\,u_x + b(x, t)\,u_t = 0$. Along a curve $(x(s), t(s))$ with $\dot x = a$, $\dot t = b$, the chain rule gives $\frac{d}{ds} u(x(s), t(s)) = a u_x + b u_t = 0$. So **$u$ is constant along these curves** — the **characteristics**. The PDE has been reduced to ODEs for the characteristic curves plus "carry the value along."

For the transport equation $u_t + c\,u_x = 0$ with $u(x, 0) = f(x)$: the characteristics are the lines $x - ct = \text{const}$, and the solution is $u(x, t) = f(x - ct)$ — the initial profile sliding right at speed $c$, undistorted.

For a **quasilinear** equation like $u_t + u\,u_x = 0$ (inviscid Burgers), the characteristic speed depends on $u$ itself. Faster parts of the wave overtake slower parts; characteristics cross; the solution develops a **shock** (a discontinuity) in finite time, after which you need a weak/entropy formulation. This is how nonlinear PDEs form singularities from smooth data — the mechanism behind sonic booms and traffic jams.

---

## The heat equation

$$u_t = \alpha\, u_{xx}, \qquad x \in (0, L),\quad u(x, 0) = f(x),\quad u(0,t) = u(L,t) = 0.$$

Physically: temperature in a rod with the ends held at zero. Properties worth internalising:

- **Smoothing.** For any $t > 0$, $u(\cdot, t)$ is infinitely differentiable, no matter how rough $f$ was. Diffusion destroys high-frequency detail fast (the $n$-th Fourier mode decays like $e^{-\alpha n^2 \pi^2 t / L^2}$ — quadratically faster for higher modes).
- **Maximum principle.** The maximum of $u$ over the space–time region occurs on the initial slice or the boundary, never strictly inside. Heat doesn't spontaneously concentrate.
- **Infinite speed.** A localized hot spot instantly raises the temperature everywhere (by an exponentially tiny amount). Unphysical in the strict sense, harmless in practice.

The **fundamental solution** on the whole line is the Gaussian $\Phi(x, t) = \frac{1}{\sqrt{4\pi\alpha t}} e^{-x^2/(4\alpha t)}$; the general solution is $f$ convolved with $\Phi$ — the diffusion spreads mass out as a widening bell curve, which is the same reason the [heat kernel underlies the normal distribution](/citadel/maths/probablity-statistics) and Brownian motion.

---

## The wave equation

$$u_{tt} = c^2\, u_{xx}.$$

On the infinite line, **d'Alembert's solution** with $u(x,0) = f(x)$, $u_t(x,0) = g(x)$:

$$u(x, t) = \tfrac{1}{2}\big[f(x - ct) + f(x + ct)\big] + \frac{1}{2c}\int_{x - ct}^{x + ct} g(\xi)\,d\xi.$$

Two copies of the initial shape, one moving left and one right at speed $c$, plus a term from the initial velocity. Read off:

- **Finite propagation speed.** The value at $(x, t)$ depends only on the initial data in $[x - ct, x + ct]$ — the **domain of dependence**. Nothing outside that interval can have influenced it. (Contrast the heat equation, where every point depends on all initial data.)
- **No smoothing.** A kink in $f$ stays a kink, travelling along a characteristic $x \pm ct = \text{const}$ forever. Energy is conserved, not dissipated.
- **Energy method.** $E(t) = \frac{1}{2}\int (u_t^2 + c^2 u_x^2)\,dx$ is constant; differentiating and integrating by parts shows $\dot E = 0$. This is the standard route to *uniqueness*: if two solutions have the same data, their difference has zero energy, hence is zero.

---

## Laplace's equation

$$u_{xx} + u_{yy} = 0.$$

Solutions are **harmonic functions** — the steady state of the heat equation ($u_t = 0$), the electrostatic potential in a charge-free region, the velocity potential of an ideal fluid.

- **Mean-value property.** $u$ at the centre of any disk equals its average over the boundary circle. Harmonic functions can't have a strict interior max or min (**maximum principle** again) — a "hill" would exceed its surrounding average.
- **Boundary determines everything.** The Dirichlet problem — $u$ harmonic inside, equal to given values on the boundary — has a unique solution. There's no evolution; the boundary data instantly fixes the interior.
- **Regularity.** Harmonic functions are real-analytic. Ellipticity is maximally smoothing.

---

## Separation of variables

The technique that solves all three on a rectangle, disk, or box. For the heat equation with the setup above, guess $u(x, t) = X(x)\,T(t)$. Substituting:

$$X T' = \alpha X'' T \;\Longrightarrow\; \frac{T'}{\alpha T} = \frac{X''}{X} = -\lambda$$

(a function of $t$ equals a function of $x$, so both equal a constant). Now two ODEs:

- $X'' + \lambda X = 0$ with $X(0) = X(L) = 0$ — an **eigenvalue problem**. Non-trivial solutions only for $\lambda_n = \left(\frac{n\pi}{L}\right)^2$, with $X_n(x) = \sin\frac{n\pi x}{L}$.
- $T' = -\alpha\lambda_n T \Rightarrow T_n(t) = e^{-\alpha\lambda_n t}$.

Superpose: $u(x, t) = \sum_{n=1}^{\infty} b_n \sin\frac{n\pi x}{L}\, e^{-\alpha (n\pi/L)^2 t}$, and choose the $b_n$ to be the [Fourier sine coefficients](/citadel/maths/differential-equations-2) of the initial data $f$. Each mode decays at its own rate; high modes vanish fastest, which *is* the smoothing.

The same recipe works for the wave equation (the $T_n$ oscillate, $\cos\omega_n t$ and $\sin\omega_n t$, instead of decaying — these are the vibrating string's harmonics) and for Laplace's equation on a rectangle (one variable's ODE is oscillatory, the other's exponential). On a disk the separated ODEs are the Bessel equation; on a sphere, Legendre's — the **special functions** are just the eigenfunctions of the Laplacian on non-rectangular domains. The general framework is **Sturm–Liouville theory**: a broad class of eigenvalue problems whose eigenfunctions are orthogonal and complete, so "expand the data in eigenfunctions and evolve each mode" always works.

![The first four vibrational modes of a string fixed at both ends: half a sine wave, one full wave, and so on, with nodes at fixed points.](../images/string-harmonics.png "The sine eigenfunctions of the 1-D Laplacian with fixed ends - the wave equation's harmonics and the heat equation's decay modes are the same spatial shapes. Source: Wikimedia Commons.")

**Green's functions** handle the same equations with a source term or awkward boundary: find the response $G(x, \xi)$ to a unit point source at $\xi$, and the solution for a general source $\rho$ is $\int G(x, \xi)\,\rho(\xi)\,d\xi$. The fundamental solution of the heat equation is exactly its free-space Green's function.

When the domain is genuinely irregular, none of this closed-form machinery applies and you discretise — **finite differences** (replace derivatives with difference quotients on a grid) or **finite elements** (expand in local basis functions and solve a linear system), the subject of [numerical analysis](/citadel/maths/numerical-analysis).

---

## The one idea to keep

Going from one independent variable to several turns "evolve the state forward" into a family of distinct problems, sorted by the sign of $B^2 - 4AC$: hyperbolic equations *propagate* information at finite speed along characteristics with no smoothing (the wave equation, d'Alembert), parabolic equations *diffuse* — infinite speed but instant smoothing, high frequencies killed fastest (the heat equation, a spreading Gaussian), and elliptic equations *equilibrate*, with the boundary data fixing every interior value (Laplace, the mean-value property). Separation of variables cracks all three on nice domains by turning the PDE into an eigenvalue problem whose eigenfunctions you expand the data in.
