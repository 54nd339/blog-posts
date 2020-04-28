---
title: Real Analysis - Rigour Behind the Calculus
description: Calculus works; real analysis proves it does, and shows where it doesn't. The completeness of the reals, why every Cauchy sequence converges, the difference uniform convergence makes, and the theorems - Bolzano-Weierstrass, Heine-Borel, the mean value theorem - that calculus quietly assumes.
date: 2020-04-28
draft: false
slug: /maths/real-analysis
tags:
  - Mathematics
  - Analysis
---

Calculus hands you rules: a continuous function on a closed interval attains a maximum; you can swap a limit and an integral; a bounded increasing sequence converges. You use them and they work. Real analysis asks *why* they work, finds the one property of the real numbers that everything rests on, and — just as usefully — produces the counterexamples that show which plausible-looking statements are false.

The payoff isn't just rigour for its own sake. Knowing that $\sum \frac{\sin(n^2 x)}{n^2}$ can be differentiated term by term but $\sum \frac{\sin(nx)}{n}$ cannot, and knowing the exact condition that separates the two, is the difference between manipulating series safely and getting a wrong answer.

---

## The completeness axiom

The rationals $\mathbb{Q}$ have a hole where $\sqrt 2$ should be: the set $\{q \in \mathbb{Q} : q^2 < 2\}$ is bounded above (by $2$, say) but has no *least* upper bound in $\mathbb{Q}$ — you can always find a rational upper bound closer to $\sqrt 2$. The real numbers are built precisely to fill every such hole.

**Completeness axiom.** Every non-empty subset of $\mathbb{R}$ that is bounded above has a least upper bound (a **supremum**) in $\mathbb{R}$.

This single axiom is what distinguishes $\mathbb{R}$ from $\mathbb{Q}$, and essentially every theorem below traces back to it. Formally $\mathbb{R}$ is constructed from $\mathbb{Q}$ either as **Dedekind cuts** (a real number *is* a downward-closed set of rationals with no maximum) or as equivalence classes of **Cauchy sequences** of rationals; both give the same ordered field, and both make completeness a theorem rather than an assumption.

Two immediate consequences:

- **Archimedean property** — for any real $x$ there is a natural number $n > x$. (If not, $\mathbb{N}$ would be bounded above, and its supremum $s$ would force $s - 1$ to be an upper bound too — contradiction.)
- **Density** — between any two distinct reals there is a rational, and an irrational.

---

## Sequences and convergence

A sequence $(a_n)$ **converges** to $L$ if for every $\varepsilon > 0$ there is an $N$ with $|a_n - L| < \varepsilon$ for all $n \ge N$. The same $\varepsilon$–$N$ shape as the [limit of a function](/citadel/maths/limits-derivatives), now indexed by $n$.

**Monotone convergence theorem.** A bounded, monotone sequence converges. *Proof:* if $(a_n)$ increases and is bounded above, let $L = \sup\{a_n\}$ (exists by completeness). For any $\varepsilon$, $L - \varepsilon$ isn't an upper bound, so some $a_N > L - \varepsilon$; monotonicity then puts every later term in $(L - \varepsilon, L]$. $\square$

This is the first place completeness does visible work: the same statement is false in $\mathbb{Q}$ (take rational truncations of $\sqrt 2$).

**Bolzano–Weierstrass.** Every bounded sequence in $\mathbb{R}$ has a convergent subsequence. *Proof sketch:* bisect the bounding interval; one half contains infinitely many terms; recurse. The nested intervals shrink to a point (completeness), and picking one term from each gives a subsequence converging to it. $\square$

**Cauchy sequences.** $(a_n)$ is **Cauchy** if for every $\varepsilon$ there is an $N$ with $|a_n - a_m| < \varepsilon$ for all $n, m \ge N$ — the terms get arbitrarily close *to each other*, with no limit named.

**Cauchy criterion.** In $\mathbb{R}$, a sequence converges *if and only if* it is Cauchy.

The "if" direction is completeness in disguise (again false in $\mathbb{Q}$), and it's enormously useful: you can prove convergence without knowing the limit, which is exactly what you need for series and for constructing solutions to equations. A metric space in which every Cauchy sequence converges is called **complete**; $\mathbb{R}$ and $\mathbb{R}^n$ are, $\mathbb{Q}$ is not.

---

## Topology of the real line

- **Open set** — a union of open intervals; equivalently, every point has a little interval around it still inside the set.
- **Closed set** — complement of an open set; equivalently, contains all its limit points.
- **Compact set** — the key notion. In $\mathbb{R}^n$, three definitions coincide:
  1. Closed and bounded.
  2. Every sequence has a subsequence converging to a point *of the set* (sequential compactness).
  3. Every open cover has a finite subcover.

**Heine–Borel theorem** is the statement that (1) $\Leftrightarrow$ (3) in $\mathbb{R}^n$. Compactness is what "closed interval" is really contributing in the calculus theorems: it lets you upgrade a *local* property (continuous at each point) to a *global* one (bounded, attains its max) by covering the set with finitely many small pieces.

---

## Continuity, done carefully

$f$ is **continuous at $c$** if $\lim_{x \to c} f(x) = f(c)$ — the $\varepsilon$–$\delta$ version: for every $\varepsilon$ there is a $\delta$ (allowed to depend on both $\varepsilon$ *and* $c$) with $|x - c| < \delta \Rightarrow |f(x) - f(c)| < \varepsilon$.

