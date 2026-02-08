---
title: Quantum Computing - Qubits, Gates, and the Famous Algorithms
description: A classical bit is 0 or 1; a qubit is a superposition of both, and n qubits hold 2^n amplitudes at once. The trick is that measurement collapses all of it to one answer, so a quantum algorithm has to arrange for interference to concentrate probability on the right answer before you look.
date: 2026-02-08
draft: false
slug: /miscs/quantum-computing
tags:
  - Quantum Computing
  - Algorithms
---

The pitch you hear — "a quantum computer tries all answers at once" — is half right and dangerously misleading. Yes, `n` qubits represent `2ⁿ` complex numbers simultaneously. But you can't read them out; a measurement returns just *one* `n`-bit string, chosen at random according to those numbers. A quantum algorithm is useful only if it uses **interference** to make the amplitudes of wrong answers cancel and the amplitude of the right answer grow, so that the one string you get is probably the one you want.

That constraint is why quantum computers help with a *specific* short list of problems — factoring, unstructured search, simulating quantum systems — and not with most of what computers do. This post is the model, the gates, the three landmark algorithms, and where the field actually stands.

## The qubit

A classical bit is 0 or 1. A **qubit**'s state is a unit vector in a 2D complex space:

$$|\psi\rangle = \alpha\,|0\rangle + \beta\,|1\rangle, \qquad |\alpha|^2 + |\beta|^2 = 1.$$

$\alpha$ and $\beta$ are **amplitudes** (complex numbers). On **measurement** in the computational basis, you get `0` with probability $|\alpha|^2$ and `1` with probability $|\beta|^2$, and the state collapses to whichever you got — the superposition is gone. Because a global phase is unobservable, a single qubit's state is a point on a sphere (the **Bloch sphere**): the north and south poles are $|0\rangle$ and $|1\rangle$, the equator is equal superpositions differing by phase.

`n` qubits live in a $2^n$-dimensional space: a general state is a superposition over all `2ⁿ` basis strings, each with its own amplitude. This exponential state space is where the potential power lives — and the measurement bottleneck is why it's so hard to use.

## Entanglement and no-cloning

Two qubits can be in a state that *cannot* be written as (state of qubit 1) ⊗ (state of qubit 2) — for example $\tfrac{1}{\sqrt2}(|00\rangle + |11\rangle)$, a **Bell state**. Measure the first qubit and you instantly know the second, even though neither had a definite value before. This **entanglement** is a genuine resource; algorithms and error correction depend on it, and it has no classical analogue.

The **no-cloning theorem**: there is no operation that copies an arbitrary unknown quantum state. This rules out the obvious error-correction strategy (keep spare copies) and is the basis of quantum key distribution's security.

## Gates and circuits

Quantum computation proceeds by applying **gates** — unitary (reversible, norm-preserving) linear operators — to the state, then measuring. Single-qubit gates rotate the Bloch sphere:

- **X** — the NOT gate, swaps $|0\rangle \leftrightarrow |1\rangle$.
- **Z** — flips the phase of $|1\rangle$.
- **Hadamard (H)** — maps $|0\rangle \to \tfrac{1}{\sqrt2}(|0\rangle + |1\rangle)$; it *creates* superposition and is in almost every algorithm's first step.
- **Phase / T** gates — small rotations about the Z axis.

The key two-qubit gate is **CNOT** (controlled-NOT): flip the target qubit iff the control is $|1\rangle$. Applied to a superposed control, it *entangles* the two qubits. `{H, T, CNOT}` is a **universal** set — any unitary can be approximated to any precision by a circuit of these. **Toffoli** (controlled-controlled-NOT) is universal for classical reversible logic and lets a quantum computer run any classical function reversibly.

A computation is a circuit: prepare $|00\dots0\rangle$, apply gates, measure. Because gates are unitary, the whole thing before measurement is reversible — no information is erased, which also means no heat is dissipated by the logic itself.

## Deutsch–Jozsa: proof of concept

Given a black-box function $f:\{0,1\}^n \to \{0,1\}$ promised to be either **constant** (same output for all inputs) or **balanced** (0 on half the inputs, 1 on the other half), decide which. Classically, worst case, you need $2^{n-1}+1$ queries. The quantum algorithm needs **one**: put all input qubits in equal superposition with Hadamards, query $f$ once (into the phase), Hadamard again, and measure — the interference produces $|00\dots0\rangle$ exactly when $f$ is constant. Contrived, but it was the first clear demonstration that quantum queries can beat classical ones by an exponential factor.

## Grover's algorithm: quadratic search speedup

Search an unstructured space of $N$ items for the one satisfying some condition (a database with no index, a SAT assignment, a hash preimage). Classically you check items one by one: $O(N)$. **Grover's algorithm** does it in $O(\sqrt N)$.

