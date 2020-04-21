---
title: Limits and Derivatives - Rates of Change Done Rigorously
description: Speed at an instant is a 0/0 that the limit makes sense of. From the ε–δ definition through the differentiation rules, the mean value theorems, and the first and second derivative tests for finding maxima and minima.
date: 2020-04-21
draft: false
slug: /maths/limits-derivatives
tags:
  - Mathematics
  - Calculus
---

Drop a stone. Over the interval from $t = 1\,\text{s}$ to $t = 1.1\,\text{s}$ its average speed is easy — distance fallen divided by $0.1\,\text{s}$. But "how fast is it going *at* $t = 1$" asks for distance divided by zero elapsed time, which is $0/0$ and means nothing as written. Shrink the interval and the average speed settles toward a definite number ($9.8\,\text{m/s}$); the **limit** is the tool that makes "settles toward" precise, and the **derivative** is what you get when you apply it to that ratio.

This post covers both: the $\varepsilon$–$\delta$ definition of a limit and the toolkit for evaluating one, continuity and its failure modes, the derivative from first principles and the rules that follow, the mean value theorems that tie a derivative to a function's overall behaviour, and the first- and second-derivative tests for locating maxima and minima.

---
## Limits: the $\varepsilon$–$\delta$ idea

**Informally**, $\lim_{x \to c} f(x) = L$ means you can force $f(x)$ as close to $L$ as you like by taking $x$ close enough to $c$ — without ever setting $x = c$.

**Formally:** $\lim_{x \to c} f(x) = L$ if for every $\varepsilon > 0$ there is a $\delta > 0$ such that
$$ 0 < |x - c| < \delta \;\Longrightarrow\; |f(x) - L| < \varepsilon $$
$\varepsilon$ is the tolerance you must hit around $L$; $\delta$ is how close to $c$ you have to sit to hit it. The limit exists only if the approach from both sides agrees:
$$ \lim_{x \to c} f(x) = L \iff \lim_{x \to c^-} f(x) = \lim_{x \to c^+} f(x) = L $$
When direct substitution gives $\tfrac00$, $\tfrac{\infty}{\infty}$, $0\cdot\infty$, $\infty - \infty$, $1^\infty$, $0^0$, or $\infty^0$, the form is **indeterminate** and needs real work.

---
## Working out limits

**The algebra of limits.** If $\lim f = L$ and $\lim g = M$ (both finite), then the limit of $f \pm g$, $fg$, $kf$, $f/g$ (for $M \neq 0$), $f^g$, and $|f|$ is what you'd get by substituting $L$ and $M$. If $f$ is continuous at $M$, $\lim f(g(x)) = f(M)$.

**Squeeze theorem.** If $g(x) \le f(x) \le h(x)$ near $c$ and $\lim g = \lim h = L$, then $\lim f = L$ — trap the function between two others that agree in the limit.

**The practical toolbox** for an indeterminate algebraic limit: substitute directly if you can; for $\tfrac00$ in a rational function, factor and cancel the vanishing factor; for roots, multiply by the conjugate; for $x \to \infty$, divide top and bottom by the highest power of $x$ in the denominator. One standard result covers a whole class:
$$ \lim_{x \to a} \frac{x^n - a^n}{x - a} = n a^{n-1} $$
Set $x = a + h$; the [binomial expansion](/citadel/maths/binomial-theory) of $(a + h)^n$ is $a^n + n a^{n-1} h + (\text{terms in } h^2)$, so the ratio is $n a^{n-1} + (\text{terms in } h) \to n a^{n-1}$.

**Standard transcendental limits** (all as $x \to 0$):
$$ \frac{\sin x}{x} \to 1, \qquad \frac{1 - \cos x}{x^2} \to \frac12, \qquad \frac{a^x - 1}{x} \to \ln a, \qquad \frac{\ln(1 + x)}{x} \to 1, \qquad (1 + x)^{1/x} \to e $$
The first is the seed for all trig derivatives. On the unit circle, for small $x > 0$, the triangle–sector–triangle areas nest as $\tfrac12\sin x \le \tfrac12 x \le \tfrac12\tan x$, i.e. $\cos x \le \tfrac{\sin x}{x} \le 1$; as $x \to 0$ both bounds go to $1$, and the squeeze finishes it.

