---
title: Advanced Differential Equations - Series, Laplace, Fourier
description: What to do when the characteristic-equation method runs out — power series and Frobenius for variable coefficients, the Laplace transform that turns calculus into algebra, Fourier decomposition into harmonics, and separation of variables for PDEs.
date: 2020-05-02
draft: false
slug: /maths/differential-equations-2
tags:
  - Mathematics
  - Calculus
---

The [elementary methods](/citadel/maths/differential-equations) — separate, integrate, guess $e^{\lambda x}$ — cover linear equations with constant coefficients and little else. Variable coefficients, discontinuous forcing, an impulse, a problem posed by its boundaries rather than its start, more than one independent variable: each breaks the standard toolkit and needs its own.

This post is the next three tools. **Power series** solve variable-coefficient ODEs and, in doing so, define the special functions (Legendre, Bessel) that physics keeps needing. The **Laplace transform** converts an ODE with initial conditions into an algebra problem. **Fourier analysis** breaks a function into pure sine and cosine components, which is exactly what separating a PDE demands.

---
## Series solutions

When the coefficients of a linear ODE are analytic but not constant, look for a solution as a power series
$$ y(x) = \sum_{n=0}^{\infty} a_n (x - x_0)^n $$
Substitute, collect powers of $x$, and set each coefficient to zero: this produces a **recurrence relation** that generates the $a_n$ from the first one or two.

**Legendre's equation** $(1 - x^2)y'' - 2xy' + \alpha(\alpha + 1)y = 0$ turns up in every spherically symmetric problem — electrostatic potential, the hydrogen atom's angular part. When $\alpha = n$ is a non-negative integer, one series solution *terminates* into a degree-$n$ polynomial, the **Legendre polynomial**
$$ P_n(x) = \frac{1}{2^n}\sum_{k=0}^{\lfloor n/2 \rfloor} \frac{(-1)^k (2n - 2k)!}{k!\,(n - k)!\,(n - 2k)!}\, x^{n - 2k} $$
so $P_0 = 1$, $P_1 = x$, $P_2 = \tfrac12(3x^2 - 1)$, $P_3 = \tfrac12(5x^3 - 3x)$. The independent second solution $Q_n(x) = \tfrac12 P_n(x)\ln\frac{1+x}{1-x} + R_n(x)$ blows up at $x = \pm 1$, so bounded problems keep only $P_n$.

**Frobenius method.** At a **regular singular point** the plain series fails; instead try
$$ y(x) = \sum_{n=0}^{\infty} a_n x^{n + r} $$
where $r$ (possibly non-integer) solves an **indicial equation** from the lowest-power terms.

**Bessel's equation** $x^2 y'' + xy' + (x^2 - \nu^2)y = 0$ — circular membranes, heat in a cylinder, cylindrical waveguides — has $x = 0$ as a regular singular point. The Frobenius solution finite at the origin is the **Bessel function of the first kind**
$$ J_\nu(x) = \sum_{k=0}^{\infty} \frac{(-1)^k\, (x/2)^{\nu + 2k}}{k!\,\Gamma(k + \nu + 1)}, \qquad \Gamma(z) = \int_0^\infty t^{z-1}e^{-t}\,dt $$
For integer order, $J_{-n}(x) = (-1)^n J_n(x)$, and $J_n$ has the integral form $J_n(x) = \tfrac{1}{\pi}\int_0^\pi \cos(n\theta - x\sin\theta)\,d\theta$; it oscillates and decays like $x^{-1/2}$ for large $x$. Useful identities: the recurrences $J_{\nu-1}(x) + J_{\nu+1}(x) = \tfrac{2\nu}{x}J_\nu(x)$ and $J_{\nu-1}(x) - J_{\nu+1}(x) = 2J_\nu'(x)$; the derivatives $(x^\nu J_\nu)' = x^\nu J_{\nu-1}$ and $(x^{-\nu}J_\nu)' = -x^{-\nu}J_{\nu+1}$; and the elementary cases $J_{1/2}(x) = \sqrt{\tfrac{2}{\pi x}}\sin x$, $J_{-1/2}(x) = \sqrt{\tfrac{2}{\pi x}}\cos x$. The second solution, $Y_\nu(x) = \dfrac{J_\nu(x)\cos\nu\pi - J_{-\nu}(x)}{\sin\nu\pi}$ (a limit for integer $\nu$), carries a $\ln(x/2)\,J_\nu(x)$ term and so blows up at $0$ — the general solution $c_1 J_\nu + c_2 Y_\nu$ drops $Y_\nu$ wherever the domain includes $x = 0$.

