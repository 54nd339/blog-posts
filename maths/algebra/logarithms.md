---
title: Exponential and Logarithmic Functions - Growth, Decay, and Scale
description: A logarithm answers one question — what power gives this number? — and that question runs earthquake scales, pH, and binary search. The two functions, the laws that follow from one-to-one-ness, how to solve equations and inequalities in them, and how log tables once worked.
date: 2020-02-04
draft: false
slug: /maths/logarithms
tags:
  - Mathematics
  - Algebra
---

A logarithm answers one question: *what power do I raise this base to, to get that number?* $\log_{10}(1000) = 3$ because $10^3 = 1000$. Everything else — earthquake magnitudes, pH, the way your ear hears loudness, the running time of [binary search](/citadel/algorithms/SortingSearching) — is that question wearing different clothes.

Exponentials run the other way: fix the power, ask for the value. The two are inverses, so every fact about one is a fact about the other read backwards. This post covers both, the algebraic laws they obey, how to solve equations and inequalities where the unknown is trapped in an exponent or inside a log, and the characteristic-and-mantissa machinery that made log tables work before calculators.

---
## Exponential functions

An **exponential function** puts the variable in the exponent:
$$ f(x) = b^x $$
The constant $b$ is the **base**, with $b > 0$ and $b \neq 1$ (a negative base breaks fractional powers; $b = 1$ gives the constant $1$).

- $b > 1$: **growth**. $2^x$ doubles every step; $10^x$ multiplies by ten.
- $0 < b < 1$: **decay**. $(1/2)^x = 2^{-x}$ halves every step.

One base matters more than the rest: Euler's number $e \approx 2.71828$, giving the **natural exponential** $e^x$. It is the base for which the function equals its own rate of change, which is why it turns up the moment you [differentiate](/citadel/maths/limits-derivatives) anything that grows in proportion to its size — populations, compound interest, [radioactive decay](/citadel/physics/nuclear-physics).

The graph of $y = b^x$ has domain all of $\mathbb{R}$ and range $(0, \infty)$ — it never reaches zero. It crosses the $y$-axis at $(0, 1)$ (any base to the zero power is $1$), runs along the $x$-axis as a horizontal asymptote on one side, and passes through $(1, b)$. It rises for $b > 1$, falls for $0 < b < 1$.

### Laws of exponents

These are the rules for combining powers of the same base. Each one is just repeated multiplication counted carefully.

$$ b^x \cdot b^y = b^{x+y} \qquad \frac{b^x}{b^y} = b^{x-y} \qquad (b^x)^y = b^{xy} $$
$$ (ab)^x = a^x b^x \qquad \left(\frac{a}{b}\right)^x = \frac{a^x}{b^x} $$
$$ b^0 = 1 \qquad b^{-x} = \frac{1}{b^x} \qquad b^{x/y} = \sqrt[y]{b^x} $$

---
## Logarithmic functions

A **logarithm** is the exponential run backwards. If $b^y = x$, then $y$ is the logarithm of $x$ to base $b$:
$$ \log_b x = y \iff b^y = x $$
Read it aloud as "the power that turns $b$ into $x$". The base $b$ carries the same restriction ($b > 0$, $b \neq 1$), and the **argument** $x$ must be positive — no power of a positive base is ever zero or negative, so nothing else has a logarithm.

Two bases have their own notation:

- **Common logarithm**, base $10$, written $\log x$. The one behind the Richter, pH, and decibel scales, all of which compress a huge multiplicative range into a readable additive one.
- **Natural logarithm**, base $e$, written $\ln x$. The one that falls out of calculus.

Because $\log_b x$ undoes $b^x$, its graph is the reflection of $y = b^x$ across the line $y = x$. That swaps every feature: domain $(0, \infty)$, range all of $\mathbb{R}$, an $x$-intercept at $(1, 0)$ since $\log_b 1 = 0$, and the $y$-axis as a vertical asymptote. It passes through $(b, 1)$ and increases for $b > 1$, decreases for $0 < b < 1$.

![Three logarithm curves on one set of axes: log base 2 and the natural log both rise from bottom-left, slowly, and pass through (1, 0); a base-below-1 logarithm falls through the same point instead. All three hug the y-axis as a vertical asymptote.](../images/exp-log-graphs.png "log₂ x, ln x, and a base-less-than-1 logarithm. Every logarithm passes through (1, 0); a bigger base flattens the curve, a base below 1 flips it. Source: Wikimedia Commons.")

### Laws of logarithms

Each law is an exponent law seen through the mirror. The product rule, for instance, is just $b^m \cdot b^n = b^{m+n}$ with the exponents renamed.

$$ \log_b(xy) = \log_b x + \log_b y \qquad \log_b\!\left(\frac{x}{y}\right) = \log_b x - \log_b y $$
$$ \log_b(x^p) = p\log_b x \qquad \log_b x = \frac{\log_c x}{\log_c b} $$

The last is **change of base** — it lets a calculator that only knows $\ln$ and $\log$ compute a logarithm to any base, via $\log_b x = \ln x / \ln b$. Two more that are true by definition: $\log_b b = 1$ and $\log_b 1 = 0$; and the inverse pair $\log_b(b^x) = x$, $b^{\log_b x} = x$, which just say the two functions cancel.

