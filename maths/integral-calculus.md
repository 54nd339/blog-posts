---
title: Integral Calculus - Antiderivatives and the Limit of a Sum
description: Integration is two ideas that turn out to be one — undoing a derivative, and adding up infinitely many slivers. The standard antiderivatives and techniques, the definite integral as a Riemann sum, the fundamental theorem that links them, and the properties that make hard integrals easy.
date: 2020-04-25
draft: false
slug: /maths/integral-calculus
tags:
  - Mathematics
  - Calculus
---

There are two questions that look unrelated. *What function has $2x$ as its derivative?* And: *what is the area under $y = x^2$ from $0$ to $3$?* The first is a puzzle in reverse-engineering; the second is a sum of infinitely many thin rectangles. The Fundamental Theorem of Calculus says they have the same answer machinery — evaluate an antiderivative at the two ends and subtract — and that is the single most useful fact in the subject.

This post covers both sides. The antiderivative side: the table of standard integrals and the techniques (substitution, parts, partial fractions, rationalising a radical) for the ones that don't fit it. The sum side: the definite integral as a limit of Riemann sums, the properties that shortcut evaluation (symmetry, periodicity, the $a + b - x$ reflection), Leibniz's rule for differentiating an integral, and the integral inequalities.

---
## The indefinite integral

$F$ is an **antiderivative** of $f$ if $F' = f$. The **indefinite integral** collects all of them:
$$ \int f(x)\,dx = F(x) + C $$
The constant $C$ is unavoidable — any two antiderivatives of the same $f$ differ by a constant, since a function with zero derivative everywhere is constant.

**Standard antiderivatives**, each just a derivative rule read backwards:

$$ \int x^n\,dx = \frac{x^{n+1}}{n+1} + C \;(n \neq -1), \qquad \int \frac{dx}{x} = \ln|x| + C $$
$$ \int e^{ax}\,dx = \frac{e^{ax}}{a} + C, \qquad \int a^x\,dx = \frac{a^x}{\ln a} + C, \qquad \int \ln x\,dx = x\ln x - x + C $$
$$ \int \sin x\,dx = -\cos x + C, \qquad \int \cos x\,dx = \sin x + C, \qquad \int \sec^2 x\,dx = \tan x + C $$
$$ \int \sec x\,dx = \ln|\sec x + \tan x| + C, \qquad \int \tan x\,dx = \ln|\sec x| + C $$
$$ \int \frac{dx}{a^2 + x^2} = \frac1a\tan^{-1}\frac{x}{a} + C, \qquad \int \frac{dx}{\sqrt{a^2 - x^2}} = \sin^{-1}\frac{x}{a} + C $$
The hyperbolic functions mirror the trig list: $\int \sinh x\,dx = \cosh x + C$, $\int \operatorname{sech}^2 x\,dx = \tanh x + C$, and so on.

