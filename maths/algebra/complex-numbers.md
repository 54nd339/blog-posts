---
title: Complex Numbers - Algebra, Geometry, and Rotation
description: A complex number is a point in a plane, and multiplying by one rotates and scales. That single picture explains the algebra, De Moivre's theorem, the roots of unity, and why complex numbers run quantum mechanics and AC circuits.
date: 2020-02-02
draft: false
slug: /maths/complex-numbers
tags:
  - Mathematics
  - Algebra
---

The equation $x^2 = -1$ has no real solution, because squaring any real number gives something $\ge 0$. Invent a symbol $i$ with $i^2 = -1$ and the equation has a solution — and, it turns out, so does *every* polynomial equation. That is the payoff: adjoining one new number closes arithmetic forever.

The other half of the story is geometric. Write $i$ as a point one unit up the vertical axis, and every complex number becomes a point in a plane. Addition is vector addition; multiplication by a fixed complex number rotates and scales the whole plane. This post builds both views — the algebra of $a + ib$ and the geometry of the Argand diagram — and shows how they make powers, roots, and plane geometry fall out of one formula, $e^{i\theta} = \cos\theta + i\sin\theta$.

---
## Where $i$ comes from

Not from square roots of negatives — mathematicians dismissed those as nonsense for centuries. It came from **cubics**. Cardano's formula for the roots of $x^3 = px + q$ sometimes runs through the square root of a negative number *even when all three roots are real and rational*. Bombelli, around 1572, pushed the symbols through anyway, let $\sqrt{-1}$ cancel itself in the final step, and got the right real answers. The "impossible" quantity was a working part of a machine that produced real results, so it earned a place.

---
## The imaginary unit and standard form

The **imaginary unit** $i$ is defined by
$$ i^2 = -1 $$
Its powers cycle with period four: $i^1 = i$, $i^2 = -1$, $i^3 = -i$, $i^4 = 1$, then it repeats. To evaluate $i^n$, divide $n$ by 4 and use the remainder — $i^{23} = i^{20}\cdot i^3 = -i$.

A **complex number** is
$$ z = a + ib, \qquad a, b \in \mathbb{R} $$
with $a = \operatorname{Re}(z)$ the **real part** and $b = \operatorname{Im}(z)$ the **imaginary part** (itself a real number — it is the coefficient of $i$, not $ib$). If $b = 0$, $z$ is real; if $a = 0$ and $b \neq 0$, $z$ is *purely imaginary*.

If the definition $i = \sqrt{-1}$ feels circular, there is a cleaner one. Take $z$ to be an **ordered pair** $(a, b)$ of reals, with
$$ (a,b) + (c,d) = (a+c,\; b+d), \qquad (a,b)\cdot(c,d) = (ac - bd,\; ad + bc) $$
Then pairs $(a, 0)$ add and multiply exactly like the reals $a$, and $(0,1)\cdot(0,1) = (-1, 0)$ — so the pair $(0,1)$ *is* a number whose square is $-1$. Call it $i$ and write $(a,b)$ as $a + ib$. No leap of faith required; $i$ is just a second coordinate.

---
## The complex plane

There is no room for $i$ on the real line, so complex numbers live on a plane: the **Argand diagram**, horizontal *real axis*, vertical *imaginary axis*. The number $z = a + ib$ is the point $(a, b)$, or equally the arrow from the origin to that point.

![The complex plane with axes labelled Re and Im. The number z = a + bi is plotted as a point, reached by an arrow from the origin O; a dotted line drops to a on the real axis and across to b on the imaginary axis.](../images/argand-diagram.png "A complex number z = a + bi as a point in the plane: its real part a is the horizontal coordinate, its imaginary part b the vertical one. Source: Wikimedia Commons.")

Adding two complex numbers is adding their arrows tip-to-tail. Everything below is about what *multiplication* does to this picture.

---
## Arithmetic

**Equality.** $a + ib = c + id$ exactly when $a = c$ *and* $b = d$. One complex equation is two real equations.

**Addition and subtraction** act componentwise:
$$ (a+ib) \pm (c+id) = (a \pm c) + i(b \pm d) $$

**Multiplication** is ordinary expansion with $i^2 = -1$ folded in:
$$ (a+ib)(c+id) = ac + iad + ibc + i^2 bd = (ac - bd) + i(ad + bc) $$

**Division** uses a trick: multiply top and bottom by the *conjugate* of the denominator (next section), which makes the denominator real.
$$ \frac{a+ib}{c+id} = \frac{(a+ib)(c-id)}{(c+id)(c-id)} = \frac{(ac+bd) + i(bc-ad)}{c^2 + d^2} $$

---
## The conjugate