---
## Solving equations

The unknown sits in an exponent or inside a log. Three moves cover almost every case.

**Match the bases.** Rewrite both sides as a power of one base, then equate the exponents — valid because $b^x$ is one-to-one.
$$ 2^{x+1} = 8 = 2^3 \implies x + 1 = 3 \implies x = 2 $$

**Take logs.** When the bases won't match, apply $\ln$ (or $\log$) to both sides and use the power rule to bring the variable down.
$$ 3^x = 5^{x-1} \implies x\ln 3 = (x-1)\ln 5 \implies x = \frac{\ln 5}{\ln 5 - \ln 3} $$

**Substitute.** If the equation is quadratic in $b^x$, say $A(b^x)^2 + B(b^x) + C = 0$, set $t = b^x$, solve the quadratic, then recover $x$ — discarding any $t \le 0$, since $b^x$ is always positive.

For logarithmic equations, combine terms into a single log with the laws, then convert to exponential form. The catch is that combining logs can *invent* solutions: $\log(x-3)$ has no meaning for $x = -1$, but $\log[(x-3)(x-1)]$ does. So fix the domain first and check every candidate against it.

$$ \log_2(x-3) + \log_2(x-1) = 3 $$

The domain needs $x - 3 > 0$ and $x - 1 > 0$, so $x > 3$. Combining and converting:
$$ \log_2[(x-3)(x-1)] = 3 \implies (x-3)(x-1) = 2^3 = 8 $$
$$ x^2 - 4x - 5 = 0 \implies (x-5)(x+1) = 0 $$
$x = 5$ clears the domain; $x = -1$ does not. The solution is $x = 5$.

---
## Solving inequalities

Everything from the equation case carries over, plus one rule: **the base decides whether the inequality flips.**

For exponentials, $b^x$ is increasing when $b > 1$ and decreasing when $0 < b < 1$. So
$$ b^{f(x)} > b^{g(x)} \implies \begin{cases} f(x) > g(x) & b > 1 \\ f(x) < g(x) & 0 < b < 1 \end{cases} $$
Taking logs works too, and the same rule applies to the log's base: $\log_c$ preserves the inequality for $c > 1$, reverses it for $0 < c < 1$.

For logarithmic inequalities, **pin down the domain first** — every argument strictly positive — then compare arguments with the same base rule:
$$ \log_b f(x) > \log_b g(x) \implies \begin{cases} f(x) > g(x) & b > 1 \\ f(x) < g(x) & 0 < b < 1 \end{cases} $$
and for $\log_b f(x) > c$, convert to $f(x) > b^c$ (base $> 1$) or $f(x) < b^c$ (base $< 1$). The final answer is the intersection of that result with the domain.

---
## Characteristic and mantissa

Before calculators, a multiplication like $345.6 \times 0.02$ was done by *adding* logarithms looked up in a table. That worked because of a split in the value of a common logarithm.

Write any positive $N$ in scientific notation, $N = m \times 10^p$ with $1 \le m < 10$ and $p$ an integer. Then
$$ \log_{10} N = \log_{10} m + p $$
The integer $p$ is the **characteristic** and $\log_{10} m$ is the **mantissa**.

- The **characteristic** places the decimal point. For $N > 1$ it is one less than the digit count before the point ($\log 345.6$: since $345.6 = 3.456 \times 10^2$, the characteristic is $2$). For $0 < N < 1$ it is negative, its magnitude one more than the number of zeros after the point before the first nonzero digit ($\log 0.03456 = \log(3.456 \times 10^{-2})$, characteristic $-2$).
- The **mantissa** $\log_{10} m$ lies in $[0, 1)$ and depends only on the *digits* of $N$, not the decimal point. $345.6$, $3.456$, and $34560$ all share the mantissa $0.5386$. That is what the table stored.

$$ \log_{10} 345.6 = \log_{10} 3.456 + 2 = 0.5386 + 2 = 2.5386 $$

For $0 < N < 1$ the logarithm is negative, e.g. $\log_{10} 0.02 \approx -1.69897$. To keep the mantissa positive for table use, the negative part is split off:
$$ -1.69897 = -2 + 0.30103 = \bar{2}.30103 $$
where $\bar{2}$ ("bar two") is a characteristic of $-2$ sitting next to a positive mantissa of $0.30103$.

Going back the other way is the **antilogarithm**: if $\log_b x = y$ then $x = b^y$. To recover $N$ from $\log N = C.M$, take the antilog of the mantissa (a number in $[1, 10)$, from an antilog table) and shift the decimal point by the characteristic: $N = (\text{antilog}_{10} M) \times 10^C$.

---
## The one idea to keep

A logarithm turns multiplication into addition and exponentiation into multiplication. That is why it collapses scales that span many orders of magnitude onto a line you can read, why it was the engine of hand computation for three centuries, and why $\ln$ is unavoidable once you start [integrating](/citadel/maths/integral-calculus) rates of change.
