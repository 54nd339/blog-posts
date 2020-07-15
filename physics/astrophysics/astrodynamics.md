---
title: Orbital Mechanics - Kepler's Laws and the Shape of an Orbit
description: The classical two-body problem worked through from the start — Kepler's three laws, Newton's inverse-square force, a first-principles derivation of orbit shape via Binet's equation, why every bound orbit is a conic section, how Kepler's third law falls out with its constant, the reduced-mass trick for binaries, and why planets seem to move backwards.
date: 2020-07-15
draft: false
slug: /physics/astrodynamics
tags:
  - Physics
  - Astrophysics
---

Kepler found, by grinding through twenty years of Tycho Brahe's position data, that planets move on ellipses. He never found *why*. The satisfying answer is that the inverse-square force does not merely permit elliptical orbits — it forbids anything else for a bound object. Turn the equation of motion into an equation for the orbit's shape and it becomes a driven harmonic oscillator (Binet's equation); the solution of a driven oscillator is a shifted cosine; and a shifted cosine, written in polar coordinates, is *exactly* the equation of a conic section. Ellipse, parabola, hyperbola — the three shapes a plane can cut from a cone are the three an inverse-square force allows, selected by the orbit's energy.

This post is the classical two-body problem start to finish: Kepler's three laws, Newton's force law, and that derivation of orbit shape. Where the Newtonian version quietly misses — Mercury's orbit, strong fields, light — is [general relativity](/citadel/physics/astrodynamics-advanced), the sequel.

## The classical picture: Kepler, then Newton

Kepler had the *what*; Newton found the *why*. Taking them in that order is also the historical order, and it shows how a pile of data becomes a law and then a mechanism.

### Kepler's three laws

In the early 1600s Johannes Kepler worked through decades of naked-eye position measurements inherited from Tycho Brahe and found three regularities in how the planets move around the Sun.

1. **The orbit is an ellipse.** Each planet traces an ellipse with the Sun at one focus, not a circle with the Sun at the centre. In polar coordinates centred on the Sun,
$$r = \frac{a(1 - e^2)}{1 + e \cos{\theta}}$$
where $r$ is the Sun–planet distance, $a$ is the semi-major axis (half the long diameter), $e$ is the eccentricity (how far from circular, with $e = 0$ a circle), and $\theta$ is measured from perihelion, the closest point to the Sun.

2. **Equal areas in equal times.** The line from the Sun to the planet sweeps out area at a constant rate, so the planet moves fast near perihelion and slow near aphelion. Writing the swept rate,
$$ \frac{dA}{dt} = \frac{1}{2}r^2\frac{d\theta}{dt} = \frac{1}{2}r^2\omega $$
and using the angular momentum $L = mr^2\omega$ for a planet of mass $m$,
$$ \frac{dA}{dt} = \frac{L}{2m} = \text{constant}. $$
A constant areal rate is the same statement as conserved angular momentum.

3. **Period squared goes as semi-major axis cubed.**
$$ T^2 \propto a^3 $$
A planet twice as far out takes $2^{3/2} \approx 2.83$ times as long to go round.

![Two elliptical planetary orbits sharing the Sun at focus F₁; for planet 1, two shaded sectors A₁ and A₂ swept in equal times are equal in area, and each orbit's semi-major axis a is marked.](../images/kepler-laws.png "Kepler's three laws in one figure: orbits are ellipses with the Sun at a focus (first); the Sun–planet line sweeps equal areas A₁ = A₂ in equal times (second); and T² ∝ a³ across planets (third). Source: Wikimedia Commons.")

```mermaid
graph LR
  D["Tycho's naked-eye<br/>position data"] --> K["Kepler's 3 laws<br/>(empirical)"]
  K --> N["Newton's inverse-square<br/>force law"]
  N --> B["Binet's equation<br/>d²u/dθ² + u = GMm²/L²"]
  B --> C["conic-section orbits<br/>+ Kepler's 3rd law with its constant"]
```

### Newton's law of gravitation

Kepler's laws are descriptions. Newton's law of universal gravitation is the mechanism they fall out of: every pair of masses attracts along the line between them, with a force proportional to each mass and to the inverse square of their separation,
$$ F = \frac{Gm_1m_2}{r^2} $$
with $G = 6.674 \times 10^{-11}\ \text{N}\,\text{m}^2/\text{kg}^2$. The same expression that makes an apple fall makes the Moon orbit; the only difference is the Moon's sideways speed. Everything below is a consequence of that one force law. The Newtonian groundwork — the field, potential energy, escape velocity, circular orbits — is in [gravitation](/citadel/physics/gravitation); here we go straight for the orbit shape.

## From the force law to the shape of an orbit

The two-body problem: one mass $m$ moving under the gravity of a mass $M$, interacting through nothing but their mutual pull. Take $M \gg m$ so the central body barely moves (or, exactly, read $r$ as the separation and work in the centre-of-mass frame — the reduced-mass section below makes this precise). In polar coordinates centred on $M$, the radial equation of motion is
$$ m(\ddot{r} - r\dot{\theta}^2) = -\frac{GMm}{r^2} $$
and dividing through by $m$,
$$ \ddot{r} - r\dot{\theta}^2 = -\frac{GM}{r^2} \quad (*) $$
where $\ddot{r}$ is the radial acceleration, $r\dot{\theta}^2$ the centripetal term, and $\dot{\theta} = d\theta/dt$. We want the *shape* of the path — $r$ as a function of $\theta$, with time eliminated.

Two standard moves do it. First, angular momentum is conserved for a central force, so $L = mr^2\dot{\theta}$ is constant and $\dot{\theta} = L/(mr^2)$. Second, substitute $u = 1/r$.

Convert the time derivatives into $\theta$ derivatives. For $\dot{r}$, using $\dot{\theta} = L/(mr^2) = Lu^2/m$ and $dr/d\theta = -\frac{1}{u^2}\,du/d\theta$:
$$ \dot{r} = \frac{dr}{d\theta}\,\dot{\theta} = \left(-\frac{1}{u^2}\frac{du}{d\theta}\right)\left(\frac{Lu^2}{m}\right) = -\frac{L}{m}\frac{du}{d\theta} $$
Differentiate again, with $\dfrac{d}{dt} = \dot{\theta}\,\dfrac{d}{d\theta} = \dfrac{Lu^2}{m}\dfrac{d}{d\theta}$:
$$ \ddot{r} = \frac{Lu^2}{m}\frac{d}{d\theta}\left(-\frac{L}{m}\frac{du}{d\theta}\right) = -\frac{L^2u^2}{m^2}\frac{d^2u}{d\theta^2} $$
Put $\ddot{r}$, $r = 1/u$, and $\dot{\theta} = Lu^2/m$ into $(*)$:
$$ -\frac{L^2u^2}{m^2}\frac{d^2u}{d\theta^2} - \frac{1}{u}\left(\frac{Lu^2}{m}\right)^2 = -GMu^2 $$
$$ -\frac{L^2u^2}{m^2}\frac{d^2u}{d\theta^2} - \frac{L^2u^3}{m^2} = -GMu^2 $$
Divide by $-L^2u^2/m^2$:
$$ \frac{d^2u}{d\theta^2} + u = \frac{GMm^2}{L^2} $$

This is **Binet's equation**: a linear oscillator equation with a constant drive. Its general solution is
$$ u(\theta) = \frac{GMm^2}{L^2} + A\cos(\theta - \theta_0) $$
Orient the axes so $\theta_0 = 0$, and switch back to $r = 1/u$:
$$ r = \frac{L^2/(GMm^2)}{1 + \left(AL^2/(GMm^2)\right)\cos{\theta}} $$
That is the polar equation of a conic section. Matching it term by term against Kepler's first law $r = \dfrac{a(1-e^2)}{1+e\cos\theta}$ gives the semi-latus rectum $p = \dfrac{L^2}{GMm^2}$ and the eccentricity $e = A\,\dfrac{L^2}{GMm^2}$.

So the force law does not merely *allow* ellipses; it forces every bound orbit to be one, and the eccentricity (which depends on the orbit's energy) fixes which conic you get:

- $e = 0$: circle
- $0 < e < 1$: ellipse — the planets
- $e = 1$: parabola — a comet on a single pass
- $e > 1$: hyperbola — an interstellar object passing through

### Kepler's third law falls out

The same solution gives the third law with its constant filled in. An ellipse has area $\pi a b$ with semi-minor axis $b = a\sqrt{1-e^2}$, and the areal rate is $dA/dt = L/(2m)$, so one full orbit takes
$$ T = \frac{\pi a b}{L/(2m)} = \frac{2m\pi a b}{L}, \qquad T^2 = \frac{4m^2\pi^2 a^2 b^2}{L^2}. $$
Use $L^2 = GMm^2\,a(1-e^2)$ (from $p$ above) and $b^2 = a^2(1-e^2)$:
$$ T^2 = \frac{4m^2\pi^2 a^2 \cdot a^2(1-e^2)}{GMm^2\,a(1-e^2)} = \frac{4\pi^2}{GM}\,a^3. $$
The proportionality constant $4\pi^2/GM$ depends only on the central mass, which is why one constant works for every planet around the Sun.

### Binary systems: reduced mass

When both bodies have comparable mass — a binary star — neither sits still; both orbit the shared centre of mass. Define the **reduced mass** $\mu = \dfrac{m_1 m_2}{m_1 + m_2}$. The equation for the separation vector $\vec{r} = \vec{r_2} - \vec{r_1}$ is
$$ \mu\,\frac{d^2\vec{r}}{dt^2} = -\frac{Gm_1 m_2}{r^2}\,\hat{r} $$
which is the one-body problem again with $m \to \mu$ and the gravitational parameter set by the *total* mass, $G(m_1 + m_2)$. Every result above carries over under those replacements: in Binet's equation the constant $GMm^2/L^2$ becomes one built from $\mu$ and $m_1 m_2$, and the period relation runs on $G(m_1 + m_2)$ rather than $GM$.

### Apparent retrograde motion

A planet sometimes appears to stop against the background stars and move backwards for a few weeks. Nothing reverses — it is a parallax effect of watching one moving planet from another. The angular rate you observe is the difference of two orbital rates,
$$ \frac{d\theta_{\text{observed}}}{dt} = \frac{d\theta_{\text{planet}}}{dt} - \frac{d\theta_{\text{Earth}}}{dt} $$
and when Earth overtakes an outer planet (or is lapped by an inner one), that difference goes briefly negative.

## Where Newton runs out

Newton's gravity is enough to fly a probe to Mars. It breaks in three regimes: very strong fields, speeds near light, and the universe treated as a whole. The clearest crack is Mercury: the long axis of its orbit slowly rotates, and after subtracting every known gravitational tug from the other planets, a small residual precession of about 43 arcseconds per century is left that the inverse-square law cannot account for. Fixing it does not mean patching the force law — [general relativity](/citadel/physics/astrodynamics-advanced) replaces the idea that gravity is a force at all, and in the same move predicts black holes, gravitational lensing, and gravitational waves.

## The one idea to keep

Convert Newton's radial equation into an equation for $u = 1/r$ as a function of angle, and it becomes Binet's equation — a harmonic oscillator with a constant drive. Its solution is a shifted cosine, which in polar form is the equation of a conic section, so the inverse-square force *compels* every orbit to be an ellipse, parabola, or hyperbola, with the choice set by energy. The same solution delivers Kepler's second law (it is conservation of angular momentum) and his third with the constant filled in, $T^2 = 4\pi^2 a^3/GM$. For two comparable masses, replace $m$ with the reduced mass $\mu$ and $M$ with the total, and everything carries over.
