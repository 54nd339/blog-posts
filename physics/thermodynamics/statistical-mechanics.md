---
title: Statistical Mechanics - From Microstates to Thermodynamics
description: Thermodynamics states laws about heat and entropy without saying what they are. Statistical mechanics derives them by counting microscopic arrangements. Entropy becomes the logarithm of that count, temperature a Lagrange multiplier, and the partition function a single object from which every thermodynamic quantity follows.
date: 2020-08-22
draft: false
slug: /physics/statistical-mechanics
tags:
  - Physics
  - Thermodynamics
---

[Thermodynamics](/citadel/physics/thermodynamics) is a set of laws about macroscopic quantities — temperature, pressure, entropy, free energy — that were extracted from experiment without any picture of what matter is made of. It tells you entropy never decreases; it doesn't tell you what entropy *is*. The [kinetic theory of gases](/citadel/physics/kinetics) took the first step toward a microscopic account, deriving $PV = NkT$ from bouncing molecules and identifying temperature with mean kinetic energy.

Statistical mechanics completes the job. Given that a macroscopic system is $10^{23}$ particles obeying known microscopic laws, it derives the entire structure of thermodynamics by *counting* — asking how many microscopic arrangements correspond to each macroscopic state, and assuming the system is equally likely to be in any of them. Entropy becomes a logarithm of a count; temperature falls out as a mathematical multiplier; and one function, the partition function, generates everything else.

## Microstates and macrostates

A **microstate** is a complete microscopic specification: every particle's position and momentum (classically), or the full quantum state. A **macrostate** is what you can actually measure: total energy, volume, particle number, and the handful of variables derived from them.

Vastly many microstates map to the same macrostate. Toss $N$ coins: the macrostate "half heads" corresponds to $\binom{N}{N/2}$ microstates, while "all heads" corresponds to exactly one. For $N = 100$ the ratio is about $10^{29}$. A gas is the same story with continuous variables: "the energy is $E$, the volume is $V$" is compatible with an enormous region of the $6N$-dimensional phase space.

**The fundamental postulate:** an isolated system in equilibrium is equally likely to be in any of its accessible microstates. Everything follows from this one assumption of ignorance — you don't know which microstate, so you weight them all equally.

## Boltzmann entropy

Let $\Omega(E, V, N)$ be the number of microstates consistent with a given macrostate (the phase-space volume, suitably made dimensionless). Boltzmann's definition of entropy is

$$S = k_B \ln \Omega,$$

with $k_B = 1.38 \times 10^{-23}\ \text{J/K}$ the Boltzmann constant. The logarithm is not a convention of taste: it's forced by requiring entropy to be **additive**. Two independent subsystems have $\Omega_{\text{total}} = \Omega_1 \Omega_2$ (each microstate of one pairs with each of the other), and $\ln(\Omega_1\Omega_2) = \ln\Omega_1 + \ln\Omega_2$, so $S$ adds.

Now the second law is a statement about counting. Two gases at different temperatures brought into contact will, overwhelmingly probably, evolve toward the macrostate with the most microstates — because that macrostate occupies essentially all of the accessible phase space. "Entropy increases" means "the system moves to the macrostate you're most likely to find it in," and for $10^{23}$ particles "most likely" is a certainty to more decimal places than anyone can write.

## The three ensembles

Rather than tracking one isolated system, it's easier to imagine a large collection (**ensemble**) of identically-prepared systems and average over it. Three standard setups, differing in what's held fixed:

- **Microcanonical** — isolated system, $(E, V, N)$ fixed. All microstates equally likely; $S = k_B \ln \Omega$ directly. Clean in principle, awkward in practice because counting states at *exactly* energy $E$ is hard.
- **Canonical** — system in contact with a heat bath at temperature $T$; $(T, V, N)$ fixed, energy fluctuates. This is the workhorse.
- **Grand canonical** — system exchanges both energy and particles with a reservoir; $(T, V, \mu)$ fixed, where $\mu$ is the **chemical potential**. Needed for quantum gases and any problem where particle number isn't conserved.

In the thermodynamic limit ($N \to \infty$) the three give identical predictions for averages, because fluctuations scale as $1/\sqrt N$.

## The Boltzmann distribution

For a canonical system, what is the probability $p_i$ of finding it in a particular microstate $i$ with energy $E_i$?

Maximise the entropy $S = -k_B \sum_i p_i \ln p_i$ (the general **Gibbs** form) subject to two constraints: probabilities sum to $1$, and the average energy $\sum_i p_i E_i$ is fixed. Using **Lagrange multipliers** $\alpha$ and $\beta$ for the two constraints and setting the derivative to zero gives

$$p_i = \frac{e^{-\beta E_i}}{Z}, \qquad Z = \sum_i e^{-\beta E_i}.$$

Two things to notice. First, the multiplier $\beta$ that enforces "average energy is fixed" turns out (by matching to thermodynamics) to be $\beta = 1/k_B T$ — **temperature is the Lagrange multiplier for energy**. Second, higher-energy states are exponentially suppressed by the **Boltzmann factor** $e^{-E_i/k_B T}$: at temperature $T$, a state costing energy $\Delta E$ above the ground state is less populated by a factor $e^{-\Delta E/k_B T}$. This one factor governs reaction rates (Arrhenius), atmospheric density with altitude, semiconductor carrier concentrations, and the population inversion needed for a [laser](/citadel/physics/laser).

