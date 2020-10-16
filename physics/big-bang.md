---
title: The Big Bang - Inflation, Nucleosynthesis, the CMB, and the Fate of the Universe
description: "The first threshold of cosmic history: the Planck scale, inflation and what it fixes, baryogenesis and the Sakharov conditions, primordial nucleosynthesis and the microwave background, the FLRW metric and Friedmann equations, the ΛCDM budget with its dark sectors, the thermal epochs from Planck time to structure formation, and how it all might end."
date: 2020-10-16
draft: false
slug: /physics/big-bang
tags:
  - Physics
  - Cosmology
---

"The universe began hot, dense, and expanding" sounds like a story you have to take on faith. It is not. Two of its consequences are things you can compute and check. Run the nuclear-physics of the [first twenty minutes](/citadel/physics/nuclear-physics) and it predicts the universe should be about 75% hydrogen and 25% helium by mass — which is what we measure, in gas clouds that have never been near a star. And the flash of light released when the cooling plasma first turned transparent, 370,000 years in, should still be arriving from all directions, stretched by the expansion into microwaves at about 2.7 K — which is the cosmic microwave background, mapped to one part in $10^5$. A model that predicts two numbers that precisely, from first principles, is doing physics, not mythology.

This is the first chapter of a longer story — cosmic history as thresholds of rising complexity. Later chapters cover [the lives of stars](/citadel/physics/stellar-astrophysics), [how the elements were forged](/citadel/physics/stellar-astrophysics), [the formation of the Solar System](/citadel/physics/solar-system-formation), and [the geological and human timeline](/citadel/physics/big-history). This one is the physics of the beginning: the hot dense start, the expansion, and where it is all headed.

![NASA timeline of the universe as a widening horn: quantum fluctuations and inflation at the narrow left end, the afterglow (CMB) at 375,000 years, the dark ages, the first stars at ~400 million years, then galaxy formation, with the horn flaring faster on the right as dark energy accelerates the expansion over 13.8 billion years.](../images/cmb-timeline.jpg "The universe's history: inflation, the CMB at ~375,000 years, the dark ages, first stars at ~400 million years, then structure formation and accelerating expansion. Source: NASA / Wikimedia Commons.")

## The earliest instant

**The Planck scale** is where a quantum theory of gravity becomes unavoidable and current physics has nothing to say. Combining $\hbar$, $G$, and $c$ gives the only scales with the right dimensions:
$$
\begin{aligned}
l_P &= \sqrt{\hbar G/c^3} \approx 1.6\times10^{-35}\ \text{m}, &
t_P &= \sqrt{\hbar G/c^5} \approx 5.4\times10^{-44}\ \text{s}, \\
m_P &= \sqrt{\hbar c/G} \approx 2.2\times10^{-8}\ \text{kg}, &
T_P &= \sqrt{\hbar c^5/G k_B^2} \approx 1.4\times10^{32}\ \text{K}.
\end{aligned}
$$
$t_P$ is the earliest moment the theory can even be posed for.

**Quantum fluctuations** — the [uncertainty principle](/citadel/physics/quantum) forbidding a field from being exactly zero with exactly zero rate of change — mean "empty" space fizzes with transient energy. **The vacuum state** in [quantum field theory](/citadel/physics/quantum-field-theory) is the lowest-energy state of the fields, and its energy density need not be zero; a non-zero vacuum energy acts gravitationally as a *repulsion*, pushing space apart. **The initial singularity** is the formal point of infinite density and curvature where the classical equations break down.

**Grand unification.** At high enough temperature the electromagnetic, weak, and strong forces may be a single force; as the universe cooled, that symmetry broke and the forces separated. Hypothetical X and Y bosons at the GUT scale would let quarks turn into leptons.

## Inflation

Between roughly $10^{-36}$ and $10^{-32}$ s the universe is thought to have expanded exponentially, driven by the energy of a metastable vacuum (the inflaton field), growing by a factor of $\sim e^{60}$ or more. Inflation resolves several puzzles at once:

