---
title: Analytical Mechanics - Lagrangian and Hamiltonian Formulations
description: Newton's laws need every force drawn as a vector, including the constraint forces you do not care about. Lagrangian mechanics rebuilds the whole subject from a single scalar and a principle of stationary action, constraint forces drop out, symmetries become conservation laws through Noether's theorem, and the Hamiltonian form is the doorway to quantum mechanics.
date: 2020-07-20
draft: false
slug: /physics/analytical-mechanics
tags:
  - Physics
  - Classical Mechanics
---

[Newtonian dynamics](/citadel/physics/newtonian-dynamics) works, but it makes you do bookkeeping you often don't want. For a bead on a wire, you must include the normal force the wire exerts — a force whose only job is to keep the bead on the wire, whose magnitude you don't know in advance, and whose value you don't care about. For a double pendulum you'd draw four unknown tension and reaction forces and solve for them alongside the motion.

Analytical mechanics, developed by Euler, Lagrange, and Hamilton, throws that away. You write down one scalar function — the difference between kinetic and potential energy — pick coordinates that already respect the constraints, and turn a crank. Constraint forces never appear. And the reformulation exposes structure Newton's vectors hide: every continuous symmetry of a system corresponds to a conserved quantity, and the equations take a form that carries almost unchanged into quantum mechanics and field theory.

## The calculus of variations

The mathematical tool first. Ordinary calculus finds the number $x$ that minimises a function $f(x)$. The calculus of variations finds the *function* $y(x)$ that minimises an integral

