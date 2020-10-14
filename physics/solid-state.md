---
title: Solid-State Physics - Crystals, Bands, and Semiconductors
description: Why one material conducts, another insulates, and a third does something adjustable in between. Periodic atoms give electrons allowed energy bands separated by forbidden gaps; whether the highest occupied band is full or partly full decides everything; and doping a semiconductor is how you engineer the answer.
date: 2020-10-14
draft: false
slug: /physics/solid-state
tags:
  - Physics
  - Condensed Matter
---

Copper conducts electricity, diamond doesn't, and silicon can be made to do either depending on what you add to it. All three are just atoms bonded into a solid; the difference in behaviour spans more than twenty orders of magnitude in conductivity. Explaining that range — and, from it, designing transistors — is what solid-state physics does, and the central idea is that packing atoms into a regular array does something specific and calculable to the electrons' allowed energies.

This post builds that up: the crystal lattice and how we probe it, why lattice vibrations are quantised into phonons, the free-electron picture and where it succeeds, and then the band theory that turns the periodic arrangement into allowed bands and forbidden gaps — which is the whole explanation for metals, insulators, and semiconductors.

## The crystal lattice

A crystal is a **basis** (one or a few atoms) repeated at every point of a **lattice** — a set of points $\mathbf{R} = n_1\mathbf{a}_1 + n_2\mathbf{a}_2 + n_3\mathbf{a}_3$ for integers $n_i$ and primitive vectors $\mathbf{a}_i$. There are 14 distinct three-dimensional lattices (**Bravais lattices**); common metals are face-centred or body-centred cubic, silicon is diamond-cubic (two interpenetrating FCC lattices).

We learn a crystal's structure by diffraction. Fire X-rays (wavelength $\sim 0.1$ nm, comparable to atomic spacing) at it; the regularly spaced planes of atoms scatter coherently only in directions where the path difference is a whole number of wavelengths — **Bragg's law**, $2d\sin\theta = n\lambda$, with $d$ the plane spacing. The pattern of bright spots is the Fourier transform of the electron density, and inverting it gives the atomic positions. The natural space to describe the diffraction condition is the **reciprocal lattice** — the Fourier dual of the real lattice — and its primitive cell, the **first Brillouin zone**, is where all the electron and phonon physics is drawn.

## Phonons

The atoms aren't fixed; they vibrate about their lattice sites, and because they're coupled by bonds, the vibrations are **collective waves** running through the whole crystal, not independent jiggles. Solve the coupled oscillator problem (a chain of masses and springs is the 1D model) and you get a **dispersion relation** $\omega(\mathbf{k})$ — the frequency of a vibrational wave as a function of its wavevector. There are **acoustic** branches (atoms in a cell move together; $\omega \to 0$ as $\mathbf{k}\to 0$, these are sound waves) and, if the basis has more than one atom, **optical** branches (atoms in a cell move against each other).

Quantise each vibrational mode — it's a harmonic oscillator, so its energy comes in steps of $\hbar\omega$ — and each quantum is a **phonon**, a particle-like packet of lattice vibration carrying energy $\hbar\omega$ and momentum $\hbar\mathbf{k}$. Phonons carry heat (thermal conductivity in an insulator is phonon transport), scatter electrons (electrical resistance in a metal rises with temperature because there are more phonons to scatter off), and mediate the attraction that pairs electrons in a conventional [superconductor](#a-note-on-superconductivity).

This also fixes the heat-capacity problem classical physics couldn't. The **Debye model** treats the solid as a continuum of phonon modes up to a cutoff frequency; modes with $\hbar\omega > k_BT$ are frozen out (the [statistical-mechanics](/citadel/physics/statistical-mechanics) Boltzmann factor suppresses them), so $C_V \propto T^3$ at low temperature and approaches the classical $3Nk_B$ only when $k_BT$ exceeds the highest phonon energy.

## The free-electron model

Simplest picture of a metal: ignore the ion cores entirely, treat the valence electrons as a gas of free particles in a box (the **Drude** model classically, the **Sommerfeld** model with quantum statistics). The allowed states are plane waves with energy $\epsilon = \hbar^2 k^2/2m$, filled up — because electrons are [fermions](/citadel/physics/statistical-mechanics) obeying Pauli exclusion — from the bottom to a top energy called the **Fermi energy** $\epsilon_F$ (a few eV in a typical metal, corresponding to a "Fermi temperature" of tens of thousands of kelvin, so at room temperature the electron gas is highly degenerate).

This surprisingly crude model gets a lot right:

- **Electrical conductivity** — electrons drift under a field, scattering off impurities and phonons every $\sim 10^{-14}$ s; $\sigma = ne^2\tau/m$.
- **The Wiedemann–Franz law** — the ratio of thermal to electrical conductivity is proportional to $T$, with a universal constant, because the same electrons carry both.
- **Electronic heat capacity** — only electrons within $\sim k_BT$ of $\epsilon_F$ can be thermally excited (the rest have no empty states to move into), so the electronic $C_V$ is linear in $T$ and small — resolving another classical over-prediction.

