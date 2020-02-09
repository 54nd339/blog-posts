---
title: Sequences and Series - Progressions, Means, and Sums
description: Whether an infinite sum settles on a number or runs away is decided by one quantity — the common ratio. Arithmetic, geometric, and harmonic progressions, the three means and the inequality that orders them, the shift-and-subtract sum trick, and the standard power-sum formulas.
date: 2020-02-09
draft: false
slug: /maths/progression
tags:
  - Mathematics
  - Algebra
---

Add $1 + 2 + 4 + 8 + \cdots$ and the total races to infinity. Add $1 + \tfrac12 + \tfrac14 + \tfrac18 + \cdots$ and it closes in on exactly $2$, never passing it. Same shape — each term a fixed multiple of the last — and the only difference is whether that multiple is bigger or smaller than $1$. That one number decides everything about a geometric sum.

This post covers the three classical progressions (arithmetic, geometric, harmonic), how to sum each one and when an infinite version converges, the arithmetic/geometric/harmonic means and the inequality $A \ge G \ge H$ that always orders them, the shift-and-subtract trick for hybrid series, and the closed forms for $\sum k$, $\sum k^2$, $\sum k^3$.

---
## Sequence, series, progression

A **sequence** $\{a_n\}$ is an ordered list of terms following a rule; it is finite or infinite. A **series** is what you get by adding a sequence's terms, and $S_n = \sum_{k=1}^{n} a_k$ is the **partial sum** of the first $n$. A **progression** is a sequence whose rule is a single fixed operation repeated — add a constant, multiply by a constant, or those applied to reciprocals.

---
## Arithmetic progressions

Consecutive terms differ by a constant, the **common difference** $d$. Starting from $a_1$,
$$ a_n = a_1 + (n - 1)d $$
For the sum, pair the first term with the last, the second with the second-last: each pair totals $a_1 + a_n$, and there are $n/2$ pairs. Hence
$$ S_n = \frac{n}{2}\big(a_1 + a_n\big) = \frac{n}{2}\big[2a_1 + (n - 1)d\big] $$
An infinite A.P. always diverges to $\pm\infty$ unless every term is zero — the terms don't shrink, so the sum can't settle.

---
## Geometric progressions

Consecutive terms have a constant *ratio*, the **common ratio** $r \neq 0$:
$$ a_n = a_1 r^{n-1} $$
To sum, write $S_n = a_1 + a_1 r + \cdots + a_1 r^{n-1}$, multiply through by $r$, and subtract: every middle term cancels, leaving $S_n(1 - r) = a_1(1 - r^n)$. So for $r \neq 1$,
$$ S_n = \frac{a_1(1 - r^n)}{1 - r} $$
and $S_n = n a_1$ when $r = 1$.

The infinite sum converges **iff $|r| < 1$**: then $r^n \to 0$, and
$$ S_\infty = \frac{a_1}{1 - r} $$
For $|r| \ge 1$ the terms don't die out and the series diverges. This is the exact line between the two sums in the opening: $1 + 2 + 4 + \cdots$ has $r = 2 \ge 1$ and diverges, while $1 + \tfrac12 + \tfrac14 + \cdots$ has $a_1 = 1$, $r = \tfrac12$, so $S_\infty = \dfrac{1}{1 - \tfrac12} = 2$ — approached but never reached. As a finite check, the first ten terms sum to $\dfrac{1 - (1/2)^{10}}{1 - 1/2} = 2 - \dfrac{1}{512} \approx 1.998$.

---
## Harmonic progressions

$a_1, a_2, a_3, \ldots$ (none zero) is an **H.P.** when the reciprocals $\tfrac{1}{a_1}, \tfrac{1}{a_2}, \ldots$ are an A.P. To get the $n$th term, work in that A.P.: with common difference $d' = \tfrac{1}{a_2} - \tfrac{1}{a_1}$,
$$ \frac{1}{a_n} = \frac{1}{a_1} + (n - 1)d', \qquad a_n = \left(\frac{1}{a_1} + (n - 1)d'\right)^{-1} $$
There is no elementary closed form for the sum of an H.P. — reciprocating an A.P.'s partial sum doesn't simplify.

---
## Means between two numbers

A **mean** of $a$ and $b$ is a value that sits between them keeping a chosen progression intact.

- **Arithmetic mean:** $a, A, b$ in A.P. forces $A = \dfrac{a + b}{2}$.
- **Geometric mean:** $a, G, b$ in G.P. forces $G^2 = ab$, so $G = \sqrt{ab}$ (for $a, b > 0$).
- **Harmonic mean:** $\tfrac{1}{a}, \tfrac{1}{H}, \tfrac{1}{b}$ in A.P. forces $\dfrac{1}{H} = \dfrac{a + b}{2ab}$, so $H = \dfrac{2ab}{a + b}$.

