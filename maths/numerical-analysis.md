---
title: Numerical Analysis - Approximating What You Can't Solve Exactly
description: When there is no closed form, you compute an approximation and bound its error. Round-off versus truncation error, root-finding by iteration, interpolation and splines, quadrature rules, and the Euler–Runge–Kutta family for differential equations.
date: 2020-05-05
draft: false
slug: /maths/numerical-analysis
tags:
  - Mathematics
  - Numerical Analysis
---

Most equations that describe real systems have no closed-form solution. You cannot write down the integral of $e^{-x^2}$, the roots of a generic degree-7 polynomial, or the trajectory of three gravitating bodies. **Numerical analysis** is the discipline of computing accurate approximations anyway — replacing a continuous problem with a discrete one a computer can grind through, and then bounding how far the answer can be off.

This post covers the core methods: the two kinds of error and how they propagate, root-finding by iteration (bisection, Newton, secant), interpolation and splines for fitting data, quadrature rules for integrals, and the Euler and Runge–Kutta methods for differential equations. Optimisation and graph algorithms are the companion post on [optimisation and graph theory](/citadel/maths/optimization-graph-theory).

---
## Error

Discretising a continuous problem introduces error of two kinds:

- **round-off** — the computer stores only finitely many digits, so every operation is slightly wrong;
- **truncation** — an infinite process (a Taylor series, an integral, a limit) is cut off after finitely many terms.

The central tension is cost versus accuracy: smaller steps mean less truncation error but more operations and more accumulated round-off. Errors are quantified as **absolute** $|x - \hat x|$ and **relative** $|x - \hat x| / |x|$, the latter being the meaningful one when values span many scales. Propagation rules of thumb: for addition and subtraction, the absolute error bounds add; for multiplication and division, the *relative* error bounds add. Subtracting two nearly equal numbers is the classic trap — it can wipe out every significant digit.

---
## Root-finding by iteration

To solve $f(x) = 0$, start with a guess and improve it until $|x_{n+1} - x_n| < \varepsilon$. The **rate** of convergence is $\lim \dfrac{|x_{n+1} - r|}{|x_n - r|^p}$ for the largest $p$ that keeps this finite — $p = 1$ linear, $p = 2$ quadratic (roughly doubling the correct digits each step).

- **Bisection.** If $f$ changes sign across $[a, b]$, a root lies between; halve the interval, keep the half with the sign change, repeat. Guaranteed to converge, but only linearly — one bit of accuracy per step.
- **Fixed-point iteration.** Rewrite $f(x) = 0$ as $x = g(x)$ and iterate $x_{n+1} = g(x_n)$. It converges to a fixed point (the root) when $g$ is a **contraction** — $|g'| < 1$ near the root.
- **Newton's method.** Take the tangent line at $x_n$ — from the first-order Taylor expansion $f(x) \approx f(x_n) + f'(x_n)(x - x_n)$ — and use its $x$-intercept:
  $$ x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)} $$
  It converges **quadratically** near a simple root of a smooth $f$, but can diverge if $f'(x_n) \approx 0$ or the start is poor.

  *Worked:* solve $x^2 - 2 = 0$ (i.e. find $\sqrt 2$) with $x_{n+1} = x_n - \dfrac{x_n^2 - 2}{2x_n} = \dfrac{x_n}{2} + \dfrac{1}{x_n}$. From $x_0 = 1$: $x_1 = 1.5$, $x_2 = 1.41\overline{6}$, $x_3 = 1.41421568\ldots$, $x_4 = 1.41421356237\ldots$ — correct to 11 decimals after four steps. The count of correct digits roughly doubles each iteration, which is what "quadratic" means in practice.
- **Secant method.** Newton's without the derivative: approximate $f'(x_n)$ by the slope through the last two points,
  $$ x_{n+1} = x_n - f(x_n)\,\frac{x_n - x_{n-1}}{f(x_n) - f(x_{n-1})} $$
  Convergence order about $1.618$ — slower than Newton per step, but no derivative needed and often more robust.

---
## Interpolation

**Interpolation** finds a function passing exactly through given data points $(x_i, y_i)$.

**Polynomial interpolation** fits one degree-$(n-1)$ polynomial through $n$ points. The **Lagrange form** writes it directly:
$$ P(x) = \sum_{i=0}^{n-1} y_i \prod_{j \neq i} \frac{x - x_j}{x_i - x_j} $$
**Newton's divided differences** build the same polynomial incrementally (cheaper to extend with a new point); on an equally spaced grid these become the forward-difference formula
$$ P(x) = f_0 + r\,\Delta f_0 + \frac{r(r-1)}{2!}\Delta^2 f_0 + \cdots, \qquad r = \frac{x - x_0}{h} $$
and its backward-difference twin using values near the end of the interval. The catch with a single high-degree polynomial is **Runge's phenomenon** — wild oscillation near the endpoints.