What it can't explain: why some materials with valence electrons are *insulators*. For that you need the ions back.

## Band theory

Put the periodic potential of the ion lattice back in. **Bloch's theorem** says the electron wavefunctions in a periodic potential have the form $\psi_{\mathbf{k}}(\mathbf{r}) = e^{i\mathbf{k}\cdot\mathbf{r}}\, u_{\mathbf{k}}(\mathbf{r})$ — a plane wave modulated by a function $u_{\mathbf{k}}$ with the same periodicity as the lattice. The electron is still delocalised across the whole crystal, but its energy $\epsilon(\mathbf{k})$ is no longer the simple parabola.

Two complementary approximations show what happens:

- **Nearly-free electron** — start from free-electron plane waves and turn on a weak periodic potential. It matters most where waves Bragg-reflect off the lattice planes (at the Brillouin-zone boundary), and there it opens a **gap**: a range of energies with no allowed states. The parabola breaks into pieces separated by forbidden gaps.
- **Tight-binding** — start from isolated-atom orbitals and let electrons hop between neighbouring atoms. Each atomic level broadens into a **band** of width set by the hopping strength; the sharper the atomic level, the narrower the band.

Both give the same structure: **allowed energy bands separated by forbidden gaps**. Each band holds a fixed number of states — exactly $2N$ (spin up and down for each of $N$ unit cells).

## Metals, insulators, semiconductors

Now fill the bands with the available electrons, lowest first, and look at the highest one that has any electrons in it:

- **Metal** — the highest occupied band is only *partly* full. Electrons at the top have empty states just above them, so an applied field accelerates them and current flows. Conductivity is high and *rises* as temperature falls (fewer phonons to scatter off).
- **Insulator** — the highest occupied band (the **valence band**) is completely full, and the next empty band (the **conduction band**) is separated by a large gap ($\sim 5$ eV for diamond). A full band carries no net current — every forward-moving electron is balanced by a backward one — and there are no nearby empty states to redistribute into. An electric field does nothing until it's strong enough to rip electrons across the gap (dielectric breakdown).
- **Semiconductor** — same as an insulator but with a *small* gap ($\sim 1.1$ eV for silicon, $\sim 0.7$ eV for germanium). At $T = 0$ it's an insulator; at room temperature, thermal energy ($k_BT \approx 0.025$ eV) excites a small but non-negligible number of electrons across the gap, leaving empty states — **holes** — behind in the valence band. Both the promoted electrons and the holes conduct, and the number of them (hence the conductivity) rises exponentially with temperature, as $e^{-E_g/2k_BT}$.

Near a band edge the energy is again approximately parabolic, $\epsilon \approx \epsilon_0 + \hbar^2 k^2/2m^*$, but with an **effective mass** $m^*$ — which can be larger or smaller than the free-electron mass, and even negative near the top of a band (that's what makes a hole behave like a positive particle).

## Doping and the p–n junction

Pure ("intrinsic") silicon has too few carriers to be useful. **Doping** engineers the carrier count:

- Add a group-V atom (phosphorus, one extra valence electron) — an **n-type** semiconductor. The extra electron sits in a shallow donor level just below the conduction band and is easily thermally freed; electrons are the majority carrier.
- Add a group-III atom (boron, one fewer electron) — a **p-type** semiconductor. It creates an acceptor level just above the valence band that readily grabs an electron, leaving a hole; holes are the majority carrier.

Join a p-type and an n-type region and you get a **p–n junction**. Near the interface, electrons and holes diffuse across and recombine, leaving a **depletion region** of exposed dopant ions and a built-in electric field. This field lets current flow easily one way (forward bias) and blocks it the other (reverse bias) — a **diode**. Two junctions back to back, with the middle region's carrier population controlled by a third terminal, is a **transistor**: a switch or amplifier with no moving parts, and the reason band theory is arguably the most economically consequential piece of physics ever worked out. Photons with energy above the gap can also create electron–hole pairs (a solar cell, a photodiode) or, run in reverse, recombination can emit photons (an LED, a laser diode).

## A note on superconductivity

Below a critical temperature, some materials lose all electrical resistance and expel magnetic fields entirely (the **Meissner effect**). In the conventional (BCS) case, a phonon-mediated attraction overcomes the electrons' Coulomb repulsion and binds them into **Cooper pairs**; the pairs are bosons and condense into a single coherent quantum state that flows without scattering. High-temperature superconductivity in the copper-oxide materials is not fully explained and is an active research problem.

## The one idea to keep

Arranging atoms periodically forces the electrons' allowed energies into bands separated by forbidden gaps (Bloch's theorem, with gaps opening where electrons Bragg-reflect off the lattice). Fill the bands with the available electrons and look at the top one: partly full means a metal, completely full with a big gap above means an insulator, and completely full with a *small* gap means a semiconductor — where thermal excitation and, more usefully, deliberate doping control the number of electron and hole carriers. A p–n junction built from doped regions is a diode, and two of them make a transistor.