To **insert $k$ means** between $a$ and $b$, build the progression of $k + 2$ terms from $a$ to $b$ and read off its step. For $k$ arithmetic means the common difference is $d = \dfrac{b - a}{k + 1}$, so the $j$th is $A_j = a + j\,d$. For $k$ geometric means the ratio is $r = \left(\dfrac{b}{a}\right)^{1/(k+1)}$, so $G_j = a\,r^j$. For $k$ harmonic means, insert $k$ arithmetic means between $\tfrac1a$ and $\tfrac1b$ and reciprocate.

---
## How the three means relate

For $a, b > 0$ with $A = \tfrac{a+b}{2}$, $G = \sqrt{ab}$, $H = \tfrac{2ab}{a+b}$:

**$G^2 = AH$.** Direct: $AH = \dfrac{a + b}{2}\cdot\dfrac{2ab}{a + b} = ab = G^2$. So $A, G, H$ are themselves a geometric progression.

**$A \ge G \ge H$.** The first inequality is $\dfrac{a + b}{2} \ge \sqrt{ab}$, which rearranges to $(\sqrt a - \sqrt b)^2 \ge 0$ — always true, with equality exactly when $a = b$. The second, $G \ge H$, is $\sqrt{ab} \ge \dfrac{2ab}{a + b}$, and multiplying out gives $a + b \ge 2\sqrt{ab}$ — the same statement again. Equality throughout iff $a = b$.

---
## Arithmetico-geometric progression

Multiply an A.P. term by term into a G.P. and you get an **A.G.P.**:
$$ S_n = a + (a + d)r + (a + 2d)r^2 + \cdots + [a + (n-1)d]\,r^{n-1} $$
Sum it with the same shift-and-subtract move as a G.P. Multiply by $r$, subtract, and the differences of the A.P. coefficients turn the middle into a plain geometric series of $n - 1$ terms:
$$ S_n(1 - r) = a + \frac{dr(1 - r^{n-1})}{1 - r} - [a + (n - 1)d]\,r^n $$
For $|r| < 1$, every $r^n$ term (including $n\,r^n$) vanishes as $n \to \infty$:
$$ S_\infty = \frac{a}{1 - r} + \frac{dr}{(1 - r)^2} $$

---
## Sums of powers

Standard closed forms for the first $n$ natural numbers, each provable by induction:
$$ \sum_{k=1}^{n} k = \frac{n(n + 1)}{2}, \qquad \sum_{k=1}^{n} k^2 = \frac{n(n + 1)(2n + 1)}{6}, \qquad \sum_{k=1}^{n} k^3 = \left[\frac{n(n + 1)}{2}\right]^2 $$
The last is the square of the first — the sum of cubes equals the square of the sum.

**Nested sums** with a dependent inner limit are done inside-out. For $S = \sum_{i=1}^{n}\sum_{j=1}^{i} j$, evaluate the inner sum first:
$$ \sum_{j=1}^{i} j = \frac{i(i + 1)}{2} \;\Longrightarrow\; S = \frac{1}{2}\sum_{i=1}^{n}(i^2 + i) = \frac{1}{2}\left[\frac{n(n+1)(2n+1)}{6} + \frac{n(n+1)}{2}\right] = \frac{n(n + 1)(n + 2)}{6} $$

---
## Means of $n$ numbers

For positive $x_1, \ldots, x_n$, define
$$ A_n = \frac{1}{n}\sum x_i, \qquad G_n = \left(\prod x_i\right)^{1/n}, \qquad H_n = \frac{n}{\sum 1/x_i} $$
The inequality generalises: $A_n \ge G_n \ge H_n$, equality iff all the $x_i$ are equal.

These are three points on one scale. The **power mean** of exponent $m$,
$$ M_m = \left(\frac{1}{n}\sum x_i^m\right)^{1/m} $$
is increasing in $m$: if $m > p$ then $M_m \ge M_p$. Here $M_1 = A_n$, $M_{-1} = H_n$, and $M_m \to G_n$ as $m \to 0$, so $A_n \ge G_n \ge H_n$ is just $M_1 \ge M_0 \ge M_{-1}$. Attaching positive weights $w_i$ gives the weighted versions $A_w = \dfrac{\sum w_i x_i}{\sum w_i}$, $G_w = \left(\prod x_i^{w_i}\right)^{1/\sum w_i}$, $H_w = \dfrac{\sum w_i}{\sum w_i/x_i}$, and $A_w \ge G_w \ge H_w$ still holds.

---
## The one idea to keep

Two ideas carry the weight here. First, an infinite geometric sum lives or dies by whether $|r| < 1$ — the prototype for every convergence test in [calculus](/citadel/maths/limits-derivatives). Second, $A \ge G \ge H$ is one of the most-used inequalities in mathematics: it bounds products by sums, proves that a fixed-perimeter rectangle has maximum area when square, and reappears throughout [inequality theory](/citadel/maths/equations-inequalities). The coefficient sums here also connect straight to the [binomial theorem](/citadel/maths/binomial-theory), where $\sum k\binom{n}{k}$ and its relatives are progressions in disguise.
