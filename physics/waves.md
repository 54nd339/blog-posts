---
title: Waves - Nature's Rhythmic Dance of Energy and Information
description: Whether it’s sound, light, or ripples in your coffee, waves are everywhere. This post dives into interference, diffraction, and the Doppler effect—because who doesn’t love a frequency shift? Click in for a wave-riding adventure through the physics of motion and melody!
date: 2023-01-28
draft: false
slug: /pensieve/physics/waves
tags:
  - Physics
  - Classical Mechanics
---

Hello, vibration voyagers and frequency followers! Have you ever considered the sheer ubiquity of waves? From the gentle lapping of water against a shore, the sound of music filling a room, the light that allows us to perceive the world, to the invisible radio waves connecting our devices – waves are fundamental to how energy and information propagate through the universe.

This exploration delves into the fascinating physics of waves, particularly **harmonic waves**, which form the basis for understanding many complex wave phenomena. We will uncover the mathematical description of waves, distinguish between their different types, and examine what happens when waves meet, bounce, or bend. Prepare for a journey through superposition, interference, diffraction, and the intriguing shifts in perception caused by the Doppler effect.

---
## The Song of the Wave: Harmonic Waves and the Wave Equation

A **harmonic wave** is a periodic disturbance that propagates through a medium or space, where the particles of the medium (if any) oscillate in simple harmonic motion around their equilibrium positions. These waves are characterized by several key properties:

* **Amplitude (A):** The maximum displacement of a particle from its equilibrium position.
* **Wavelength ($\lambda$):** The spatial period of the wave; the distance between two consecutive points that are in the same phase (e.g., crest to crest).
* **Frequency (f):** The number of complete wave cycles passing a given point per unit of time.
* **Period (T):** The time taken for one complete wave cycle to pass a point ($T = 1/f$).
* **Wave Velocity (v):** The speed at which the wave disturbance propagates through the medium ($v = \lambda f$).

The mathematical description of a one-dimensional progressive harmonic wave traveling in the positive x-direction is often given by:
$$ y(x, t) = A \sin(kx - \omega t + \phi) $$
Where:
* $y(x,t)$ is the displacement at position $x$ and time $t$.
* $k$ is the **wave number**, $k = 2\pi/\lambda$.
* $\omega$ is the **angular frequency**, $\omega = 2\pi f = 2\pi/T$.
* $\phi$ is the **phase constant**, determining the initial phase of the wave at $x=0, t=0$.

This sinusoidal function is a solution to the **classical wave equation**, a fundamental second-order linear partial differential equation. We can derive it:
Taking partial derivatives of $y(x,t)$ with respect to $x$:
$$ \frac{\partial y}{\partial x} = kA \cos(kx - \omega t + \phi) $$
$$ \frac{\partial^2 y}{\partial x^2} = -k^2 A \sin(kx - \omega t + \phi) = -k^2 y(x,t) $$
Taking partial derivatives of $y(x,t)$ with respect to $t$:
$$ \frac{\partial y}{\partial t} = -\omega A \cos(kx - \omega t + \phi) $$
$$ \frac{\partial^2 y}{\partial t^2} = -\omega^2 A \sin(kx - \omega t + \phi) = -\omega^2 y(x,t) $$
From these, we see $\frac{\partial^2 y}{\partial x^2} / \frac{\partial^2 y}{\partial t^2} = k^2/\omega^2$.
Since the wave velocity $v = \omega/k$, then $1/v^2 = k^2/\omega^2$.
Thus, we arrive at the 1D wave equation:
$$ \frac{\partial^2 y}{\partial x^2} = \frac{1}{v^2} \frac{\partial^2 y}{\partial t^2} $$
This equation governs the propagation of many types of waves.

---
## Unveiling the Two Faces of Waves: Transverse and Longitudinal

Waves can be broadly categorized based on the direction of particle oscillation relative to the direction of wave propagation.

### Transverse Waves: Wiggling Perpendicular to Travel
In **transverse waves**, the particles of the medium oscillate perpendicular (transverse) to the direction in which the wave is traveling.
Examples: Waves on a string, electromagnetic waves (like light), ripples on a water surface.

