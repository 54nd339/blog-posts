---
title: Fuzzy Set Theory - Degrees of Membership and Fuzzy Inference
description: How to reason with vague categories - fuzzy sets and their membership functions, the set operations built on min and max, the standard parameterised membership shapes, fuzzy relations and composition, fuzzy if-then rules, and the Mamdani and Sugeno inference systems that turn linguistic rules into crisp outputs.
date: 2023-02-10
draft: false
slug: /artificial-intelligence/fuzzy-theory
tags:
  - Artificial Intelligence
  - Fuzzy Logic
  - Probabilistic Models
---

In classical set theory a person is tall or not tall — you pick a cutoff, say 180 cm, and 179 cm is simply short. That's a bad model of how "tall" actually works. **Fuzzy set theory**, introduced by Lotfi Zadeh in 1965, replaces the in-or-out membership of a set with a *degree* of membership between 0 and 1. A person 196 cm has membership 1.0 in `Tall`, someone 178 cm has membership 0.7, someone 152 cm has 0.1.

That one change — a set is a function to $[0, 1]$ instead of a subset — carries all the way to working control systems. This post follows it from the definition through to a fuzzy inference system that takes crisp sensor readings and produces a crisp control output.

## Fuzzy sets and their vocabulary

A **fuzzy set** $A$ over a **universe of discourse** $X$ (all the elements in play) is a set of pairs

$$A = \{(x, \mu_A(x)) \mid x \in X\}$$

where the **membership function** $\mu_A : X \to [0, 1]$ gives each element's degree of membership. $\mu_A(x) = 1$ is full membership, $0$ is none, anything between is partial. A crisp set is the special case where $\mu_A$ only ever returns 0 or 1.

The terms you need to read fuzzy-logic papers:

- **Support** — $\{x \mid \mu_A(x) > 0\}$, the elements with any membership at all.
- **Core** — $\{x \mid \mu_A(x) = 1\}$, the full members.
- **Normal** — the core is non-empty (at least one element has membership 1).
- **Crossover points** — where $\mu_A(x) = 0.5$; the points of maximum fuzziness.
- **Fuzzy singleton** — support is a single point, with membership 1 there.
- **$\alpha$-cut** $A_\alpha$ — the crisp set $\{x \mid \mu_A(x) \ge \alpha\}$. The **strong** $\alpha$-cut uses $> \alpha$.
- **Convex** — $\mu_A(\lambda x_1 + (1-\lambda) x_2) \ge \min(\mu_A(x_1), \mu_A(x_2))$ for $\lambda \in [0, 1]$; the membership function has one hump and no dips.
- **Bandwidth** — for a normal convex set, the distance between its two crossover points.
- **Symmetric** — $\mu_A(c + x) = \mu_A(c - x)$ around some centre $c$.

## Operations

Fuzzy set operations reduce to arithmetic on membership grades. The three standard ones:

$$\mu_{A \cup B}(x) = \max(\mu_A(x), \mu_B(x)) \qquad \mu_{A \cap B}(x) = \min(\mu_A(x), \mu_B(x)) \qquad \mu_{\lnot A}(x) = 1 - \mu_A(x)$$

`max` is the standard **t-conorm** (fuzzy OR); `min` is the standard **t-norm** (fuzzy AND). Other choices are valid:

- **Algebraic product** — $\mu_A(x)\,\mu_B(x)$, another t-norm.
- **Algebraic (probabilistic) sum** — $\mu_A(x) + \mu_B(x) - \mu_A(x)\,\mu_B(x)$, another t-conorm.
- **Bounded sum** — $\min(1,\ \mu_A(x) + \mu_B(x))$; **bounded difference** — $\max(0,\ \mu_A(x) + \mu_B(x) - 1)$.
- **Power** $A^\alpha$ — $(\mu_A(x))^\alpha$. With $\alpha > 1$ this is **concentration** ("very tall" sharpens the set); with $\alpha < 1$, **dilation** ("somewhat tall" widens it).
- **Cartesian product** — $\mu_{A \times B}(x, y) = \min(\mu_A(x), \mu_B(y))$, which produces a fuzzy relation.

Fuzzy sets keep commutativity, associativity, distributivity, idempotence of min/max, involution, and De Morgan's laws.

## Membership function shapes

For a continuous universe you define $\mu_A$ with a parameterised curve. The common ones:

**Triangular**, parameters $\{a, b, c\}$ (feet at $a$ and $c$, peak at $b$):

$$f(x; a, b, c) = \max\!\left(0,\ \min\!\left(\tfrac{x-a}{b-a},\ \tfrac{c-x}{c-b}\right)\right)$$

**Trapezoidal**, $\{a, b, c, d\}$ (feet at $a$ and $d$, flat top over $[b, c]$):