The **conjugate** of $z = a + ib$ is
$$ \bar{z} = a - ib $$
— geometrically, the reflection of $z$ across the real axis. Its usefulness is one identity:
$$ z\bar{z} = (a+ib)(a-ib) = a^2 + b^2 $$
a *real*, non-negative number. That is what cleared the denominator in the division rule, and it is the bridge to the modulus below.

Conjugation passes through every operation: $\overline{z_1 + z_2} = \bar{z}_1 + \bar{z}_2$, $\overline{z_1 z_2} = \bar{z}_1\,\bar{z}_2$, $\overline{z^n} = (\bar{z})^n$. Also $z + \bar{z} = 2\operatorname{Re}(z)$ and $z - \bar{z} = 2i\operatorname{Im}(z)$, and $z = \bar{z}$ precisely when $z$ is real.

---
## Modulus and argument

Instead of coordinates $(a, b)$, describe $z$ by *how far* and *which way*.

The **modulus** is the distance from the origin:
$$ |z| = \sqrt{a^2 + b^2} = \sqrt{z\bar{z}} $$
It behaves multiplicatively: $|z_1 z_2| = |z_1|\,|z_2|$, $|z_1/z_2| = |z_1|/|z_2|$, $|z^n| = |z|^n$, and it obeys the triangle inequality $|z_1 + z_2| \le |z_1| + |z_2|$ (the arrow to $z_1 + z_2$ is no longer than the two arrows laid end to end).

The **argument** $\arg(z)$ is the angle the arrow makes with the positive real axis, counterclockwise. You get it from $\tan\theta = b/a$ *plus* the quadrant of $(a, b)$ — $\arg(1+i) = \pi/4$ but $\arg(-1-i) = -3\pi/4$, not $\pi/4$. It is only defined up to adding $2\pi k$; pinning it to $(-\pi, \pi]$ gives the **principal argument** $\operatorname{Arg}(z)$. Arguments *add* under multiplication: $\arg(z_1 z_2) = \arg(z_1) + \arg(z_2)$, and $\arg(z^n) = n\arg(z)$, both mod $2\pi$.

---
## Polar and Euler form

Read $a = r\cos\theta$ and $b = r\sin\theta$ straight off the [right triangle](/citadel/maths/trigonometry) in the Argand diagram, with $r = |z|$ and $\theta = \arg(z)$. Then
$$ z = r(\cos\theta + i\sin\theta) $$
This is the **polar form**. Euler's formula compresses the bracket:
$$ e^{i\theta} = \cos\theta + i\sin\theta \quad\Longrightarrow\quad z = re^{i\theta} $$
It is not a definition pulled from nowhere — expand $e^{i\theta}$, $\cos\theta$, and $\sin\theta$ as power series and the identity is term by term. What it buys you is that multiplication becomes exponent arithmetic:
$$ z_1 z_2 = r_1 r_2\, e^{i(\theta_1 + \theta_2)}, \qquad \frac{z_1}{z_2} = \frac{r_1}{r_2}\, e^{i(\theta_1 - \theta_2)} $$
**Multiplying by $z$ scales every point by $|z|$ and rotates it by $\arg(z)$.** Multiplying by $i$ (modulus 1, argument $\pi/2$) is a quarter-turn — which is why $i^2 = -1$ is a half-turn, i.e. negation.

---
## De Moivre's theorem

Apply the multiplication rule $n$ times to a number of modulus 1:
$$ (\cos\theta + i\sin\theta)^n = \cos(n\theta) + i\sin(n\theta) $$
In Euler form it is just $(e^{i\theta})^n = e^{in\theta}$. For a general $z = re^{i\theta}$,
$$ z^n = r^n\big(\cos(n\theta) + i\sin(n\theta)\big) = r^n e^{in\theta} $$
Beyond computing powers, it is an identity factory: expand $(\cos\theta + i\sin\theta)^3$ by the binomial theorem, match real and imaginary parts, and out drop $\cos 3\theta = 4\cos^3\theta - 3\cos\theta$ and $\sin 3\theta = 3\sin\theta - 4\sin^3\theta$.

---
## Roots

To find the $n$th roots of $z = r(\cos\theta + i\sin\theta)$ — the $w$ with $w^n = z$ — write $w = s(\cos\psi + i\sin\psi)$. Then $w^n = s^n(\cos n\psi + i\sin n\psi)$, so matching modulus and angle:
$$ s^n = r \;\Rightarrow\; s = r^{1/n}, \qquad n\psi = \theta + 2k\pi \;\Rightarrow\; \psi_k = \frac{\theta + 2k\pi}{n} $$
The $2k\pi$ matters: because sine and cosine repeat, the angle is only fixed mod $2\pi$, and dividing by $n$ turns those copies into $n$ *distinct* answers.
$$ w_k = r^{1/n}\left[\cos\!\left(\frac{\theta + 2k\pi}{n}\right) + i\sin\!\left(\frac{\theta + 2k\pi}{n}\right)\right], \qquad k = 0, 1, \ldots, n-1 $$
They sit at equal spacing $2\pi/n$ around a circle of radius $r^{1/n}$ — the vertices of a regular $n$-gon.