Consider a wave traveling along a stretched string with tension $T$ and linear mass density $\mu$ (mass per unit length). The speed ($v$) of a transverse wave on this string is:
$$ v = \sqrt{\frac{T}{\mu}} $$
*Derivation Sketch:* Consider a small segment of the string $dm = \mu dx$. If this segment is part of a circular arc (due to the wave pulse), the net vertical component of tension provides the centripetal force $dm \cdot v^2/R_{curvature}$. A geometric argument relating the curvature to the second spatial derivative, or considering forces on a small element with slopes at its ends, leads to the wave equation with $v^2 = T/\mu$. Alternatively, for a pulse, one can analyze forces in a frame moving with the pulse. If a small segment $dx$ of the string is part of a curve of radius $r_{curve}$ (different from $r$ in $mv^2/r$), and the angles made by tension vectors with horizontal are small, the net vertical force $T\Delta\theta \approx T (dx/r_{curve})$. The mass is $\mu dx$. The centripetal acceleration is $v^2/r_{curve}$. Thus $T(dx/r_{curve}) = (\mu dx) v^2/r_{curve} \implies T = \mu v^2$.

**Energy, Power, and Intensity in Transverse Waves (on a string):**
Each segment of the string in SHM has energy.
The maximum kinetic energy of a small segment $dm$ is $KE_{max} = \frac{1}{2}dm (v_p)_{max}^2 = \frac{1}{2}(\mu dx)(A\omega)^2$.
The average kinetic energy over one period is $KE_{avg} = \frac{1}{4}(\mu dx)A^2\omega^2$.
The average potential energy is also $PE_{avg} = \frac{1}{4}(\mu dx)A^2\omega^2$.
The total average energy per unit length is $dE/dx = \frac{1}{2}\mu A^2\omega^2$.
**Power ($P$)** transmitted by the wave is energy per unit time: $P = (dE/dx)(dx/dt) = (dE/dx)v$.
$$ P = \frac{1}{2}\mu \omega^2 A^2 v $$
**Intensity ($I$)** for waves spreading in 3D is power per unit area ($P/Area_{wavefront}$). For a 1D wave like on a string, intensity is less commonly used, but power is well-defined. If considering sound waves or light waves in 3D, where $\rho$ is the volume density of the medium:
$$ I = \frac{1}{2}\rho v \omega^2 A^2 $$

### Longitudinal Waves: Pushing and Pulling Along the Path
In **longitudinal waves** (or compressional waves), the particles of the medium oscillate parallel to the direction of wave propagation. This creates alternating regions of compression (higher density/pressure) and rarefaction (lower density/pressure).
Examples: Sound waves in air, seismic P-waves.

The speed ($v$) of a longitudinal wave (like sound) in a fluid (gas or liquid) depends on the fluid's elastic property (Bulk Modulus, $B$) and its inertial property (density, $\rho$):
$$ v = \sqrt{\frac{B}{\rho}} $$
For an ideal gas undergoing adiabatic compressions and rarefactions (which is typical for sound waves): $B_{adiabatic} = \gamma P$, where $P$ is the equilibrium pressure and $\gamma$ is the adiabatic index ($C_p/C_v$).
So, the speed of sound in an ideal gas is:
$$ v = \sqrt{\frac{\gamma P}{\rho}} $$
Using the ideal gas law ($P/\rho = RT/M_w$, where $M_w$ is molar mass, $R$ is gas constant, $T$ is absolute temperature):
$$ v = \sqrt{\frac{\gamma RT}{M_w}} $$
*Derivation Sketch for $v=\sqrt{B/\rho}$*:
Consider a fluid element of area $S$ and thickness $dx$. If a pressure difference $dP$ acts across it, the net force is $S dP$. The mass is $\rho S dx$. Acceleration is $d^2u/dt^2$, where $u$ is displacement.
$S dP = (\rho S dx) \frac{\partial^2 u}{\partial t^2} \implies \frac{\partial P}{\partial x} = \rho \frac{\partial^2 u}{\partial t^2}$.
The bulk modulus relates pressure change to volumetric strain: $dP = -B (dV/V_0)$.
For a segment $dx$, $dV/V_0 = du/dx$ (strain). So $dP = -B (du/dx)$.
Then $\frac{\partial P}{\partial x} = -B \frac{\partial^2 u}{\partial x^2}$.
Substituting: $-B \frac{\partial^2 u}{\partial x^2} = \rho \frac{\partial^2 u}{\partial t^2} \implies \frac{\partial^2 u}{\partial x^2} = \frac{\rho}{B} \frac{\partial^2 u}{\partial t^2}$.
Comparing with the wave equation $\frac{\partial^2 y}{\partial x^2} = \frac{1}{v^2} \frac{\partial^2 y}{\partial t^2}$, we get $v^2 = B/\rho$.