$$f(x; a, b, c, d) = \max\!\left(0,\ \min\!\left(\tfrac{x-a}{b-a},\ 1,\ \tfrac{d-x}{d-c}\right)\right)$$

**Gaussian**, centre $c$ and width $\sigma$: $\;f(x; c, \sigma) = e^{-\frac{1}{2}\left(\frac{x - c}{\sigma}\right)^2}$

**Generalised bell**, $\{a, b, c\}$ ($a$ width, $b$ slope, $c$ centre): $\;f(x; a, b, c) = \dfrac{1}{1 + \left|\frac{x - c}{a}\right|^{2b}}$

**Sigmoidal**, slope $a$ and crossover $c$: $\;f(x; a, c) = \dfrac{1}{1 + e^{-a(x - c)}}$ — an S-curve for "large" or "small".

## Fuzzy relations and composition

A **fuzzy relation** $R$ from $X$ to $Y$ is a fuzzy set on $X \times Y$; $\mu_R(x, y)$ is the strength of the link between $x$ and $y$ ("$x$ is *slightly larger than* $y$").

Relations compose. If $R$ runs $X \to Y$ and $S$ runs $Y \to Z$, the **max-min composition** $T = R \circ S$ runs $X \to Z$:

$$\mu_{R \circ S}(x, z) = \max_{y \in Y} \min\bigl(\mu_R(x, y),\ \mu_S(y, z)\bigr)$$

The **extension principle** is the companion idea: it says how an ordinary crisp function $y = f(x_1, \dots, x_n)$ acts on fuzzy input sets to produce a fuzzy output set, by taking, for each output value, the best membership over all input tuples that map to it.

## Fuzzy if-then rules

Expert knowledge goes into rules of the form

`IF temperature IS high AND humidity IS high THEN fan_speed IS very_fast`

where `high` and `very_fast` are fuzzy sets. A rule `IF x IS A THEN y IS B` defines a fuzzy relation $R$ between $A$ and $B$, and there are several ways to build it:

- **Mamdani (min) implication** — $\mu_R(x, y) = \min(\mu_A(x), \mu_B(y))$. The common choice.
- **Larsen (product) implication** — $\mu_R(x, y) = \mu_A(x)\,\mu_B(y)$.
- **Material implication** ($\lnot A \lor B$) — $\mu_R(x, y) = \max(1 - \mu_A(x),\ \mu_B(y))$.
- **Zadeh's max-min rule** — $\mu_R(x, y) = \max\bigl(\min(\mu_A(x), \mu_B(y)),\ 1 - \mu_A(x)\bigr)$.

## Fuzzy reasoning

Given rules and a fuzzy fact, you infer a fuzzy conclusion by **generalised modus ponens**:

- Rule: IF $x$ is $A$ THEN $y$ is $B$
- Fact: $x$ is $A'$ (possibly not exactly $A$)
- Conclusion: $y$ is $B'$, computed as $B' = A' \circ R$ with the max-min composition.

The classical case ($A' = A$) recovers $B' = B$; a partial match gives a partial conclusion.

## Fuzzy inference systems

A **fuzzy inference system** (FIS) maps crisp inputs to crisp outputs through four stages:

1. **Fuzzification** — convert each crisp input to membership grades in the relevant fuzzy sets.
2. **Rule base** — the collection of fuzzy if-then rules.
3. **Inference engine** — compute each rule's firing strength (a t-norm over its antecedent memberships), apply it to the consequent, and aggregate all rules' outputs.
4. **Defuzzification** — collapse the aggregated fuzzy output to a single crisp value (e.g. centroid).

Three standard models differ in the shape of the consequent:

- **Mamdani** — antecedents and consequents are both fuzzy sets. Each rule's firing strength clips (or scales) its consequent set; the clipped sets are aggregated with `max`; the result is defuzzified. Intuitive and the usual choice for expressing human knowledge.
- **Sugeno (TSK)** — antecedents are fuzzy, but each consequent is a crisp function of the inputs, usually linear: `IF x₁ IS A₁ AND x₂ IS A₂ THEN y = p₁x₁ + p₂x₂ + r`. The output is the firing-strength-weighted average of the rules' crisp outputs — no separate defuzzification step. Efficient, and it fits adaptive tuning, which is why [ANFIS](/citadel/artificial-intelligence/neuro-fuzzy) is built on it.
- **Tsukamoto** — each consequent is a fuzzy set with a *monotonic* membership function, so each rule yields a crisp value directly; the system output is again the firing-strength-weighted average.

## The one idea to keep

Fuzzy logic keeps the *structure* of classical logic — sets, connectives, modus ponens, if-then rules — and swaps in a membership function to $[0, 1]$ for the characteristic function. `AND` becomes `min`, `OR` becomes `max`, and a rule becomes a relation. That's enough to build a system that reasons from linguistic rules a human can read and still outputs the crisp number a motor needs.
