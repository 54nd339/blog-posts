---
title: Trigonometry - Angles, Ratios, Identities, and Periodicity
description: Trigonometry starts as three ratios in a right triangle and turns into the language of anything that repeats. The six functions from triangle and unit circle, the identities worth knowing, how to write the general solution of a trig equation or inequality, and the inverse functions.
date: 2020-02-08
draft: false
slug: /maths/trigonometry
tags:
  - Mathematics
  - Trigonometry
---

Stand a ladder against a wall. Its angle to the ground fixes the ratio of every pair of sides — how high it reaches per metre of length, how far its foot sits from the wall. Those ratios are the sine, cosine, and tangent, and for a long time trigonometry was nothing more than a table of them for surveyors and astronomers.

The subject grew when someone let the angle keep turning past a right angle, past a full circle, into negative numbers. The ratios became functions of a real variable, and because a full turn brings you back to the start, they became *periodic* — which is why the same sine that measures a roof pitch also describes an AC voltage, a sound wave, and a planet's shadow. This post builds both pictures, collects the identities you actually reuse, and shows how to solve equations and inequalities where the unknown is an angle.

---
## Triangles and angles

A triangle has three sides and three angles summing to $\pi$. Sorted by sides it is **scalene** (all different), **isosceles** (two equal), or **equilateral** (all equal); by angles, **acute** (all $< 90°$), **right** (one $= 90°$), or **obtuse** (one $> 90°$). Two triangles are **congruent** — same shape and size — if they match on SSS, SAS, ASA, AAS, or RHS; **similar** — same shape, scaled — if they match on AA, SSS proportionality, or SAS proportionality. The right triangle is where the ratios start.

An **angle** measures rotation of a ray about its endpoint, from an initial side to a terminal side. Counter-clockwise is positive, clockwise negative. Two units:

- **Degrees**: a full turn is $360°$, subdivided as $1° = 60'$ and $1' = 60''$.
- **Radians**: the angle whose arc equals the radius. Since a circle's circumference is $2\pi r$, a full turn is $2\pi$ radians. So $\pi \text{ rad} = 180°$, giving $1 \text{ rad} \approx 57.296°$ and $1° \approx 0.01745 \text{ rad}$.

Place the angle in the plane with its initial side along the positive $x$-axis. The **quadrant** its terminal side lands in decides the signs of the ratios. **Coterminal** angles share a terminal side and differ by a whole number of full turns ($360°$ or $2\pi$).

---
## The six functions

For an acute angle $\theta$ in a right triangle, with sides named relative to $\theta$:
$$ \sin\theta = \frac{\text{opp}}{\text{hyp}} \qquad \cos\theta = \frac{\text{adj}}{\text{hyp}} \qquad \tan\theta = \frac{\text{opp}}{\text{adj}} = \frac{\sin\theta}{\cos\theta} $$
The other three are reciprocals:
$$ \csc\theta = \frac{1}{\sin\theta} \qquad \sec\theta = \frac{1}{\cos\theta} \qquad \cot\theta = \frac{1}{\tan\theta} = \frac{\cos\theta}{\sin\theta} $$

To let $\theta$ be *any* real number, switch to the **unit circle** $x^2 + y^2 = 1$. Put $\theta$ in standard position and let $P(x, y)$ be where its terminal side crosses the circle. Then
$$ \cos\theta = x \qquad \sin\theta = y \qquad \tan\theta = \frac{y}{x} $$
with $\sec\theta = 1/x$, $\csc\theta = 1/y$, $\cot\theta = x/y$ where the denominators are nonzero. For an acute angle this is the triangle definition with hypotenuse $1$; for any other angle it is the definition. Sine and cosine are the $y$- and $x$-coordinates of a point going around a circle — that is the whole reason they oscillate.

**Signs by quadrant.** All six are positive in quadrant I; then sine (and cosecant) in II, tangent (and cotangent) in III, cosine (and secant) in IV. The mnemonic is **All, Silver, Tea, Cups** (or *All Students Take Calculus*).

**Domain, range, period.**

| Function | Domain | Range | Period |
| :-- | :-- | :-- | :-- |
| $\sin\theta,\ \cos\theta$ | $\mathbb{R}$ | $[-1, 1]$ | $2\pi$ |
| $\tan\theta$ | $\mathbb{R} \setminus \{(n+\tfrac12)\pi\}$ | $\mathbb{R}$ | $\pi$ |
| $\cot\theta$ | $\mathbb{R} \setminus \{n\pi\}$ | $\mathbb{R}$ | $\pi$ |
| $\sec\theta$ | $\mathbb{R} \setminus \{(n+\tfrac12)\pi\}$ | $(-\infty,-1] \cup [1,\infty)$ | $2\pi$ |
| $\csc\theta$ | $\mathbb{R} \setminus \{n\pi\}$ | $(-\infty,-1] \cup [1,\infty)$ | $2\pi$ |

