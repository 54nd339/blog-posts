---
title: Special Relativity - Lorentz Transformations, Spacetime, and E = mc²
description: "From two postulates to a rebuilt kinematics: the Lorentz transformation derived from the invariance of the interval, time dilation and length contraction with their thought experiments, proper time, the four-vector formalism, the energy-momentum relation, the relativistic Doppler effect, E = mc² from a recoil argument, and how to read a Minkowski diagram."
date: 2020-10-09
draft: false
slug: /physics/relative-mech
tags:
  - Physics
  - Relativity
---

By 1900, [Maxwell's equations](/citadel/physics/electromag) said light travels at a fixed speed $c$ set by two constants of the vacuum — with no mention of who is measuring. That flatly contradicts the classical rule that velocities add. Einstein's 1905 resolution was to keep the constancy of $c$ and give up instead on absolute time and length. This post derives the consequences: the Lorentz transformation, the two famous distortions of time and space, the four-vector machinery that makes it all tidy, the energy-momentum relation, and $E = mc^2$.

## Galilean relativity, and why it fails

Classical mechanics already has a relativity principle: the laws of motion are identical in every **inertial frame** — any frame at rest or in uniform motion. Relating a frame $S$ to a frame $S'$ moving at speed $v$ along the shared $x$-axis, with origins coinciding at $t = t' = 0$, the **Galilean transformation** is
$$ x' = x - vt, \qquad y' = y, \qquad z' = z, \qquad t' = t $$
Time is universal, and velocities add: an object moving at $u'$ in $S'$ moves at $u = u' + v$ in $S$. Apply that to a light pulse and $S$ and $S'$ get different speeds for it — which experiment (Michelson–Morley) says never happens.

## The two postulates

1. **Principle of relativity.** The laws of physics — all of them, electromagnetism included — take the same form in every inertial frame.
2. **Constancy of light speed.** Light travels through vacuum at $c$ as measured in every inertial frame, whatever the motion of the source or the observer.

The second is the radical one: approaching or receding, a source's light is still measured at $c$. Keeping it forces space and time measurements to transform in a new way.

## The Lorentz transformation

A flash goes off at the coinciding origins at $t = t' = 0$. Its wavefront is a sphere expanding at $c$ in both frames:
$$ x^2 + y^2 + z^2 = c^2 t^2, \qquad x'^2 + y'^2 + z'^2 = c^2 t'^2 $$
Homogeneity of space and time makes the transformation linear; motion along $x$ leaves $y, z$ alone. Take the ansatz
$$ x' = \gamma(x - vt), \qquad t' = \gamma\!\left(t - \frac{vx}{c^2}\right) $$
and demand that the interval $c^2t^2 - x^2$ be equal to $c^2t'^2 - x'^2$. Substituting,
$$ c^2t^2 - x^2 = \gamma^2\!\left(c^2t^2 - 2vtx + \frac{v^2x^2}{c^2}\right) - \gamma^2\!\left(x^2 - 2vtx + v^2t^2\right) = \gamma^2\left(c^2t^2 - x^2\right)\!\left(1 - \frac{v^2}{c^2}\right) $$
The cross terms cancel, and matching the two sides fixes
$$ \gamma = \frac{1}{\sqrt{1 - v^2/c^2}} $$
the **Lorentz factor**: $\gamma = 1$ at rest, rising without bound as $v \to c$. For $v \ll c$ the equations collapse back to the Galilean ones.

**Velocities no longer add.** Differentiating the transformation, an object moving at $u'$ along $x$ in $S'$ moves in $S$ at
$$ u = \frac{u' + v}{1 + u'v/c^2} $$
For $u', v \ll c$ this is the Galilean $u' + v$; but set $u' = c$ and it returns $u = c$ exactly, for any $v$ — the second postulate is built into the transformation, and nothing sub-$c$ can be boosted past $c$.

**What is preserved** is the spacetime interval. In four-vector language, with signature $(+,-,-,-)$, every four-vector $A^\mu = (A^0, A^1, A^2, A^3)$ has a Minkowski norm
$$ A^\mu A_\mu = (A^0)^2 - (A^1)^2 - (A^2)^2 - (A^3)^2 $$
that every inertial observer computes to the same value. This invariance *is* the geometry of Minkowski spacetime, and the [curved-spacetime generalisation](/citadel/physics/astrodynamics-advanced) is general relativity.

## Time dilation

Let a clock sit at rest in $S$, ticking off a **proper time** interval $\Delta t_0$ between two events at the same place. Any frame moving at $v$ relative to the clock measures a longer interval:
$$ \Delta t = \gamma\,\Delta t_0 \ge \Delta t_0 $$

**Light-clock derivation.** The clock bounces light between two mirrors a distance $L$ apart, transverse to the motion; one round trip is one tick. In the clock's own frame the light goes straight across and back, $\Delta t_0 = 2L/c$. In a frame where the clock moves at $v$, the light travels a longer diagonal path of half-length $D$, while the clock advances $v\,\Delta t$:
$$ D^2 = L^2 + \left(\tfrac{1}{2}v\,\Delta t\right)^2 $$
Put $L = \tfrac{1}{2}c\,\Delta t_0$ and $D = \tfrac{1}{2}c\,\Delta t$:
$$ (c\,\Delta t)^2 = (c\,\Delta t_0)^2 + (v\,\Delta t)^2 \;\Longrightarrow\; \Delta t^2\left(c^2 - v^2\right) = c^2\,\Delta t_0^2 \;\Longrightarrow\; \Delta t = \gamma\,\Delta t_0 $$
The moving clock is seen to run slow. This is not an illusion of signalling delay — it is what careful measurements in that frame give.

## Length contraction

An object of **proper length** $L$ at rest in $S$ is measured in a frame $S'$ moving at $v$ to be shorter along the motion:
$$ L' = \frac{L}{\gamma} $$
The subtlety is that measuring a moving object's length means marking both ends *at the same time in $S'$*. Transform both endpoints,
$$ x_1' = \gamma(x_1 - vt_1), \qquad x_2' = \gamma(x_2 - vt_2) $$
so $L' = x_2' - x_1' = \gamma\big(L - v\,\Delta t\big)$ where $\Delta t = t_2 - t_1$ in $S$. The simultaneity condition in $S'$ is $\Delta t' = \gamma(\Delta t - vL/c^2) = 0$, giving $\Delta t = vL/c^2$. Substitute:
$$ L' = \gamma L\left(1 - \frac{v^2}{c^2}\right) = \frac{L}{\gamma} $$
Only the dimension along $v$ shrinks; transverse lengths are unchanged.

## Proper time along a worldline

The interval between two nearby events,
$$ ds^2 = c^2\,dt^2 - dx^2 - dy^2 - dz^2 $$
is invariant. For a particle moving at speed $u$, its spatial step in time $dt$ is $u\,dt$, so
$$ ds^2 = c^2\,dt^2 - u^2\,dt^2 = c^2\,dt^2\left(1 - \frac{u^2}{c^2}\right) $$
Define **proper time** $d\tau$ as the time in the particle's own rest frame, where $ds^2 = c^2\,d\tau^2$. Then
$$ d\tau = dt\,\sqrt{1 - \frac{u^2}{c^2}} = \frac{dt}{\gamma} $$
$\tau$ is an invariant scalar — the elapsed time a clock carried along that specific worldline reads. It is the natural parameter for differentiating four-vectors, because every observer agrees on it.

## Four-vectors

Quantities that transform like $(ct, x, y, z)$ under a Lorentz boost are **four-vectors**.

**Four-position:** $x^\mu = (ct,\, x,\, y,\, z)$.

**Four-velocity:** differentiate with respect to the invariant $\tau$, using $dt/d\tau = \gamma$ and the chain rule:
$$ u^\mu = \frac{dx^\mu}{d\tau} = \left(c\,\frac{dt}{d\tau},\; \frac{dx}{d\tau},\; \dots\right) = \gamma\,(c,\; u_x,\; u_y,\; u_z) $$

**Four-momentum:** $p^\mu = m_0\,u^\mu = m_0\gamma\,(c,\, u_x,\, u_y,\, u_z)$, with $m_0$ the rest mass. To identify its time component, compute the work done accelerating the particle from rest. With $p = m_0\gamma u$,
$$ \frac{dp}{du} = m_0\,\frac{d}{du}\frac{u}{\sqrt{1 - u^2/c^2}} = m_0\gamma^3 \quad\Longrightarrow\quad E = \int u\,dp = m_0\int_0^u \gamma^3 u\,du = m_0 c^2(\gamma - 1) $$
That is the kinetic energy. Adding the rest energy $E_0 = m_0 c^2$ gives the total
$$ E = \gamma m_0 c^2, \qquad p^\mu = \left(\frac{E}{c},\; p_x,\; p_y,\; p_z\right) $$

## Energy–momentum relation

The norm of $p^\mu$ is invariant. From $E = \gamma m_0 c^2$ and $p = \gamma m_0 u$,
$$ E^2 - p^2 c^2 = m_0^2 c^4 \gamma^2\left(1 - \frac{u^2}{c^2}\right) = m_0^2 c^4 $$
so
$$ \boxed{E^2 = p^2 c^2 + m_0^2 c^4} $$
For a massless particle this is $E = pc$; for one at rest, $E = m_0 c^2$. Writing $m = \gamma m_0$ (the "relativistic mass") recovers $E = mc^2$ and $p = mv$, though modern usage keeps $m_0$ as *the* mass and carries the $\gamma$ explicitly.

## Relativistic Doppler effect

The phase of a plane wave, $\phi = \omega t - \vec{k}\cdot\vec{x}$, counts wave crests — a number every observer must agree on, so $\phi$ is a Lorentz scalar. That means the **wave four-vector** $k^\mu = (\omega/c,\, k_x,\, k_y,\, k_z)$ transforms like $x^\mu$, and $\phi = k^\mu x_\mu$. Boosting $k^\mu$ along $x$:
$$ \frac{\omega'}{c} = \gamma\left(\frac{\omega}{c} - \frac{v}{c}k_x\right) $$
With $k_x = (\omega/c)\cos\theta$ for light emitted at angle $\theta$ to the boost,
$$ \omega' = \gamma\,\omega\left(1 - \frac{v}{c}\cos\theta\right) $$
Inverting to the usual setup — observer moving at $v$ relative to a source at rest —
$$ f' = \frac{f}{\gamma\left(1 + \dfrac{v}{c}\cos\theta\right)} $$
Along the line of motion this is the ordinary red/blue shift, but sharpened by the $\gamma$. At $\theta = 90°$ it leaves a purely relativistic **transverse Doppler shift** $f' = f/\gamma$ — a redshift with no line-of-sight motion at all, a direct measurement of time dilation. (The non-relativistic [sound Doppler effect](/citadel/physics/waves) has no such transverse term.)

## E = mc² from a recoil argument

A body of rest mass $m_0$, at rest in $S$, emits two equal light pulses along $+x$ and $-x$, total energy $L$, one carrying $L/2$ each. Their momenta $\pm L/2c$ cancel, so the body stays put.

View the same event from $S'$ moving at $v$ along $+x$. The relativistic Doppler factor shifts the two pulses oppositely; since a photon's energy scales with its frequency,
$$ E_+' = \frac{L}{2\gamma(1 + v/c)}, \qquad E_-' = \frac{L}{2\gamma(1 - v/c)} $$
Using $p = E/c$ for light, the net momentum the radiation carries off in $S'$ is
$$ \Delta p' = \frac{L}{2c\gamma}\left(\frac{1}{1 - v/c} - \frac{1}{1 + v/c}\right) = \gamma\,\frac{Lv}{c^2} $$
In $S'$ the body was moving at $v$ throughout. Before emission its momentum is $\gamma m_0 v$; after, $\gamma(m_0 - \Delta m)v$, a change of $\gamma\,\Delta m\,v$. Momentum conservation demands this balance the radiation's:
$$ \gamma\,\frac{Lv}{c^2} = \gamma\,\Delta m\,v \quad\Longrightarrow\quad L = \Delta m\,c^2 $$
Losing energy $L$ costs the body mass $L/c^2$. Rest mass is a store of energy, at the exchange rate $E = mc^2$ — the principle behind fission, fusion, and the Sun.

## Reading a Minkowski diagram

Plot $ct$ up and one space axis $x$ across; the invariant $ds^2 = (c\,dt)^2 - (dx)^2$ replaces Euclidean distance.

![A light cone in a spacetime diagram: the future cone opening upward, the past cone downward, the present at the apex, and the causally disconnected "elsewhere" region outside both.](../images/light-cone.png "The light cone through an event. Inside: absolute future and past (timelike). On the surface: light paths (null). Outside: elsewhere (spacelike), causally disconnected. Source: Wikimedia Commons.")

- **Worldlines** are an object's path through spacetime. At rest: a vertical line. Constant velocity: a straight line tilted toward the light cone, but never past $45°$.
- **The light cone** through an event is the set of all light paths through it. Its interior splits into the event's absolute **future** and absolute **past**; everything outside is **elsewhere**, causally cut off. Timelike separations ($ds^2 > 0$) lie inside and can be connected by a sub-$c$ signal; null separations ($ds^2 = 0$) lie on the surface — light; spacelike separations ($ds^2 < 0$) lie outside, and no signal joins them.
- **Relativity of simultaneity.** A frame's line of "same time" is parallel to its $x$-axis. Boost to another frame and that line tilts. Two events level in one frame are not level in another — there is no universal "now", only each observer's slice through spacetime.

![Animation of a Lorentz boost on a spacetime diagram: a set of events on a worldline, with the time and space axes shearing toward the 45° light cone as the boost velocity changes, while the light cone itself stays fixed.](../images/lorentz-worldline.gif "A Lorentz boost shears the time and space axes symmetrically toward the light cone (fixed), never past 45°. Source: Wikimedia Commons.")

## The one idea to keep

Keep the constancy of $c$ and give up absolute time and length instead. Everything follows from demanding that the interval $c^2t^2 - x^2$ be the same for every inertial observer: the Lorentz transformation, the factor $\gamma = 1/\sqrt{1 - v^2/c^2}$, velocities that add so nothing crosses $c$, and the four-vector formalism in which energy and momentum are one object whose invariant length is $m_0 c^2$ — hence $E^2 = p^2c^2 + m_0^2c^4$ and $E = mc^2$. Time dilation, length contraction, and the relativity of simultaneity are not three separate effects but one geometric fact seen three ways: different observers slice the same four-dimensional spacetime along differently tilted time and space axes, disagreeing about durations, lengths, and "now" while agreeing on every interval. Curve that spacetime with mass-energy and you have [general relativity](/citadel/physics/astrodynamics-advanced).