---
## Roots of unity

Take $z = 1$, so $r = 1$ and $\theta = 0$. The **$n$th roots of unity** are
$$ \omega_k = \cos\!\left(\frac{2k\pi}{n}\right) + i\sin\!\left(\frac{2k\pi}{n}\right) = e^{i\,2k\pi/n}, \qquad k = 0, 1, \ldots, n-1 $$
— a regular $n$-gon inscribed in the unit circle with one vertex nailed at $1$. Two facts fall out of the geometry: they **sum to zero** for $n > 1$ (the polygon's centroid is the origin), and they **multiply to $(-1)^{n-1}$**.

![The fifth roots of unity plotted on the unit circle in the complex plane: five points equally spaced 72 degrees apart, one at (1, 0), forming a regular pentagon.](../images/roots-of-unity.png "The fifth roots of unity: solutions of z⁵ = 1, equally spaced around the unit circle. Source: Wikimedia Commons.")

The **cube roots of unity** ($z^3 = 1$) are $1$, $\omega = -\tfrac12 + i\tfrac{\sqrt3}{2}$, and $\omega^2 = -\tfrac12 - i\tfrac{\sqrt3}{2}$. They satisfy $\omega^3 = 1$, $1 + \omega + \omega^2 = 0$, and $\omega^2 = \bar{\omega}$ — the trio that simplifies countless factorisations, e.g. $a^3 + b^3 = (a+b)(a+\omega b)(a + \omega^2 b)$.

---
## Geometry in the complex plane

Because $|z - z_0|$ *is* the distance between the points $z$ and $z_0$, geometric conditions become one-line equations in $z$:

| Condition on $z$ | Locus |
| --- | --- |
| $\lvert z - z_0 \rvert = R$ | circle, centre $z_0$, radius $R$ |
| $\lvert z - z_1 \rvert = \lvert z - z_2 \rvert$ | perpendicular bisector of $z_1 z_2$ |
| $\lvert z - z_1 \rvert + \lvert z - z_2 \rvert = 2a$ | ellipse, foci $z_1, z_2$ |
| $\big\lvert\, \lvert z - z_1 \rvert - \lvert z - z_2 \rvert \,\big\rvert = 2a$ | hyperbola, foci $z_1, z_2$ |
| $\arg\!\big(\frac{z - z_1}{z - z_2}\big) = \alpha$ | arc of a circle through $z_1, z_2$ (a diameter when $\alpha = \pm\pi/2$) |

**Rotation about a point.** The number $\dfrac{z_3 - z_1}{z_2 - z_1}$ is the arrow $z_1 \to z_3$ measured against the arrow $z_1 \to z_2$; its argument is the angle $\angle z_2 z_1 z_3$ and its modulus the ratio of side lengths. Setting that expression purely real is the test for $z_1, z_2, z_3$ **collinear**.

**Section formula.** The point dividing $z_1 z_2$ in ratio $m : n$ is $z = \dfrac{m z_2 + n z_1}{m + n}$ (internal) or $\dfrac{m z_2 - n z_1}{m - n}$ (external); the midpoint is $\tfrac12(z_1 + z_2)$.

**Line.** Parametrically $z = (1-t)z_1 + t z_2$ for real $t$; in closed form $\bar{a}z + a\bar{z} + b = 0$ with $a$ a nonzero complex constant and $b$ real.

**Circle, general form.** Expanding $|z - z_0|^2 = R^2$ gives $z\bar{z} + \bar{a}z + a\bar{z} + b = 0$ with $a = -z_0$ and $b = |z_0|^2 - R^2$ real; reading it backwards, the centre is $-a$ and the radius $\sqrt{|a|^2 - b}$.

---
## The one idea to keep

A complex number is a scaling-and-rotation waiting to be applied. That is why alternating current is bookkept as $re^{i\omega t}$ in [electrical engineering](/citadel/physics/ac-circuits), why a quantum state's phase is a unit complex number, and why iterating $z \mapsto z^2 + c$ in the plane draws the Mandelbrot set. Push the calculus of these functions further — derivatives, contour integrals, the fact that being differentiable *once* forces infinitely many derivatives — and you are doing [complex analysis](/citadel/maths/complex-analysis).