For a longitudinal wave (e.g., sound), the displacement is $s(x,t) = s_m \cos(kx-\omega t)$.
The pressure variation $\Delta P(x,t) = -B \frac{\partial s}{\partial x} = -B (-ks_m \sin(kx-\omega t)) = Bk s_m \sin(kx-\omega t)$.
The pressure amplitude is $P_0 = Bks_m = B(\omega/v)s_m = \rho v \omega s_m$.
Intensity of a longitudinal wave:
$$ I = \frac{1}{2} \rho v \omega^2 s_m^2 = \frac{P_0^2}{2\rho v} = \frac{P_0^2 v}{2B} $$
(where $s_m$ is displacement amplitude).

---
## The Art of Overlap: The Principle of Superposition

What happens when two waves try to occupy the same space at the same time? They don't crash and burn; they **superpose**!
The **Principle of Superposition** states that when two or more waves overlap, the resultant displacement at any point and at any instant is the vector (or algebraic, for 1D) sum of the displacements that each individual wave would produce alone at that point and instant.
$$ y_{total}(x, t) = y_1(x, t) + y_2(x, t) + \ldots = \sum_{i=1}^{n} y_i(x, t) $$
This simple rule is incredibly powerful and leads to phenomena like interference and beats.

---
## The Bouncing Back Act: Reflection of Waves

When a wave traveling in one medium encounters a boundary with another medium, some or all of the wave energy can be **reflected** back into the first medium.
The basic **law of reflection** (for light, and often for other waves from flat surfaces) states:
1.  The angle of incidence equals the angle of reflection.
2.  The incident wave, the reflected wave, and the normal to the surface at the point of incidence all lie in the same plane.

**Phase Changes on Reflection:**
* **Transverse Waves on a String:**
    * If a wave reflects from a **fixed end** (denser boundary), the reflected wave is inverted (undergoes a $180^\circ$ or $\pi$ radian phase change).
    * If it reflects from a **free end** (rarer boundary), the reflected wave is not inverted (no phase change).
* **Longitudinal Waves (e.g., Sound):**
    * Reflection from a **rigid wall** (denser medium): Displacement reflects with a phase change (like a fixed end for a string if displacement is the variable), but pressure reflects without a phase change (pressure antinode).
    * Reflection from an **open end** of a pipe (rarer medium): Displacement reflects without a phase change (pressure node).

---
## The Wave Dance-Off: Interference – Constructive High-Fives & Destructive Face-Palms

**Interference** is a direct consequence of the superposition principle. When two (or more) coherent waves (waves with a constant phase difference, usually from the same source or phase-locked sources) meet, their displacements add up.

Consider two waves $y_1 = A_1 \sin(kx - \omega t)$ and $y_2 = A_2 \sin(kx - \omega t + \phi_{initial})$, where $\phi_{initial}$ is their initial phase difference. More generally, if they travel different path lengths $\Delta x$ from a common source to a point, there's a path difference leading to a phase difference $\Delta\phi = k\Delta x = (2\pi/\lambda)\Delta x$.
The resultant amplitude $R$ (from vector addition of phasors or trigonometric combination, similar to coupled oscillations in the same direction) depends on the amplitudes $A_1, A_2$ and the total phase difference $\phi = (kx_2 - \omega t + \phi_{2,0}) - (kx_1 - \omega t + \phi_{1,0}) = k(x_2-x_1) + (\phi_{2,0}-\phi_{1,0}) = k\Delta x + \Delta\phi_0$.
The resultant intensity $I \propto R^2$.
* **Constructive Interference:** Occurs when waves meet in phase (e.g., crest meets crest). The resultant amplitude is maximum ($A_1+A_2$ if aligned). This happens when the phase difference $\phi$ is an even multiple of $\pi$ (i.e., $0, 2\pi, 4\pi, \ldots$).
    Path difference condition: $\Delta x = m\lambda \quad (m = 0, 1, 2, \ldots)$
* **Destructive Interference:** Occurs when waves meet out of phase (e.g., crest meets trough). The resultant amplitude is minimum ($|A_1-A_2|$ if aligned). This happens when the phase difference $\phi$ is an odd multiple of $\pi$ (i.e., $\pi, 3\pi, 5\pi, \ldots$).
    Path difference condition: $\Delta x = \left(m + \frac{1}{2}\right)\lambda \quad (m = 0, 1, 2, \ldots)$

### Interference of Light Waves: The Colorful Evidence
Light, being a wave, exhibits interference.
* **Huygens' Principle:** A useful model stating that every point on a wavefront can be considered as a source of secondary spherical wavelets. The new wavefront is the envelope of these wavelets.
* **Conditions for Observable/Sustained Light Interference:**
    1.  **Coherent Sources:** The sources must maintain a constant phase difference. (Often achieved by splitting light from a single source).
    2.  **Same Frequency/Wavelength:** The light must be (nearly) monochromatic.
    3.  **Same or Parallel Polarization:** For vector waves like light.
    4.  **Sufficiently Close Sources & Large Screen Distance:** To see distinct fringes.
    5.  **Comparable Amplitudes:** For good contrast between bright and dark fringes.