- **Flatness** — any initial curvature is stretched away, leaving space very nearly flat.
- **Horizon** — regions now too far apart to have ever exchanged light were in causal contact before inflation, explaining why the microwave sky has the same temperature everywhere.
- **Structure** — quantum fluctuations present before inflation are stretched to macroscopic scale, seeding the density variations that later grew into galaxies.

## Baryogenesis

Matter and antimatter should have been made in equal amounts and annihilated to a bath of photons — yet matter remains. Sakharov's three conditions for generating the imbalance:

1. **Baryon number violation** — some process must change the net baryon count.
2. **C and CP violation** — matter and antimatter, and their mirror images, must behave slightly differently.
3. **Departure from thermal equilibrium** — the imbalance-generating reactions must run while conditions are changing, not settled.

The observed excess is about one extra baryon per billion photons.

## The first elements and the first light

**Big Bang nucleosynthesis.** Between about one second and twenty minutes, the universe was a nuclear furnace. Protons and neutrons fused into the lightest nuclei: by mass roughly 75% hydrogen, 25% helium-4, with traces of deuterium, helium-3, and lithium-7. The predicted abundances match observation closely — one of the strongest pieces of evidence for the hot Big Bang. The [physics is the same fusion](/citadel/physics/nuclear-physics) that runs in stars, run once, everywhere, as the universe cooled through the right temperature window.

**The cosmic neutrino background.** About one second in, the universe became transparent to neutrinos; they have streamed freely ever since, a relic background analogous to the CMB but far harder to detect.

**Recombination and the CMB.** For about 370,000 years the universe was an opaque plasma — photons could not travel without scattering off free electrons. When it cooled to about 3000 K, electrons and nuclei combined into neutral atoms (**recombination**) and the photons were released. Stretched by expansion from visible/infrared down to microwaves, that light is the **cosmic microwave background**: a nearly uniform glow with tiny temperature anisotropies, the imprint of the inflation-seeded density variations.

![All-sky map of the cosmic microwave background: an oval projection speckled with hot and cold spots at the level of one part in 100,000 around the mean 2.7 K.](../images/cmb-map.png "The CMB temperature anisotropies (WMAP, 9-year). Red is hotter, blue cooler, by about ±200 µK around 2.7 K — the seeds of all later structure. Source: NASA / Wikimedia Commons.")

## The geometry: FLRW and Friedmann

**Hubble's law.** Distant galaxies recede at a speed proportional to their distance,
$$ v = H_0 d $$
with $H_0$ the present expansion rate — the signature of an expanding universe.

**The cosmological principle** — on large scales the universe is **homogeneous** (same everywhere) and **isotropic** (same in all directions) — leads to the **Friedmann–Lemaître–Robertson–Walker metric**,
$$ ds^2 = -c^2\,dt^2 + a(t)^2\left(\frac{dr^2}{1 - kr^2} + r^2\,d\Omega^2\right) $$
where $a(t)$ is the **scale factor** setting how distances stretch and $k = 0, +1, -1$ is flat, spherical, or hyperbolic spatial curvature. Feeding this into [general relativity](/citadel/physics/astrodynamics-advanced) gives the **Friedmann equation** governing the expansion:
$$ \left(\frac{\dot a}{a}\right)^2 = \frac{8\pi G}{3}\rho_{\text{total}} - \frac{kc^2}{a^2} + \frac{\Lambda c^2}{3} $$
with $\rho_{\text{total}}$ the combined density of radiation, matter, and dark energy, and $\Lambda$ the cosmological constant.

## The ΛCDM budget

The standard model of cosmology is **ΛCDM**: the hot Big Bang plus a cosmological constant $\Lambda$ (dark energy) and cold dark matter. The present energy budget is roughly:

| Component | Share | What it is |
|---|---|---|
| Dark energy ($\Lambda$) | ~68% | drives the *accelerating* expansion (seen in distant supernovae, late 1990s) |
| Cold dark matter | ~27% | gravitates but does not interact electromagnetically; the scaffolding for galaxy rotation, clustering, and structure |
| Ordinary matter | ~5% | atoms — stars, gas, planets, us |

Both dark components are inferred only gravitationally, and the nature of each is unknown.