---
## Values you should know cold

The three standard angles, from the $30$–$60$–$90$ and $45$–$45$–$90$ triangles:

| $\theta$ | $0$ | $\pi/6$ | $\pi/4$ | $\pi/3$ | $\pi/2$ |
| :-- | :-- | :-- | :-- | :-- | :-- |
| $\sin\theta$ | $0$ | $\tfrac12$ | $\tfrac{1}{\sqrt2}$ | $\tfrac{\sqrt3}{2}$ | $1$ |
| $\cos\theta$ | $1$ | $\tfrac{\sqrt3}{2}$ | $\tfrac{1}{\sqrt2}$ | $\tfrac12$ | $0$ |
| $\tan\theta$ | $0$ | $\tfrac{1}{\sqrt3}$ | $1$ | $\sqrt3$ | — |

![The unit circle marked at every multiple of 30° and 45°, each point labelled with its angle in both degrees and radians and its (cos θ, sin θ) coordinates.](../images/unit-circle.png "The special angles around the unit circle. Each point's coordinates are (cos θ, sin θ); the first-quadrant values repeat with sign changes in the other three. Source: Wikimedia Commons.")

Every other angle you reduce to these using the **related-angle** identities. Complementary angles ($\theta$ and $\pi/2 - \theta$) swap each function with its cofunction:
$$ \sin(\tfrac{\pi}{2}-\theta) = \cos\theta \qquad \tan(\tfrac{\pi}{2}-\theta) = \cot\theta \qquad \sec(\tfrac{\pi}{2}-\theta) = \csc\theta $$
Reflections across the axes flip signs according to the quadrant:
$$ \sin(\pi - \theta) = \sin\theta \qquad \cos(\pi - \theta) = -\cos\theta \qquad \tan(\pi - \theta) = -\tan\theta $$
$$ \sin(\pi + \theta) = -\sin\theta \qquad \cos(\pi + \theta) = -\cos\theta \qquad \tan(\pi + \theta) = \tan\theta $$
$$ \sin(-\theta) = -\sin\theta \qquad \cos(-\theta) = \cos\theta \qquad \tan(-\theta) = -\tan\theta $$

---
## Identities

An **identity** holds for every value where both sides are defined. These are the ones that earn their keep.

**Pythagorean** — the circle equation $x^2 + y^2 = 1$ in disguise, then divided by $\cos^2\theta$ and by $\sin^2\theta$:
$$ \sin^2\theta + \cos^2\theta = 1 \qquad 1 + \tan^2\theta = \sec^2\theta \qquad 1 + \cot^2\theta = \csc^2\theta $$

**Compound angles** — the engine everything below is built from:
$$ \sin(A \pm B) = \sin A \cos B \pm \cos A \sin B $$
$$ \cos(A \pm B) = \cos A \cos B \mp \sin A \sin B $$
$$ \tan(A \pm B) = \frac{\tan A \pm \tan B}{1 \mp \tan A \tan B} $$

**Double angle** — set $A = B = \theta$ above:
$$ \sin 2\theta = 2\sin\theta\cos\theta \qquad \tan 2\theta = \frac{2\tan\theta}{1 - \tan^2\theta} $$
$$ \cos 2\theta = \cos^2\theta - \sin^2\theta = 2\cos^2\theta - 1 = 1 - 2\sin^2\theta $$

**Half angle** — solve the last two $\cos 2\theta$ forms for $\sin$ and $\cos$, with the sign set by the quadrant of $\theta/2$:
$$ \sin\frac{\theta}{2} = \pm\sqrt{\frac{1 - \cos\theta}{2}} \qquad \cos\frac{\theta}{2} = \pm\sqrt{\frac{1 + \cos\theta}{2}} \qquad \tan\frac{\theta}{2} = \frac{\sin\theta}{1 + \cos\theta} = \frac{1 - \cos\theta}{\sin\theta} $$

**Product ↔ sum** — add and subtract the compound-angle formulas:
$$ 2\sin A\cos B = \sin(A+B) + \sin(A-B) \qquad 2\cos A\cos B = \cos(A+B) + \cos(A-B) $$
$$ 2\sin A\sin B = \cos(A-B) - \cos(A+B) $$
$$ \sin C + \sin D = 2\sin\frac{C+D}{2}\cos\frac{C-D}{2} \qquad \cos C + \cos D = 2\cos\frac{C+D}{2}\cos\frac{C-D}{2} $$
$$ \sin C - \sin D = 2\cos\frac{C+D}{2}\sin\frac{C-D}{2} \qquad \cos C - \cos D = -2\sin\frac{C+D}{2}\sin\frac{C-D}{2} $$