* **Types of Interference Setups (Examples):**
    1.  **Division of Wavefront:** The original wavefront is split into parts that travel different paths (e.g., Young's double-slit experiment, Fresnel biprism, Lloyd's mirror).
    2.  **Division of Amplitude:** The amplitude of an incident wave is split into two or more parts by partial reflection and refraction (e.g., interference in thin films like soap bubbles, Newton's rings, Michelson interferometer).

---
## The Rhythmic Pulse: Beats – When Frequencies Almost Match

When two sound waves of slightly different frequencies $f_1$ and $f_2$ superpose, the resultant sound intensity periodically increases and decreases, creating a phenomenon called **beats**.
The perceived loudness pulsates at the **beat frequency ($f_{beat}$)**:
$$ f_{beat} = |f_1 - f_2| $$
This is commonly used by musicians to tune instruments.

---
## Bending Around Corners: Diffraction – Light's Sneaky Detour

**Diffraction** is the tendency of waves to spread out or bend as they pass through narrow openings or around obstacles. It's a clear demonstration of the wave nature of light (and other waves). The amount of bending is significant when the size of the obstacle or opening is comparable to or smaller than the wavelength of the wave.

* **Fresnel vs. Fraunhofer Diffraction:**
    * **Fresnel Diffraction:** Occurs when either the source of light or the observation screen (or both) are at a finite distance from the diffracting obstacle/aperture. The wavefronts are generally spherical or cylindrical. Patterns are complex.
    * **Fraunhofer Diffraction:** A simpler case that occurs when both the source and the screen are effectively at infinite distance from the obstacle (achieved using lenses to make incident light parallel and focus diffracted light). Wavefronts are planar.

* **Diffraction by a Single Slit (Fraunhofer):**
    When parallel light passes through a narrow slit of width $a$, it produces a diffraction pattern on a distant screen, consisting of a wide, bright central maximum, flanked by dimmer and narrower secondary maxima, separated by minima (dark fringes).
    The angular positions of the **minima** are given by:
    $$ a \sin\theta = m\lambda \quad (m = \pm 1, \pm 2, \pm 3, \ldots) $$
    (Note: $m=0$ corresponds to the central maximum, not a minimum).
    The intensity distribution is given by:
    $$ I(\theta) = I_0 \left(\frac{\sin\alpha}{\alpha}\right)^2 \quad \text{where } \alpha = \frac{\pi a \sin\theta}{\lambda} $$
    $I_0$ is the intensity at the center of the pattern ($\theta=0$).

---
## The Directional Dance: Polarization – Light's Hidden Orientation

For transverse waves like light (which are electromagnetic waves), the direction of the oscillation (e.g., the electric field vector $\vec{E}$) is perpendicular to the direction of wave propagation. **Polarization** refers to the orientation of this oscillation in the plane perpendicular to propagation.
Unpolarized light (like from the sun or a light bulb) has its E-field vectors oscillating in all random directions in that plane.

A general polarized wave can be described by components:
$$ \vec{E}(z,t) = E_{0x} \cos(kz - \omega t + \phi_x) \hat{i} + E_{0y} \cos(kz - \omega t + \phi_y) \hat{j} $$
Let $\delta = \phi_y - \phi_x$ be the relative phase.
* **Linear Polarization:** If $\delta = 0$ or $\pi$ (or if one component is zero), the E-field oscillates along a single line. This can be achieved by passing unpolarized light through a **polarizer** (like a Polaroid filter).
* **Circular Polarization:** If $E_{0x} = E_{0y} = E_0$ and $\delta = \pm \pi/2$. The tip of the E-field vector traces a circle as the wave propagates. (Right-circular or left-circular depending on the sign of $\delta$).
* **Elliptical Polarization:** The most general case (unequal amplitudes and/or other phase differences). The E-field vector traces an ellipse.

Polarization is used in sunglasses, LCD screens, 3D movie glasses, and many optical instruments.

---
## Trapped Tunes: Standing Waves & Resonance – The Perfect Harmony

When a wave is confined within a region (like on a guitar string or in an organ pipe), it can reflect back and forth. The incident and reflected waves can interfere to create a **standing wave** (or stationary wave).
In a standing wave, some points called **nodes** remain permanently at rest (zero amplitude), while other points called **antinodes** oscillate with maximum amplitude. The wave pattern itself does not travel along the medium.

* **Standing Waves in Strings (Fixed at Both Ends):**
    The ends must be nodes. This restricts the possible wavelengths. For a string of length $L$:
    Possible wavelengths: $L = n(\lambda_n/2) \implies \lambda_n = \frac{2L}{n}$.
    Possible frequencies (harmonics or overtones): $f_n = v/\lambda_n = \frac{nv}{2L} = n f_1 \quad (n=1, 2, 3, \ldots)$.
    $n=1$ is the fundamental frequency (first harmonic). $n=2, 3, \ldots$ are higher harmonics.

* **Standing Waves in Organ Pipes:**
    * **Open Pipe (Open at Both Ends):** Requires antinodes at both ends (where air can freely move).
        Possible wavelengths: $L = n(\lambda_n/2) \implies \lambda_n = \frac{2L}{n}$.
        Possible frequencies: $f_n = \frac{nv}{2L} = n f_1 \quad (n=1, 2, 3, \ldots)$. (All harmonics are present).
    * **Closed Pipe (Closed at One End, Open at the Other):** Requires a node at the closed end and an antinode at the open end.
        Possible wavelengths: $L = (2n-1)(\lambda_n/4) \implies \lambda_n = \frac{4L}{2n-1}$.
        Possible frequencies: $f_n = \frac{(2n-1)v}{4L} = (2n-1) f_1 \quad (n=1, 2, 3, \ldots)$.
        (Only odd harmonics are present: $f_1, 3f_1, 5f_1, \ldots$).

**Resonance** occurs when a system capable of oscillating (like a string or air column) is driven by an external frequency that matches one of its natural frequencies of oscillation (its harmonic frequencies). At resonance, the system vibrates with a very large amplitude.

---
## The Shifting Pitch: Doppler Effect – Sounds (and Light) on the Move

The **Doppler effect** is the change in observed frequency (and wavelength) of a wave in relation to an observer who is moving relative to the wave source. It's why a siren sounds higher pitched as it approaches you and lower pitched as it moves away.

For **sound waves**, the general formula for the observed frequency $f'$ when the source emits frequency $f$, and $v$ is the speed of sound, $v_o$ is the observer's speed, and $v_s$ is the source's speed:
$$ f' = f \left(\frac{v \pm v_o}{v \mp v_s}\right) $$
Sign conventions:
* Numerator ($v \pm v_o$): Use `+` if observer moves towards source, `-` if observer moves away from source.
* Denominator ($v \mp v_s$): Use `-` if source moves towards observer, `+` if source moves away from observer.

The Doppler effect also applies to light waves (though its formula is different due to relativity, as seen in the Special Relativity blog), and is crucial in astronomy for determining the motion of stars and galaxies (redshift/blueshift).

---
## Key Takeaways: Your Wave Wisdom Cheat Sheet

Waves are nature's way of transmitting energy and keeping things interesting! Here are the key ripples to remember:

* **Wave Basics:** Harmonic waves are described by amplitude, wavelength, frequency, period, and velocity ($v=f\lambda$). The 1D wave equation $\partial^2y/\partial x^2 = (1/v^2)\partial^2y/\partial t^2$ governs their motion.
* **Types of Waves:** **Transverse** (oscillation $\perp$ to propagation, e.g., light, string waves $v=\sqrt{T/\mu}$) and **Longitudinal** (oscillation $\parallel$ to propagation, e.g., sound $v=\sqrt{B/\rho}$).
* **Superposition & Interference:** Waves add up! This leads to **constructive interference** (amplitudes add, path diff. $m\lambda$) and **destructive interference** (amplitudes cancel, path diff. $(m+1/2)\lambda$). Key for thin films, Newton's rings.
* **Beats:** Result from superposition of waves with slightly different frequencies ($f_{beat}=|f_1-f_2|$).
* **Diffraction:** Waves bend around obstacles or through narrow openings (significant when opening size $\sim \lambda$). Single slits produce a central max with side minima ($a\sin\theta = m\lambda$).
* **Polarization:** For transverse waves like light, describes the orientation of oscillation. Can be linear, circular, or elliptical.
* **Standing Waves:** Formed by interference of a wave with its reflection in a confined medium, creating stationary nodes and antinodes. Frequencies are quantized (harmonics) for strings and pipes.
* **Doppler Effect:** The observed frequency of a wave changes if there's relative motion between the source and observer ($f' = f ((v\pm v_o)/(v\mp v_s))$ for sound).

From the grandest cosmic signals to the tiniest quantum jitters, the language of waves is fundamental to understanding our universe. Keep making waves with your curiosity!