The mechanism: start in equal superposition over all $N$ items. Repeat about $\tfrac{\pi}{4}\sqrt N$ times: (1) an **oracle** flips the phase of the target item, (2) a "diffusion" operator reflects all amplitudes about their mean. Geometrically, each iteration rotates the state vector a small angle toward the target; after $\sqrt N$ rotations it's aligned, and a measurement returns the target with high probability. Overshoot and it rotates *past*, so the iteration count matters.

Quadratic, not exponential — but it applies to a huge class of problems, and it's provably optimal for unstructured search. Its practical bite: it halves the effective key length of symmetric ciphers (a 128-bit key has $\sim 64$ bits of quantum security), which is why post-quantum guidance is "double your symmetric key sizes."

## Shor's algorithm: factoring in polynomial time

This is the one that started the funding. Factoring an `n`-bit integer is believed to take super-polynomial time classically (the best known, the general number field sieve, is sub-exponential), and [RSA](/citadel/cyber-security/asymmetric-key) security rests on that. **Shor's algorithm** factors in polynomial time on a quantum computer.

It reduces factoring to **period-finding**: to factor $N$, pick a random $a$ and find the period $r$ of the function $x \mapsto a^x \bmod N$ (the smallest $r$ with $a^r \equiv 1$). From $r$, elementary number theory recovers a factor. The period is found by preparing a superposition over many $x$, computing $a^x \bmod N$ into a register, and applying the **quantum Fourier transform** — which maps a periodic amplitude pattern to sharp peaks at multiples of $1/r$, so a measurement reveals $r$. The QFT is the quantum speedup; it runs in $O(n^2)$ gates versus the classical FFT's $O(n\log n)$ *operations on $2^n$ numbers*.

Shor also breaks Diffie–Hellman and elliptic-curve cryptography (discrete logs are period-finding too). This is the entire motivation for **post-quantum cryptography** — lattice-based and hash-based schemes (see [advanced cryptography](/citadel/cyber-security/advanced)) whose hardness doesn't reduce to period-finding.

## Complexity: BQP

The class of problems a quantum computer solves efficiently is **BQP** (bounded-error quantum polynomial time). It contains **P**, and factoring sits in BQP but is not known to be in P. BQP is *not* known to contain **NP** — Grover only gives a quadratic speedup on NP-complete search, not an exponential one, and there's good evidence quantum computers cannot solve NP-complete problems efficiently. Quantum computing is a targeted advantage, not a universal one.

## Decoherence and error correction

The obstacle is that qubits are fragile: interaction with the environment (**decoherence**) scrambles amplitudes and phases in microseconds to milliseconds, and every gate is slightly imperfect. Classical error correction (make copies) is forbidden by no-cloning.

**Quantum error correction** instead spreads one **logical qubit** across many **physical qubits** and measures *parity checks* (which reveal that an error occurred, and where, without revealing the data and collapsing it). The **surface code** — physical qubits on a 2D grid with local checks — is the leading scheme; it has a **threshold**: if the physical error rate is below roughly 1%, adding more physical qubits per logical qubit drives the logical error rate arbitrarily low. Estimates for breaking RSA-2048 with Shor run to millions of physical qubits given current error rates — hence "not soon, but plan for it."

## Where it stands (NISQ)

Today's devices are **NISQ** — Noisy Intermediate-Scale Quantum — tens to a few hundred physical qubits, no full error correction, so circuit depth is limited before noise dominates. Hardware approaches: superconducting circuits (Google, IBM), trapped ions (Quantinuum, IonQ), neutral atoms (QuEra), and photonics. Near-term algorithms are **variational** — a shallow parameterised quantum circuit tuned by a classical optimiser — aimed at chemistry (**VQE**, estimating molecular ground-state energies) and optimisation (**QAOA**). Whether these beat classical methods on a useful problem is still an open question; the clearest expected payoff remains **simulating quantum systems** (materials, catalysts, drug binding), which is what Feynman proposed quantum computers for in the first place.

## The one idea to keep

`n` qubits hold `2ⁿ` amplitudes, but measurement collapses all of it to one random `n`-bit outcome — so a quantum algorithm is worthless unless it engineers interference to cancel the wrong answers and amplify the right one before you look. That's doable for a short list of structured problems: Grover gets a quadratic speedup on unstructured search (halving symmetric key strength), and Shor gets a polynomial-time factoring algorithm via quantum-Fourier period-finding (breaking RSA and ECC, hence post-quantum cryptography). It is not a general speedup — BQP is not believed to contain NP — and fragile qubits mean useful machines need error correction and millions of physical qubits.