For $x \to 0$ limits, substituting the first terms of a **Taylor series** often collapses the problem: $e^x = 1 + x + \tfrac{x^2}{2!} + \cdots$, $\sin x = x - \tfrac{x^3}{3!} + \cdots$, $\cos x = 1 - \tfrac{x^2}{2!} + \cdots$, $\ln(1 + x) = x - \tfrac{x^2}{2} + \cdots$. And **L'Hôpital's rule**: for $\tfrac00$ or $\tfrac{\infty}{\infty}$ with $f, g$ differentiable near $c$ and $g' \neq 0$,
$$ \lim_{x \to c} \frac{f(x)}{g(x)} = \lim_{x \to c} \frac{f'(x)}{g'(x)} $$
whenever the right-hand limit exists or is $\pm\infty$.

---
## Continuity

$f$ is **continuous at $c$** when three things hold: $f(c)$ is defined, $\lim_{x \to c} f(x)$ exists, and the two are equal. The picture is a graph you can draw without lifting the pen. On $[a, b]$, continuity means continuity at every interior point plus one-sided continuity at each end.

Discontinuities come in kinds:

- **Removable** — the limit exists but misses $f(c)$ (or $f(c)$ is undefined); a single "hole" you could plug.
- **Jump** — the one-sided limits exist but differ; the graph steps. The greatest-integer function $[x]$ does this at every integer, $\operatorname{sgn}(x)$ at $0$.
- **Infinite** — a one-sided limit runs off to $\pm\infty$; a vertical asymptote.

Polynomials, $\sin$, $\cos$, and $e^x$ are continuous everywhere; rational functions everywhere their denominator is nonzero; $|x|$ everywhere; a composite of continuous functions is continuous. The payoff theorem is the **Intermediate Value Theorem**: a function continuous on $[a, b]$ takes every value between $f(a)$ and $f(b)$ somewhere in $(a, b)$ — the basis of every "sign change means a root" argument.

---
## The derivative

$$ f'(x) = \lim_{h \to 0} \frac{f(x + h) - f(x)}{h} $$
when that limit exists — differentiation **from first principles**. It is the slope of the tangent to $y = f(x)$, and the instantaneous rate of change of $f$: for position $s(t)$, $s'(t)$ is velocity.

Differentiability needs the left and right difference quotients to exist *and match*. It is strictly stronger than continuity, and the implications run one way only:

```mermaid
flowchart LR
    A["differentiable at c"] -->|"always"| B["continuous at c"]
    B -->|"always"| C["limit exists at c"]
    B -.->|"not always<br/>(abs value at 0)"| A
```

$|x|$ is continuous at $0$ but has slope $-1$ from the left and $+1$ from the right, so it is not differentiable there — a corner. In general, differentiable-plus-non-differentiable is non-differentiable; and if $f(x) = |g(x)|$ fails to be differentiable at $a$ while $g$ is smooth, it is because $g(a) = 0$.

---
## Rules and standard forms

$$ (f \pm g)' = f' \pm g', \qquad (fg)' = f'g + fg', \qquad \left(\frac{f}{g}\right)' = \frac{f'g - fg'}{g^2}, \qquad \big(f(g(x))\big)' = f'(g(x))\,g'(x) $$
The **product rule** comes from adding and subtracting one cross term in the difference quotient:
$$ \frac{f(x+h)g(x+h) - f(x)g(x)}{h} = f(x+h)\,\frac{g(x+h) - g(x)}{h} + g(x)\,\frac{f(x+h) - f(x)}{h} $$
and letting $h \to 0$ ($f$ continuous, so $f(x+h) \to f(x)$) leaves $f g' + g f'$. The **chain rule** in Leibniz notation is $\dfrac{dy}{dx} = \dfrac{dy}{du}\cdot\dfrac{du}{dx}$.

**Standard derivatives:**

| $f(x)$ | $f'(x)$ | $f(x)$ | $f'(x)$ |
| --- | --- | --- | --- |
| $x^n$ | $n x^{n-1}$ | $\sin x$ | $\cos x$ |
| $e^x$ | $e^x$ | $\cos x$ | $-\sin x$ |
| $a^x$ | $a^x \ln a$ | $\tan x$ | $\sec^2 x$ |
| $\ln\lvert x\rvert$ | $1/x$ | $\sec x$ | $\sec x \tan x$ |
| $\log_a x$ | $1/(x \ln a)$ | $\cot x$ | $-\csc^2 x$ |
| $\sin^{-1} x$ | $1/\sqrt{1 - x^2}$ | $\tan^{-1} x$ | $1/(1 + x^2)$ |
| $\cos^{-1} x$ | $-1/\sqrt{1 - x^2}$ | $\sec^{-1} x$ | $1/(\lvert x\rvert\sqrt{x^2 - 1})$ |

**Trigonometric substitutions** that rationalise a radical: $x = a\tan\theta$ for $x^2 + a^2$, $x = a\sec\theta$ for $x^2 - a^2$, $x = a\sin\theta$ for $a^2 - x^2$, $x = a\cos\theta$ for $\sqrt{\tfrac{a+x}{a-x}}$.