$$S[y] = \int_{a}^{b} L\big(x, y(x), y'(x)\big)\, dx,$$

called a **functional** — it eats a whole function and returns a number. The classic problem: among all curves joining two points, which gives the shortest length? (A straight line.) Among all frictionless tracks between two points, which gives the fastest slide under gravity? (A cycloid — the **brachistochrone**, solved by Johann Bernoulli in 1696.)

The condition for $y$ to make $S$ stationary (a minimum, maximum, or saddle in function space) is the **Euler–Lagrange equation**:

$$\frac{d}{dx}\!\left(\frac{\partial L}{\partial y'}\right) - \frac{\partial L}{\partial y} = 0.$$

*Why:* perturb the optimal $y$ by a small function $\varepsilon\,\eta(x)$ that vanishes at the endpoints. Expand $S[y + \varepsilon\eta]$ to first order in $\varepsilon$, integrate the $\eta'$ term by parts (the boundary term dies because $\eta(a) = \eta(b) = 0$), and demand the first-order change vanish for *every* $\eta$. The bracketed expression multiplying $\eta$ must then be zero everywhere — that's the equation above.

## The principle of stationary action

Now the physics. Define the **Lagrangian** of a mechanical system as

$$L = T - V,$$

kinetic energy minus potential energy — a single number at each instant, not a vector. Define the **action** as its time integral along a candidate trajectory:

$$S = \int_{t_1}^{t_2} L\, dt.$$

**Hamilton's principle:** the trajectory a system actually follows between two configurations at two times is the one for which $S$ is **stationary** — its first variation is zero.

Apply Euler–Lagrange with $t$ as the independent variable and the position coordinate $q$ playing the role of $y$:

$$\frac{d}{dt}\!\left(\frac{\partial L}{\partial \dot q}\right) - \frac{\partial L}{\partial q} = 0.$$

For a single particle in a potential, $L = \tfrac12 m\dot q^2 - V(q)$, so $\partial L/\partial \dot q = m\dot q$ and $\partial L/\partial q = -dV/dq$, and the equation reads $m\ddot q = -dV/dq = F$. **Newton's second law drops out.** The principle isn't a new physics; it's a repackaging — but the packaging is what pays.

## Generalised coordinates and constraints

The real power: you may use *any* set of coordinates $q_1, \dots, q_n$ that specifies the configuration — angles, arc lengths, normal-mode amplitudes — not just Cartesian $x, y, z$. Choose them to satisfy the constraints automatically.

A **holonomic** constraint is one expressible as an equation among coordinates (and possibly time): $f(q, t) = 0$. A bead on a wire of shape $y = g(x)$ has one such constraint, so instead of two Cartesian coordinates plus a constraint plus an unknown normal force, you use *one* generalised coordinate — the arc length $s$ along the wire — and write $L(s, \dot s)$ directly. The wire's normal force does no work (it's perpendicular to the motion), so it never enters $T - V$, and the Euler–Lagrange equation in $s$ gives the motion with no constraint force anywhere in sight. A double pendulum: two angles, one Lagrangian, two coupled equations, zero tension unknowns.

Non-holonomic constraints (inequalities, or conditions on velocities that don't integrate to coordinate relations — a ball rolling without slipping on a plane) need Lagrange multipliers or other machinery, and are where the method's elegance frays.

## Cyclic coordinates and conserved momenta

If the Lagrangian doesn't depend on a particular coordinate $q_k$ (only on its velocity $\dot q_k$), that coordinate is **cyclic**, and the Euler–Lagrange equation collapses:

$$\frac{d}{dt}\!\left(\frac{\partial L}{\partial \dot q_k}\right) = 0 \;\Longrightarrow\; p_k \equiv \frac{\partial L}{\partial \dot q_k} = \text{constant}.$$

$p_k$ is the **generalised** (or canonical) **momentum** conjugate to $q_k$. Examples: if $L$ has no explicit $x$ (translational symmetry), linear momentum $m\dot x$ is conserved; if $L$ has no explicit angle $\phi$ (rotational symmetry about that axis), angular momentum $\partial L/\partial \dot\phi$ is conserved. Conservation laws become *automatic* — you read them off by inspecting which coordinates are absent.

## Noether's theorem

Emmy Noether (1918) turned that observation into a theorem of enormous reach: **every continuous symmetry of the action corresponds to a conserved quantity.**

- Invariance under time translation ($L$ has no explicit $t$) → conservation of **energy**.
- Invariance under spatial translation → conservation of **linear momentum**.
- Invariance under rotation → conservation of **angular momentum**.

The theorem is constructive: given the symmetry transformation, it hands you the formula for the conserved current. In field theory this is how charge conservation follows from the phase symmetry of the electron field, and how the entire catalogue of conservation laws in particle physics is organised. It reframes "why is energy conserved?" as "because the laws of physics don't change with time" — a statement about symmetry, not about forces.

## The Hamiltonian

Lagrangian mechanics lives in configuration-and-velocity space $(q, \dot q)$ and gives $n$ second-order equations. The **Hamiltonian** formulation moves to configuration-and-*momentum* space $(q, p)$ and gives $2n$ first-order equations, with a more symmetric structure.

The switch is a **Legendre transform**. Define $p_i = \partial L/\partial \dot q_i$, solve for the $\dot q_i$ in terms of $(q, p)$, and set

$$H(q, p) = \sum_i p_i \dot q_i - L.$$

For the standard $L = T - V$ with $T$ quadratic in the velocities, this evaluates to $H = T + V$ — the **total energy**, now expressed as a function of positions and momenta. Hamilton's equations of motion are

$$\dot q_i = \frac{\partial H}{\partial p_i}, \qquad \dot p_i = -\frac{\partial H}{\partial q_i}.$$

Two first-order equations per degree of freedom, and a pleasing antisymmetry between $q$ and $p$: the roles of coordinate and momentum are nearly interchangeable.

The state of the system is now a single point in **phase space**, the $2n$-dimensional space of all $(q, p)$. As time runs, that point traces a curve, and Hamilton's equations are the flow. **Liouville's theorem** says this flow is incompressible: a blob of initial conditions sweeps out a region whose *volume* in phase space never changes, only its shape (it stretches and folds). That conservation of phase-space volume is the foundation of statistical mechanics.

## Poisson brackets and the bridge to quantum mechanics

For two phase-space functions $f(q, p)$ and $g(q, p)$, the **Poisson bracket** is

$$\{f, g\} = \sum_i \left(\frac{\partial f}{\partial q_i}\frac{\partial g}{\partial p_i} - \frac{\partial f}{\partial p_i}\frac{\partial g}{\partial q_i}\right).$$

Any quantity's time evolution is $\dot f = \{f, H\}$ (plus any explicit time dependence), so a quantity is conserved exactly when its bracket with $H$ vanishes. The fundamental brackets are $\{q_i, p_j\} = \delta_{ij}$, $\{q_i, q_j\} = \{p_i, p_j\} = 0$.

Compare the [quantum formalism](/citadel/physics/quantum-formalism): observables become operators, the Poisson bracket becomes the commutator via $\{f, g\} \to \frac{1}{i\hbar}[\hat f, \hat g]$, $\{q, p\} = 1$ becomes $[\hat q, \hat p] = i\hbar$, and $\dot f = \{f, H\}$ becomes the Heisenberg equation of motion. The Hamiltonian formulation is the classical skeleton that quantum mechanics puts flesh on — which is why "write down the Hamiltonian" is the first step in quantising any system.

**Canonical transformations** are changes of phase-space coordinates that preserve Hamilton's equations' form; the art of choosing a good one can make a hard problem trivial. The extreme case is the **Hamilton–Jacobi equation**, a single PDE for a function $S(q, t)$ whose solution turns the entire dynamics into "the new coordinates are constants" — and whose short-wavelength limit is geometrical optics, while its role in the WKB approximation makes it the classical limit of the Schrödinger equation.

## Small oscillations and normal modes

A practical payoff. Near a stable equilibrium, expand $V$ to second order and $T$ as a constant-matrix quadratic form. The Lagrangian becomes $L = \tfrac12 \dot{\mathbf q}^\top \mathbf M \dot{\mathbf q} - \tfrac12 \mathbf q^\top \mathbf K \mathbf q$, and Euler–Lagrange gives $\mathbf M\ddot{\mathbf q} + \mathbf K\mathbf q = 0$. Solving the generalised [eigenvalue problem](/citadel/maths/linear-algebra) $\mathbf K \mathbf v = \omega^2 \mathbf M \mathbf v$ yields the **normal modes**: special patterns of motion in which every coordinate oscillates at the same frequency $\omega_k$, and any small motion is a superposition of them. This is how you find the vibrational frequencies of a molecule, a bridge, or a crystal lattice — and in the last case, quantising each mode gives [phonons](/citadel/physics/solid-state).

![Lissajous figures for different frequency ratios between two perpendicular oscillations, forming closed loops when the ratio is rational.](../images/lissajous-ratios.png "Two normal modes superposed: the trajectory closes only when the frequency ratio is rational, otherwise it fills the box. Source: Wikimedia Commons.")

## The one idea to keep

Analytical mechanics rebuilds all of classical dynamics from one scalar, $L = T - V$, and one statement: the actual trajectory makes the action $\int L\,dt$ stationary. Choosing generalised coordinates that already satisfy the constraints makes the constraint forces vanish from the problem entirely, and any coordinate the Lagrangian doesn't depend on hands you a conserved momentum — the special case of Noether's theorem, which ties every continuous symmetry to a conservation law. The Legendre transform to $H(q, p)$ then puts the dynamics in phase space with a symmetric first-order form whose Poisson-bracket structure becomes, almost verbatim, the commutator algebra of quantum mechanics.