---
## Laplace transforms

$$ \mathcal{L}\{f(t)\} = F(s) = \int_0^\infty e^{-st} f(t)\,dt $$
Linear, and for a $p$-periodic $f$, $\mathcal{L}\{f\} = \dfrac{1}{1 - e^{-ps}}\displaystyle\int_0^p e^{-st}f(t)\,dt$.

The reason it solves ODEs is the **derivative rule**. Integrating $\int_0^\infty e^{-st}f'(t)\,dt$ by parts (with $[e^{-st}f(t)]_0^\infty = -f(0)$):
$$ \mathcal{L}\{f'\} = sF(s) - f(0), \qquad \mathcal{L}\{f''\} = s^2 F(s) - sf(0) - f'(0) $$
Each derivative becomes multiplication by $s$ plus initial-value terms, so an ODE with initial conditions turns into an *algebraic* equation for $F(s)$; solve it, then invert. Supporting rules:

- **$s$-shift:** $\mathcal{L}\{e^{at}f(t)\} = F(s - a)$ — from replacing $s$ by $s - a$ in the definition.
- **$t$-shift:** with the unit step $u(t - a)$ (0 before $a$, 1 after), $\mathcal{L}\{f(t - a)\,u(t - a)\} = e^{-as}F(s)$.
- **Impulse:** $\mathcal{L}\{\delta(t - a)\} = e^{-as}$, where $\delta$ is the Dirac spike of unit area.
- **Integral:** $\mathcal{L}\left\{\int_0^t f\right\} = F(s)/s$.
- **Transform calculus:** $\mathcal{L}\{t f(t)\} = -F'(s)$, $\mathcal{L}\{f(t)/t\} = \int_s^\infty F(\sigma)\,d\sigma$.
- **Convolution:** $(f * g)(t) = \int_0^t f(\tau)g(t - \tau)\,d\tau$ transforms to a plain product, $\mathcal{L}\{f * g\} = F(s)G(s)$ — the key to inverting products and solving integral equations.

| $f(t)$ | $F(s)$ | $f(t)$ | $F(s)$ |
| --- | --- | --- | --- |
| $1$ | $1/s$ | $\sin\omega t$ | $\omega/(s^2 + \omega^2)$ |
| $t^n$ | $n!/s^{n+1}$ | $\cos\omega t$ | $s/(s^2 + \omega^2)$ |
| $t^a$ $(a > -1)$ | $\Gamma(a+1)/s^{a+1}$ | $\sinh\omega t$ | $\omega/(s^2 - \omega^2)$ |
| $e^{at}$ | $1/(s - a)$ | $\cosh\omega t$ | $s/(s^2 - \omega^2)$ |
| $t^{n-1}e^{at}/(n-1)!$ | $1/(s - a)^n$ | $e^{at}\sin\omega t$ | $\omega/[(s-a)^2 + \omega^2]$ |
| | | $e^{at}\cos\omega t$ | $(s-a)/[(s-a)^2 + \omega^2]$ |

**Worked example.** Solve $y'' + y = 0$, $y(0) = 0$, $y'(0) = 1$. Transform both sides: $s^2 Y - s y(0) - y'(0) + Y = 0$, i.e. $(s^2 + 1)Y - 1 = 0$, so $Y(s) = \dfrac{1}{s^2 + 1}$. Read the table backwards: that is the transform of $\sin t$. The initial conditions were absorbed into the algebra — no arbitrary constants to fix afterwards, which is the whole advantage over the characteristic-equation route when the forcing is an impulse or a step.

---
## Fourier analysis

A **Fourier series** writes a periodic $f$ of period $2L$ as
$$ f(x) = a_0 + \sum_{n=1}^{\infty}\left(a_n\cos\frac{n\pi x}{L} + b_n\sin\frac{n\pi x}{L}\right) $$
The coefficients come from **orthogonality** — over $[-L, L]$, the integral of any two distinct members of $\{1, \cos\frac{n\pi x}{L}, \sin\frac{n\pi x}{L}\}$ is zero, and the integral of a squared sine or cosine is $L$. Multiply the series by one basis function and integrate; every term but one dies:
$$ a_0 = \frac{1}{2L}\int_{-L}^{L} f\,dx, \qquad a_n = \frac{1}{L}\int_{-L}^{L} f\cos\frac{n\pi x}{L}\,dx, \qquad b_n = \frac{1}{L}\int_{-L}^{L} f\sin\frac{n\pi x}{L}\,dx $$
An even $f$ has only cosine terms, an odd $f$ only sine terms; a function on $[0, L]$ can be extended either way for a **half-range** expansion. Where $f$ is continuous the series converges to it; at a jump it converges to the midpoint of the two one-sided limits (Dirichlet). In complex form,
$$ f(x) = \sum_{n=-\infty}^{\infty} c_n e^{i n\pi x/L}, \qquad c_n = \frac{1}{2L}\int_{-L}^{L} f(x)\,e^{-i n\pi x/L}\,dx $$

**Sturm–Liouville theory** is the general reason such expansions exist. Any equation $\big[p(x)y'\big]' + \big[q(x) + \lambda w(x)\big]y = 0$ with suitable boundary conditions has eigenfunctions $y_n$, for distinct eigenvalues $\lambda_n$, that are **orthogonal with weight $w$**: $\int_a^b y_m y_n w\,dx = 0$ for $m \neq n$. So any reasonable $f$ expands as $\sum a_n y_n(x)$ — a **generalised Fourier series**. Trigonometric functions, Legendre polynomials, and Bessel functions are all Sturm–Liouville eigenfunctions, which is why each gives an expansion.

For non-periodic functions the sum becomes an integral. The **Fourier transform** and its inverse:
$$ \hat{f}(\omega) = \frac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty} f(x)\,e^{-i\omega x}\,dx, \qquad f(x) = \frac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty} \hat{f}(\omega)\,e^{i\omega x}\,d\omega $$
with **cosine** and **sine** transforms $\sqrt{2/\pi}\int_0^\infty f(x)\cos\omega x\,dx$ and $\sqrt{2/\pi}\int_0^\infty f(x)\sin\omega x\,dx$ for even and odd functions; their derivative rules ($\mathcal{F}_s\{f'\} = -\omega\,\mathcal{F}_c\{f\}$, and $\mathcal{F}_c\{f''\} = -\omega^2\mathcal{F}_c\{f\} - \sqrt{2/\pi}\,f'(0)$) turn a PDE into an ODE. **Parseval's theorem** $\int|f|^2\,dx = \int|\hat{f}|^2\,d\omega$ says the transform preserves energy; the **convolution theorem** $\mathcal{F}\{f * g\} = \sqrt{2\pi}\,\hat{f}\hat{g}$ mirrors Laplace's. The **DFT/FFT** are the discrete numerical versions that run digital signal processing.

---
## Partial differential equations

A **PDE** relates a multivariable function to its partial derivatives. The workhorse solution method is **separation of variables** — assume the answer is a product of one-variable functions. For the 1D heat equation $\dfrac{\partial u}{\partial t} = k\dfrac{\partial^2 u}{\partial x^2}$, set $u(x, t) = X(x)T(t)$:
$$ X T' = k X'' T \;\Longrightarrow\; \frac{T'}{kT} = \frac{X''}{X} $$
The left side depends only on $t$, the right only on $x$, so both equal a constant $-\lambda^2$. That splits the PDE into two ODEs, $T' + \lambda^2 k T = 0$ and $X'' + \lambda^2 X = 0$; solve each, impose the boundary conditions on $X$, and superpose the allowed modes — which assembles into a Fourier series.

**Laplace's equation** $\nabla^2 u = 0$ governs steady-state potentials. Its form is coordinate-dependent — in polar coordinates $\nabla^2 u = u_{rr} + \tfrac1r u_r + \tfrac{1}{r^2}u_{\theta\theta}$; the cylindrical version adds $u_{zz}$ and separates into Bessel functions, the spherical version into Legendre polynomials (spherical harmonics). A unique solution needs boundary data: **Dirichlet** fixes $u$ on the boundary, **Neumann** fixes the normal derivative $\partial u/\partial n$. The Laplace transform also attacks PDEs, transforming the time variable and leaving an ODE in space.

---
## The one idea to keep

The through-line is *change of representation*. A Laplace transform trades $t$-domain calculus for $s$-domain algebra; a Fourier transform trades a function for its spectrum; a series solution trades a function for its coefficient sequence. In each case the hard operation — differentiation, a discontinuity, a variable coefficient — becomes easy in the new representation, and the work is getting there and back. These methods feed straight into the [wave equation](/citadel/physics/electromag) and the quantum [hydrogen atom](/citadel/physics/quantum-formalism), where Legendre and Bessel functions are not curiosities but the answer.