Integration is linear — $\int(kf \pm g) = k\int f \pm \int g$ — and $\int f(ax + b)\,dx = \tfrac1a F(ax + b) + C$. Two patterns worth recognising on sight:
$$ \int \frac{f'(x)}{f(x)}\,dx = \ln|f(x)| + C, \qquad \int \big(f(x)\big)^n f'(x)\,dx = \frac{\big(f(x)\big)^{n+1}}{n+1} + C $$

---
## Techniques

**Substitution** reverses the chain rule. For $\int f(g(x))\,g'(x)\,dx$, put $u = g(x)$, $du = g'(x)\,dx$, and the integral becomes $\int f(u)\,du$. Trigonometric substitutions clear radicals: $x = a\sin\theta$ for $\sqrt{a^2 - x^2}$, $x = a\tan\theta$ for $\sqrt{a^2 + x^2}$, $x = a\sec\theta$ for $\sqrt{x^2 - a^2}$. These generate the standard forms
$$ \int \frac{dx}{x^2 - a^2} = \frac{1}{2a}\ln\left|\frac{x - a}{x + a}\right| + C, \qquad \int \frac{dx}{\sqrt{a^2 + x^2}} = \sinh^{-1}\frac{x}{a} + C $$
For $\int \dfrac{px + q}{ax^2 + bx + c}\,dx$, write the numerator as $\lambda(2ax + b) + \mu$ (matching coefficients), splitting it into a $\ln$ piece and an $\arctan$ piece. Rational functions of $\sin x, \cos x$ yield to the **Weierstrass substitution** $t = \tan\tfrac{x}{2}$, under which $\sin x = \dfrac{2t}{1 + t^2}$, $\cos x = \dfrac{1 - t^2}{1 + t^2}$, $dx = \dfrac{2\,dt}{1 + t^2}$; forms like $\int \dfrac{dx}{a\cos^2 x + b\sin^2 x}$ are quicker with $t = \tan x$.

**Integration by parts** reverses the product rule. Integrating $\dfrac{d}{dx}(uv) = uv' + u'v$ and rearranging:
$$ \int u\,v'\,dx = uv - \int u'\,v\,dx $$
Choose $u$ by **ILATE** — Inverse-trig, Log, Algebraic, Trig, Exponential — the earlier in that list, the better a choice for $u$ (it should simplify when differentiated). A useful consequence: $\int e^x\big(f(x) + f'(x)\big)\,dx = e^x f(x) + C$, and by parts twice,
$$ \int e^{ax}\sin bx\,dx = \frac{e^{ax}}{a^2 + b^2}\big(a\sin bx - b\cos bx\big) + C $$

**Partial fractions** integrate a rational function $P/Q$ with $\deg P < \deg Q$ by splitting on the factors of $Q$: a distinct linear factor $x - a$ contributes $\dfrac{A}{x - a}$; a repeated factor $(x - a)^k$ contributes $\dfrac{A_1}{x - a} + \cdots + \dfrac{A_k}{(x - a)^k}$; an irreducible quadratic contributes $\dfrac{Bx + C}{x^2 + bx + c}$. Each piece is then a $\ln$ or an $\arctan$.

**Irrational integrands** are attacked by rationalising substitutions. The three square-root-of-quadratic integrals are standard:
$$ \int \sqrt{a^2 - x^2}\,dx = \frac{x}{2}\sqrt{a^2 - x^2} + \frac{a^2}{2}\sin^{-1}\frac{x}{a} + C $$
with the $\sqrt{a^2 + x^2}$ and $\sqrt{x^2 - a^2}$ versions replacing the last term by $\pm\tfrac{a^2}{2}\ln|x + \sqrt{\cdots}|$. For $\int \dfrac{dx}{(\text{linear})\sqrt{\text{quadratic}}}$ put the linear part $= 1/t$; for $\int \dfrac{dx}{(ax^2 + b)\sqrt{cx^2 + d}}$ put $x = 1/t$.

---
## The definite integral

$\displaystyle\int_a^b f(x)\,dx$ is the **signed area** between $y = f(x)$ and the $x$-axis over $[a, b]$ — below the axis counts negative. Its definition is a limit: partition $[a, b]$ into $n$ pieces of width $\Delta x = \tfrac{b - a}{n}$, pick a sample point $x_k$ in each, and add up the rectangle areas:
$$ \int_a^b f(x)\,dx = \lim_{n \to \infty}\sum_{k=1}^{n} f(x_k)\,\Delta x $$
Evaluating one from the definition needs closed-form sums — $\sum k = \tfrac{n(n+1)}{2}$, $\sum k^2 = \tfrac{n(n+1)(2n+1)}{6}$, $\sum k^3 = \big[\tfrac{n(n+1)}{2}\big]^2$, the geometric sum, and the arithmetic-argument trig sums
$$ \sum_{k=1}^{n}\sin\big(\alpha + (k-1)\beta\big) = \frac{\sin\frac{n\beta}{2}\,\sin\!\big(\alpha + \tfrac{n-1}{2}\beta\big)}{\sin\frac{\beta}{2}} $$
(with $\cos$ in place of the outer $\sin$ for the cosine version). Certain infinite series — $\sum 1/k^2 = \pi^2/6$, $\sum \tfrac{(-1)^{k-1}}{k} = \ln 2$ — show up as *values* of specific integrals and Fourier sums later, rather than as tools here.

---
## The fundamental theorem

**FTC part 1.** For $f$ continuous on $[a, b]$, the accumulation function $F(x) = \displaystyle\int_a^x f(t)\,dt$ is differentiable with
$$ F'(x) = f(x) $$
— differentiating an integral gives back the integrand.

**FTC part 2** (the evaluation rule). If $F$ is *any* antiderivative of a continuous $f$, then
$$ \int_a^b f(x)\,dx = F(b) - F(a) $$
Proof: let $G(x) = \int_a^x f$. By part 1, $G' = f$, so $F = G + C$. Then $F(b) - F(a) = G(b) - G(a) = \int_a^b f - 0$. This is what lets you skip the Riemann sum entirely.

---
## Properties that shortcut evaluation

Beyond linearity and $\int_a^b = \int_a^c + \int_c^b$:

- **Reversal:** $\int_a^b f = -\int_b^a f$, and $\int_a^a f = 0$.
- **Reflection:** $\displaystyle\int_a^b f(x)\,dx = \int_a^b f(a + b - x)\,dx$ — often halves the work by adding the integral to itself.
- **Doubling:** $\displaystyle\int_0^{2a} f(x)\,dx = \int_0^a\big[f(x) + f(2a - x)\big]\,dx$.
- **Symmetry:** over $[-a, a]$, an even $f$ gives $2\int_0^a f$, an odd $f$ gives $0$.
- **Periodicity:** for period $T$, $\displaystyle\int_0^{nT} f = n\int_0^T f$, and $\int_{a+nT}^{b+nT} f = \int_a^b f$.

A classic that falls out of the reflection property: $\displaystyle\int_0^{\pi/2}\ln\sin x\,dx = \int_0^{\pi/2}\ln\cos x\,dx = -\tfrac{\pi}{2}\ln 2$.

**Leibniz's rule** differentiates an integral with variable limits:
$$ \frac{d}{dx}\int_{a(x)}^{b(x)} f(x, t)\,dt = f(x, b)\,b'(x) - f(x, a)\,a'(x) + \int_{a(x)}^{b(x)} \frac{\partial f}{\partial x}\,dt $$

**Integral inequalities** parallel the ones for sums: $f \ge g$ on $[a, b]$ implies $\int f \ge \int g$; $m \le f \le M$ implies $m(b - a) \le \int_a^b f \le M(b - a)$; $\big|\int f\big| \le \int |f|$; and the Cauchy–Schwarz form
$$ \left(\int_a^b fg\,dx\right)^2 \le \int_a^b f^2\,dx \cdot \int_a^b g^2\,dx $$
with Minkowski's $\int|f + g| \le \int|f| + \int|g|$ alongside it.

---
## Area

For $f \ge 0$ on $[a, b]$, the area under the curve is $\int_a^b f\,dx$; where $f$ dips below the axis, take $|\cdot|$ on that stretch. Between two curves with $f \ge g$:
$$ A = \int_a^b \big[f(x) - g(x)\big]\,dx $$
Set the integral up only after **tracing the curves** — intercepts, symmetry, asymptotes, where each is on top — so the limits and the integrand are right. The area of the circle $x^2 + y^2 = r^2$ is $\int_{-r}^{r} 2\sqrt{r^2 - x^2}\,dx = \pi r^2$ (trig substitution), and the same method on $\tfrac{x^2}{a^2} + \tfrac{y^2}{b^2} = 1$ gives the [ellipse](/citadel/maths/conic-section) area $\pi ab$.

---
## The one idea to keep

Integration is the inverse of [differentiation](/citadel/maths/limits-derivatives) and, at the same time, the continuous version of a sum — the Fundamental Theorem is the claim that those are the same operation. That dual nature is why it computes areas and volumes *and* solves [differential equations](/citadel/maths/differential-equations), and why it extends, in [vector calculus](/citadel/maths/vector-calculus), to line and surface integrals over curved regions. The technique list looks long, but it is really four moves — substitute, integrate by parts, split into partial fractions, rationalise — tried in turn until one lands.