On a **compact** set, continuity gives you a lot:

- **Extreme value theorem** — a continuous function on a compact set attains a maximum and a minimum. (The image is compact, hence bounded and closed, hence contains its sup.)
- **Uniform continuity** — $f$ is uniformly continuous if a *single* $\delta$ works for every $c$ at once. **Heine–Cantor theorem:** a continuous function on a compact set is uniformly continuous. On a non-compact set it can fail: $f(x) = 1/x$ on $(0, 1]$ is continuous but not uniformly (the required $\delta$ shrinks to $0$ near the origin), and $f(x) = x^2$ on all of $\mathbb{R}$ fails for the opposite reason.

**Intermediate value theorem** — a continuous $f$ on $[a, b]$ takes every value between $f(a)$ and $f(b)$. This one needs *connectedness* of the interval rather than compactness, and it too is a completeness statement (the set where $f < \text{target}$ has a supremum, and continuity pins $f$ to the target there).

---

## Differentiation and the mean value theorem

$f'(c) = \lim_{h \to 0} \frac{f(c + h) - f(c)}{h}$ when the limit exists; differentiability at $c$ implies continuity at $c$ (not conversely — $|x|$ at $0$).

**Mean value theorem.** If $f$ is continuous on $[a, b]$ and differentiable on $(a, b)$, there is a $\xi \in (a, b)$ with

$$f'(\xi) = \frac{f(b) - f(a)}{b - a}.$$

*Proof:* Rolle's theorem (the $f(a) = f(b)$ case, where a max or min in the interior has a zero derivative — using the extreme value theorem, hence compactness) applied to $f(x) - \big[f(a) + \frac{f(b)-f(a)}{b-a}(x - a)\big]$. $\square$

The MVT is the workhorse behind "if $f' = 0$ everywhere then $f$ is constant," Taylor's theorem with the Lagrange remainder, and the error bounds in [numerical analysis](/citadel/maths/numerical-analysis).

**Riemann integrability.** $f$ is Riemann integrable on $[a, b]$ if the supremum of lower sums equals the infimum of upper sums over all partitions. A bounded function is Riemann integrable iff its set of discontinuities has **measure zero** (can be covered by intervals of arbitrarily small total length). So a function with countably many jumps is integrable; the indicator of the rationals (discontinuous everywhere) is not — which is one motivation for the [Lebesgue integral](/citadel/maths/set-theory).

---

## Sequences and series of functions

This is where the subtlety lives, and where "obvious" swaps of limits go wrong.

A sequence of functions $f_n \to f$ **pointwise** if $f_n(x) \to f(x)$ for each fixed $x$. It converges **uniformly** if

$$\sup_x |f_n(x) - f(x)| \to 0$$

— the worst-case gap over *all* $x$ shrinks, at one rate.

The difference is not academic:

- $f_n(x) = x^n$ on $[0, 1]$ converges pointwise to a function that's $0$ on $[0,1)$ and $1$ at $x = 1$ — each $f_n$ is continuous, the limit is not. Convergence here is not uniform (near $x = 1$ the gap stays large for every $n$).
- $f_n(x) = \frac{\sin(nx)}{\sqrt n}$ converges uniformly to $0$, but $f_n'(x) = \sqrt n \cos(nx)$ diverges — uniform convergence of functions does **not** imply anything about their derivatives.

What uniform convergence *does* buy:

- **Continuity is preserved.** A uniform limit of continuous functions is continuous.
- **Integration commutes with the limit.** $\int f_n \to \int f$ on a bounded interval.
- **Differentiation** needs more: if $f_n \to f$ pointwise *and* $f_n' \to g$ **uniformly**, then $f$ is differentiable with $f' = g$.

**Weierstrass M-test.** If $|f_n(x)| \le M_n$ for all $x$ and $\sum M_n$ converges, then $\sum f_n$ converges uniformly. This is the practical tool: it's why $\sum \frac{\sin(n^2 x)}{n^2}$ (bounded by $\sum 1/n^2$) is a well-behaved continuous function you can integrate term by term, while $\sum \frac{\sin(nx)}{n}$ (only conditionally convergent, no uniform bound) must be handled with care.

Two capstones:

- **Stone–Weierstrass theorem** — every continuous function on a compact interval is a *uniform* limit of polynomials. Continuous functions are exactly as flexible as polynomials, in the limit.
- **Arzelà–Ascoli theorem** — a family of functions has a uniformly convergent subsequence iff it is uniformly bounded and **equicontinuous** (one $\delta$ works for the whole family). This is the "Bolzano–Weierstrass for function spaces," and it's how existence proofs for differential equations and calculus-of-variations problems get off the ground.

---

## The one idea to keep

Every theorem calculus takes for granted rests on the completeness of $\mathbb{R}$ — the existence of least upper bounds — which is also why Cauchy sequences converge, why bounded sequences have convergent subsequences, and why closed bounded intervals are compact. Compactness is what turns "continuous at each point" into "bounded and attains its max." And the one distinction that decides whether you can move a limit past an integral or a derivative is *uniform* versus *pointwise* convergence: pointwise is not enough, uniform is enough for continuity and integration, and derivatives need the derivatives themselves to converge uniformly.