![Maxwell-Boltzmann speed distributions at three temperatures, the peak shifting to higher speed and the tail flattening as temperature rises.](../images/maxwell-boltzmann2.png "The Boltzmann factor applied to molecular speeds: raising T shifts weight into the high-energy tail. Source: Wikimedia Commons.")

## The partition function

$Z = \sum_i e^{-E_i/k_B T}$ (or an integral over phase space) looks like a mere normalisation constant. It is in fact the generating function for all of thermodynamics. Once you can compute $Z$, you differentiate:

$$\langle E \rangle = -\frac{\partial \ln Z}{\partial \beta}, \qquad F = -k_B T \ln Z, \qquad S = -\frac{\partial F}{\partial T}, \qquad P = -\frac{\partial F}{\partial V}, \qquad C_V = \frac{\partial \langle E\rangle}{\partial T}.$$

$F = E - TS$ is the **Helmholtz free energy**, the quantity a system at fixed $T$ and $V$ minimises at equilibrium. So the recipe for *any* system is: enumerate its energy levels, sum $e^{-\beta E_i}$ to get $Z$, take logs and derivatives. All the thermodynamic laws — including that entropy is $-\partial F/\partial T$ and that it increases — are now consequences, not postulates.

For independent subsystems (non-interacting particles), $Z$ **factorises**: $Z = z^N$ (or $z^N/N!$ for indistinguishable classical particles, the $N!$ resolving the **Gibbs paradox** — without it, mixing two samples of the same gas would spuriously increase entropy). The one-particle $z$ for an ideal gas gives, after the derivatives, exactly $PV = Nk_BT$, the energy $\tfrac32 Nk_BT$, and the **Sackur–Tetrode** formula for the absolute entropy of a monatomic gas.

## Equipartition and its failure

If the energy is a sum of quadratic terms (each momentum component, each spring coordinate), the canonical average of each such term is $\tfrac12 k_B T$ — the **equipartition theorem**. A monatomic gas has three translational quadratic terms, so $\langle E\rangle = \tfrac32 Nk_BT$ and $C_V = \tfrac32 Nk_B$. A diatomic gas adds two rotational terms, giving $\tfrac52$.

Equipartition is a *classical* result, and its failures were among the first clues to quantum mechanics:

- **Heat capacity of solids.** Classically $C_V = 3Nk_B$ (Dulong–Petit) at all temperatures. Measured, it drops toward zero as $T \to 0$. Einstein and then Debye fixed it by quantising the lattice vibrations — modes with $\hbar\omega \gg k_B T$ are "frozen out," contributing nothing.
- **Diatomic gases.** The vibrational mode doesn't contribute until the temperature is high enough that $k_B T$ reaches the vibrational quantum, so $C_V$ rises in steps as rotational then vibrational modes switch on.
- **Blackbody radiation.** Classical equipartition over all electromagnetic modes gives infinite energy (the ultraviolet catastrophe). Planck's quantisation $E = n\hbar\omega$ per mode, fed through the same partition-function machinery, gives the correct spectrum — historically the birth of the quantum.

## Quantum statistics

For identical quantum particles, microstates can't be counted by pretending the particles are labelled — swapping two makes no new state. The counting splits by particle type:

- **Bosons** (integer spin — photons, phonons, helium-4) can pile arbitrarily many particles into one state. The grand-canonical count gives the **Bose–Einstein distribution** $\langle n_k\rangle = \dfrac{1}{e^{(\epsilon_k - \mu)/k_BT} - 1}$.
- **Fermions** (half-integer spin — electrons, protons) obey the Pauli exclusion principle: at most one per state. The count gives the **Fermi–Dirac distribution** $\langle n_k\rangle = \dfrac{1}{e^{(\epsilon_k - \mu)/k_BT} + 1}$ — a smoothed step, essentially $1$ below the chemical potential and $0$ above it.

Both reduce to the classical **Maxwell–Boltzmann** distribution $e^{-(\epsilon_k - \mu)/k_BT}$ in the dilute, high-temperature limit where states are rarely multiply occupied and the $\pm 1$ is negligible.

The consequences are large. Fermi–Dirac statistics give metals their electronic heat capacity and pressure (a degenerate electron gas exerts pressure even at $T = 0$ — the "Fermi pressure" that holds up [white dwarfs](/citadel/physics/stellar-astrophysics)), and they're the reason [semiconductors](/citadel/physics/solid-state) work. Bose–Einstein statistics give the photon gas (Planck's law) and, below a critical temperature, **Bose–Einstein condensation** — a macroscopic fraction of the particles collapsing into the single lowest state, observed directly in dilute atomic gases in 1995. Paramagnetism, the two-state system in a magnetic field, is the simplest exactly-solvable example: $Z = 2\cosh(\mu B/k_BT)$, and every magnetic property follows by differentiation.

## The one idea to keep

Statistical mechanics derives thermodynamics by counting microstates and assuming each is equally likely. Entropy is $k_B \ln \Omega$ — the log forced by additivity — so the second law just says the system drifts to the macrostate that occupies almost all the phase space. For a system at temperature $T$, the probability of a microstate is $e^{-E_i/k_BT}/Z$, with temperature appearing as the Lagrange multiplier that fixes the average energy, and the partition function $Z = \sum e^{-E_i/k_BT}$ is the single object from which energy, entropy, pressure, and heat capacity all come by differentiation. Where classical equipartition breaks — solids at low $T$, blackbody radiation — is exactly where quantum counting (Bose–Einstein, Fermi–Dirac) takes over.