**Spline interpolation** avoids that by fitting a separate low-degree polynomial between each pair of points. A **cubic spline** uses a cubic on each subinterval and forces the value, first derivative, and second derivative to match at every interior knot — a smooth curve with no overshoot. **Hermite interpolation** goes further, matching prescribed *derivative* values at the data points as well.

---
## Numerical integration and differentiation

**Quadrature** approximates $\int_a^b f\,dx$ by sampling $f$ on a grid of spacing $h$.

- **Trapezoidal rule:** $\dfrac{h}{2}\big[f_0 + 2f_1 + \cdots + 2f_{n-1} + f_n\big]$ — straight-line segments, error $O(h^2)$.
- **Simpson's rule:** $\dfrac{h}{3}\big[f_0 + 4f_1 + 2f_2 + 4f_3 + \cdots + 4f_{n-1} + f_n\big]$ — parabolic segments over pairs of intervals, error $O(h^4)$.
- **Romberg integration** applies **Richardson extrapolation** to trapezoidal estimates at successively halved $h$, combining them as $R(m, k) = \dfrac{4^k R(m, k-1) - R(m-1, k-1)}{4^k - 1}$ to cancel leading error terms.
- **Gaussian quadrature** chooses both the sample points and their weights to integrate polynomials up to degree $2n - 1$ exactly with only $n$ evaluations: $\int_a^b f\,dx \approx \sum w_i f(x_i)$.
- **Monte Carlo** integration averages $f$ at random points: $(b - a)\cdot\frac1N\sum f(x_i)$, with error shrinking as $O(1/\sqrt N)$ regardless of dimension — the method of choice for high-dimensional integrals.

**Numerical differentiation** comes from Taylor expansions. From $f(x \pm h) = f(x) \pm h f'(x) + \tfrac{h^2}{2}f''(x) \pm \cdots$: the **forward** and **backward differences** $\dfrac{f(x + h) - f(x)}{h}$ have error $O(h)$; subtracting the two expansions cancels the $f''$ term and gives the **central difference**
$$ f'(x) \approx \frac{f(x + h) - f(x - h)}{2h}, \qquad \text{error } O(h^2) $$

---
## Differential equations

For the initial value problem $y' = f(x, y)$, $y(x_0) = y_0$, step forward by $h$:

- **Euler's method** — follow the tangent: $y_{n+1} = y_n + h\,f(x_n, y_n)$. Error per step $O(h^2)$, accumulated $O(h)$; simple but crude.
- **Heun's method** (improved Euler) — predictor–corrector: predict with Euler, then correct using the average of the slope at the start and at the predicted point, $y_{n+1} = y_n + \tfrac{h}{2}\big(f(x_n, y_n) + f(x_{n+1}, \tilde y_{n+1})\big)$.
- **Runge–Kutta 4 (RK4)** — sample the slope four times per step:
  $$ y_{n+1} = y_n + \frac{h}{6}\big(k_1 + 2k_2 + 2k_3 + k_4\big) $$
  with $k_1 = f(x_n, y_n)$, $k_2 = f(x_n + \tfrac h2, y_n + \tfrac h2 k_1)$, $k_3 = f(x_n + \tfrac h2, y_n + \tfrac h2 k_2)$, $k_4 = f(x_n + h, y_n + h k_3)$. Global error $O(h^4)$ — the standard workhorse.
- **Multistep methods** reuse several past points instead of re-sampling: **Adams–Bashforth** (explicit) predicts, **Adams–Moulton** (implicit) corrects, cheaper per step on smooth problems.

An $n$th-order ODE becomes a first-order **system** by naming the derivatives: $y'' = f(x, y, y')$ turns into $z_1' = z_2$, $z_2' = f(x, z_1, z_2)$, then solved with vector Euler or RK4. **Stiff** equations — where some component decays far faster than others — force tiny explicit steps for stability, so an **implicit** method like **backward Euler** $y_{n+1} = y_n + h\,f(x_{n+1}, y_{n+1})$ is used instead, solving a (nonlinear) equation each step in exchange for stability.

**PDEs** are handled by discretising the domain into a grid. **Elliptic** equations ($\nabla^2 u = 0$) become a large linear system via the **five-point stencil** for the Laplacian, solved iteratively. **Parabolic** equations (the heat equation) use explicit schemes like FTCS — easy but stability-limited — or the implicit, unconditionally stable **Crank–Nicolson**. **Hyperbolic** equations (the wave equation) use centred differences in both space and time.

---
## The one idea to keep

Every method here trades an exact answer you cannot get for an approximate one you can, plus an error bound. The recurring themes: Taylor expansion is where the formulas and their error orders come from; halving the step and extrapolating (Richardson, Romberg) buys accuracy cheaply; and implicit methods cost more per step but stay stable where explicit ones blow up. The optimisation and network side of computational mathematics is in [optimisation and graph theory](/citadel/maths/optimization-graph-theory); the exact calculus these methods approximate is in [differentiation](/citadel/maths/limits-derivatives), [integration](/citadel/maths/integral-calculus), and [differential equations](/citadel/maths/differential-equations).