There is a shorter road to all of them through [Euler's formula](/citadel/maths/complex-numbers) $e^{i\theta} = \cos\theta + i\sin\theta$: the compound-angle identities are just $e^{i(A+B)} = e^{iA}e^{iB}$ with real and imaginary parts read off.

---
## Solving trigonometric equations

An equation like $2\sin^2\theta - \sin\theta = 0$ has infinitely many solutions, because the functions repeat. Three terms name them:

- a **root** is any angle that satisfies the equation;
- the **principal solution** is the root in a chosen base interval, usually $[0, 2\pi)$;
- the **general solution** is a formula covering all roots, with an integer parameter $n \in \mathbb{Z}$.

Reduce the equation with identities until it reads $\sin\theta = \sin\alpha$ (or the same for $\cos$, $\tan$) for some known $\alpha$, then apply the pattern:
$$ \sin\theta = \sin\alpha \implies \theta = n\pi + (-1)^n\alpha $$
$$ \cos\theta = \cos\alpha \implies \theta = 2n\pi \pm \alpha $$
$$ \tan\theta = \tan\alpha \implies \theta = n\pi + \alpha $$

The boundary cases are worth memorising:
$$ \sin\theta = 0 \implies \theta = n\pi \qquad \cos\theta = 0 \implies \theta = (2n+1)\tfrac{\pi}{2} \qquad \tan\theta = 0 \implies \theta = n\pi $$
$$ \sin\theta = 1 \implies \theta = 2n\pi + \tfrac{\pi}{2} \qquad \cos\theta = 1 \implies \theta = 2n\pi \qquad \cos\theta = -1 \implies \theta = (2n+1)\pi $$

---
## Solving trigonometric inequalities

Same idea, one period at a time. Find where the two sides are equal, read off from the graph or the unit circle which stretch of angles satisfies the inequality inside one period, then add $2n\pi$ (or $n\pi$ for a $\pi$-periodic function). Substituting $t = \sin x$ and solving the algebraic inequality for $t$ first often helps.

For $\sin x > \tfrac12$: the boundary $\sin x = \tfrac12$ sits at $x = \pi/6$ and $x = 5\pi/6$ in $[0, 2\pi)$, and the sine graph is above $\tfrac12$ between them. So
$$ 2n\pi + \frac{\pi}{6} < x < 2n\pi + \frac{5\pi}{6}, \qquad n \in \mathbb{Z} $$

---
## Inverse trigonometric functions

To go from a ratio back to an angle you need an inverse — but $\sin$, $\cos$, $\tan$ each hit every value infinitely often, so they have no inverse until you restrict the domain to one stretch where the function is one-to-one. That restricted inverse takes the **principal value**.

| Inverse | Domain | Range (principal value) |
| :-- | :-- | :-- |
| $\arcsin x$ | $[-1, 1]$ | $[-\pi/2,\ \pi/2]$ |
| $\arccos x$ | $[-1, 1]$ | $[0,\ \pi]$ |
| $\arctan x$ | $\mathbb{R}$ | $(-\pi/2,\ \pi/2)$ |
| $\text{arccot}\,x$ | $\mathbb{R}$ | $(0,\ \pi)$ |
| $\text{arcsec}\,x$ | $(-\infty,-1] \cup [1,\infty)$ | $[0,\pi/2) \cup (\pi/2,\pi]$ |
| $\text{arccsc}\,x$ | $(-\infty,-1] \cup [1,\infty)$ | $[-\pi/2,0) \cup (0,\pi/2]$ |

Each graph is the reflection of its restricted parent across $y = x$. The composition $\sin(\arcsin x) = x$ always holds on the domain, but $\arcsin(\sin x) = x$ **only** when $x$ already lies in $[-\pi/2, \pi/2]$ — otherwise it returns the coterminal principal value. Handy symmetries: $\arcsin(-x) = -\arcsin x$ and $\arccos(-x) = \pi - \arccos x$.

---
## The one idea to keep

Every identity in the middle of this post is a consequence of two facts — a point on the unit circle satisfies $x^2 + y^2 = 1$, and rotating by $A + B$ is rotating by $A$ then by $B$. Learn the compound-angle formulas and the Pythagorean identity; the rest you can rebuild. The payoff is in [oscillations and waves](/citadel/physics/oscillations), where every periodic signal is a sum of these functions.