---
## Beyond the basic rules

- **Implicit differentiation** — for a relation like $x^2 + y^2 = r^2$, differentiate through, treating $y$ as a function of $x$, then solve for $dy/dx$.
- **Logarithmic differentiation** — for $y = x^x$ or a tangle of products, take $\ln$ of both sides, simplify with log laws, differentiate implicitly.
- **Parametric** — with $x = f(t)$, $y = g(t)$: $\dfrac{dy}{dx} = \dfrac{g'(t)}{f'(t)}$.
- **One function against another** — $\dfrac{du}{dv} = \dfrac{du/dx}{dv/dx}$.
- **Determinant of a matrix $A(t)$** — $\dfrac{d}{dt}\det A(t) = \det A(t)\cdot \operatorname{tr}\!\big(A^{-1}(t)\,A'(t)\big)$.
- **Higher order** — $f''$ is the derivative of $f'$, and so on to $f^{(n)}$.
- **Partial derivatives** — for $f(x, y)$, differentiate with respect to one variable holding the others fixed; written $\partial f/\partial x$.

---
## What the derivative says about a curve

At $(x_1, y_1)$ on $y = f(x)$ with slope $m = f'(x_1)$:
$$ \text{tangent: } y - y_1 = m(x - x_1), \qquad \text{normal: } y - y_1 = -\frac{1}{m}(x - x_1) $$
The **subtangent** and **subnormal** are the projections of the tangent and normal segments onto the $x$-axis, with lengths $|y_1/m|$ and $|y_1 m|$; the tangent and normal segments themselves are $|y_1/m|\sqrt{1 + m^2}$ and $|y_1|\sqrt{1 + m^2}$.

The **angle between two curves** at a crossing is the angle between their tangents, $\tan\theta = \left|\dfrac{m_1 - m_2}{1 + m_1 m_2}\right|$.

**Curvature** measures how fast the tangent direction turns:
$$ \kappa = \frac{|y''|}{\big(1 + (y')^2\big)^{3/2}}, \qquad R = \frac{1}{\kappa} $$
$R$ is the radius of the circle that best hugs the curve at that point.

---
## The mean value theorems

- **Rolle's theorem.** $f$ continuous on $[a, b]$, differentiable on $(a, b)$, and $f(a) = f(b)$ $\Rightarrow$ some $c \in (a, b)$ has $f'(c) = 0$. A curve that returns to its starting height must be level somewhere.
- **Lagrange's MVT.** Drop the $f(a) = f(b)$ requirement: some $c$ has
  $$ f'(c) = \frac{f(b) - f(a)}{b - a} $$
  the instantaneous rate somewhere equals the average rate. Proof: apply Rolle to $g(x) = f(x) - \dfrac{f(b) - f(a)}{b - a}(x - a)$, which has $g(a) = g(b) = f(a)$.
- **Cauchy's MVT.** For two functions, with $g' \neq 0$ on $(a, b)$: some $c$ has $\dfrac{f(b) - f(a)}{g(b) - g(a)} = \dfrac{f'(c)}{g'(c)}$. This is the engine behind L'Hôpital's rule.

---
## Increasing, decreasing, and extreme

Where $f' > 0$ the function strictly increases; where $f' < 0$ it strictly decreases. A **critical point** is where $f'(c) = 0$ or $f'$ is undefined — the only places a local extremum can sit.

- **First-derivative test.** At a critical point $c$: $f'$ going $+ \to -$ is a local maximum, $- \to +$ a local minimum, no sign change is neither.
- **Second-derivative test.** If $f'(c) = 0$: $f''(c) < 0$ is a local max, $f''(c) > 0$ a local min, $f''(c) = 0$ inconclusive.
- **Inflection point** — where concavity flips; there $f''(c) = 0$ or is undefined *and* $f''$ changes sign.
- **Absolute extrema on $[a, b]$** — compare $f$ at every interior critical point and at both endpoints; largest and smallest win.

This last procedure is **optimization**: maximum enclosed area for a fixed fence, minimum material for a required volume, least-cost production, the path of least time. Every such problem is "write the quantity as a function of one variable, set its derivative to zero, check the candidates."

---
## The one idea to keep

Limits give calculus its rigour — a way to speak precisely about the infinitely small — and the derivative is the first thing that rigour buys: an exact rate of change where naive arithmetic gives $0/0$. The reverse operation, recovering a function from its rate, is [integration](/citadel/maths/integral-calculus); putting a derivative into an equation and solving for the unknown function is a [differential equation](/citadel/maths/differential-equations). Both lean on everything above.