## From smooth to lumpy

The early universe was almost perfectly uniform. **Perturbation theory** tracks how the tiny inflation-seeded density variations grew under gravity over billions of years. A region collapses when gravity overcomes its pressure support — the **Jeans instability** — above the **Jeans length**
$$ \lambda_J = \sqrt{\frac{\pi v_s^2}{G\rho_0}} \approx \sqrt{\frac{15 k_B T}{4\pi G\mu m_H \rho_0}} $$
($v_s$ the sound speed, $\mu$ the mean molecular weight). The same criterion governs the collapse of a gas cloud into a star.

## The thermal epochs

| Epoch | Time | Temperature | What happens |
|---|---|---|---|
| Planck | $<10^{-43}$ s | $>10^{32}$ K | quantum gravity; theory unknown |
| GUT | $10^{-43}$–$10^{-36}$ s | $10^{32}$–$10^{28}$ K | gravity already separate; strong/electroweak still unified |
| Inflation | $10^{-36}$–$10^{-32}$ s | — | exponential expansion; fluctuations stretched to cosmic scale |
| Electroweak | to $10^{-12}$ s | to $10^{15}$ K | strong force distinct; EM and weak still unified; $W$, $Z$ still massless |
| Quark | to $10^{-5}$ s | to $10^{12}$ K | electroweak symmetry breaks, $W$/$Z$ gain mass; quark–gluon plasma; baryogenesis |
| Hadron | to $1$ s | to $10^{10}$ K | quarks bind into protons and neutrons; antimatter annihilates, leaving the baryon excess |
| Lepton | $1$–$10$ s | to $10^9$ K | neutrinos decouple; neutron:proton ratio freezes near 1:6 |
| Nucleosynthesis | to $\sim20$ min | $10^9$–$10^7$ K | light nuclei form |
| Photon | to 370,000 yr | to 3000 K | opaque plasma of nuclei, electrons, photons |
| Recombination | ~370,000 yr | ~3000 K | neutral atoms form; universe becomes transparent; CMB released |
| Dark Ages | to ~150 Myr | to ~60 K | neutral gas, no stars; only 21-cm hydrogen emission |
| Reionisation | ~150 Myr–1 Gyr | — | first stars and quasars ionise the gas again |
| Structure formation | 1 Gyr–now | to 2.7 K | the cosmic web assembles: galaxies, clusters, superclusters, voids |

## The far future

- **Stelliferous era** — the current star-forming age, lasting perhaps $10^{14}$ years until the last stars burn out.
- **Degenerate era** — only white dwarfs, neutron stars, and black holes remain; if the proton decays, even these dissolve into leptons and photons over $10^{30}$–$10^{40}$ years.
- **Black hole era** — black holes are the last structures, themselves evaporating by [Hawking radiation](/citadel/physics/astrodynamics-advanced) over up to $\sim10^{100}$ years.
- **Dark era** — a cold, near-empty bath of photons, neutrinos, and stray leptons approaching maximum entropy: the **heat death**, the thermodynamic [end state](/citadel/physics/thermodynamics).

Alternative endings depend on dark energy and the vacuum. A **Big Rip** if dark energy strengthens with time until it tears apart galaxies, then atoms, then spacetime. A **Big Crunch** (disfavoured by current data) if expansion reverses into a collapse. **False-vacuum decay** if our vacuum is only metastable: a bubble of true vacuum could nucleate and expand at light speed, rewriting physics — thought to be extraordinarily unlikely.

## The one idea to keep

The hot Big Bang is a testable physical model, not a creation myth: it predicts the primordial 75%/25% hydrogen/helium split and the 2.7 K microwave background, both confirmed to high precision. Inflation — a brief burst of exponential expansion — flattens space, explains why the sky has one temperature, and stretches quantum fluctuations into the seeds of galaxies. General relativity's FLRW metric and the Friedmann equation govern the expansion, driven by a budget that is only ~5% ordinary matter, ~27% dark matter, ~68% dark energy — the last accelerating the expansion and pointing toward a cold, dilute heat death, unless the dark sector holds surprises.
