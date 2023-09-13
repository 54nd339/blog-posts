---
title: Advanced Cryptography - Zero-Knowledge Proofs and Post-Quantum Lattices
description: Two frontiers of cryptography - zero-knowledge proofs including SNARKs and STARKs and how they become non-interactive, and lattice-based cryptography as the leading answer to the threat quantum computers pose to RSA and elliptic curves.
date: 2023-09-13
draft: false
slug: /cyber-security/advanced
tags:
  - Security
  - Cryptography
  - Public-Key
---

Two areas of cryptography are moving fast enough that they deserve their own post. **Zero-knowledge proofs** let one party convince another that a statement is true while revealing nothing else, and recent constructions make those proofs tiny and non-interactive. **Lattice-based cryptography** is the leading candidate to replace RSA and elliptic curves once a large quantum computer exists. This post covers both.

## Zero-knowledge proofs

A **zero-knowledge proof** lets a **prover** convince a **verifier** that they know a secret, or that a statement is true, without revealing the secret or anything beyond the statement's truth. Three properties define it:

1. **Completeness** — if the statement is true, an honest prover convinces an honest verifier.
2. **Soundness** — if the statement is false, a cheating prover cannot convince an honest verifier, except with negligible probability.
3. **Zero-knowledge** — the verifier learns nothing except that the statement holds.

The "Ali Baba cave" gives the intuition: Peggy proves she knows the word that opens a door deep in a ring-shaped cave by always emerging from whichever side Victor calls out, without ever saying the word. The [network security](/citadel/cyber-security/network-security) post introduces this at a high level; here is the structure.

### Interactive versus non-interactive

- **Interactive ZKPs** need several rounds of back-and-forth — the verifier sends random challenges, the prover responds, and the responses demonstrate knowledge. The cave is interactive. The need for a live exchange limits where it can be used.
- **Non-interactive ZKPs (NIZKs)** produce a single proof string that anyone can verify later without contacting the prover. Two ways to get there: the **random oracle model**, where the **Fiat-Shamir heuristic** replaces the verifier's random challenges with the output of a hash function (this is exactly how a Schnorr identification protocol becomes a [Schnorr signature](/citadel/cyber-security/digital-signatures)); and the **common reference string** model, where prover and verifier share a trusted random string generated in a setup phase.

### Proofs versus arguments

A **proof** has soundness even against a computationally unbounded cheating prover. An **argument** has soundness only against a bounded prover — which is what matters in practice, since real attackers are bounded. Most deployed "ZKPs" are technically arguments.

### SNARKs and STARKs

**ZK-SNARK** — Zero-Knowledge Succinct Non-interactive ARgument of Knowledge.

- **Succinct** — the proof is very small and verification is very fast, regardless of how complex the statement is. This is the headline property.
- **Non-interactive**, and an **argument of knowledge** — the prover demonstrates knowledge of a secret **witness** that makes the statement true.
- **Trusted setup** — many SNARK constructions need an initial setup to generate public parameters. If the randomness used in setup is not destroyed, whoever has it can forge proofs. This is a real operational risk.
- Applications: privacy-preserving cryptocurrencies (Zcash), and scaling blockchains by verifying an off-chain computation with a small on-chain proof.

**ZK-STARK** — Zero-Knowledge Scalable Transparent ARgument of Knowledge.

- **Scalable** — proof generation and verification scale quasi-linearly (verification even poly-logarithmically) with the size of the computation, suiting very large statements.
- **Transparent** — **no trusted setup**; the public parameters come from publicly verifiable randomness. This removes the SNARK's main operational weakness.
- Security rests on simpler assumptions — collision-resistant hash functions — which also gives better post-quantum prospects. Proofs are larger than SNARKs, though shrinking.

### Building blocks

ZKP constructions draw on hard mathematical problems (discrete logarithm, factoring, elliptic-curve pairings), **commitment schemes** (commit to a value while hiding it, reveal it later), **polynomial commitments** (commit to a polynomial and later prove its value at chosen points without revealing the whole thing), and probabilistically checkable proofs.

### More applications

Beyond authentication: **privacy-preserving credentials** (prove you hold a valid licence without revealing your name or address), **verifiable computation** (prove an untrusted server ran a computation correctly without re-running it), enforcing honest behaviour in multi-party protocols, and both privacy and scalability (rollups) in blockchains.

## Post-quantum cryptography

A large-scale quantum computer would break most public-key cryptography in use. **Shor's algorithm** factors large integers efficiently, defeating RSA, and an adaptation solves the discrete logarithm and elliptic-curve discrete logarithm problems, defeating Diffie-Hellman, DSA, ECDH, and ECDSA. Symmetric ciphers and hashes are affected far less — roughly a doubling of key size restores their margin.

**Post-quantum cryptography** is the search for algorithms secure against both classical and quantum attackers. Lattice-based cryptography is the most developed family.

### Lattices

A **lattice** is a regular infinite grid of points in $n$-dimensional space. Formally, given linearly independent basis vectors $B = \{\mathbf{b}_1, \ldots, \mathbf{b}_n\}$, the lattice is every integer combination of them:

$$L(B) = \left\{ \sum_{i=1}^{n} x_i \mathbf{b}_i \;\middle|\; x_i \in \mathbb{Z} \right\}$$

Picture the points on infinite graph paper, or the atoms in a perfect crystal. The [linear algebra](/citadel/maths/linear-algebra) of bases carries directly over.

### Hard lattice problems

- **Shortest Vector Problem (SVP)** — given a basis, find the shortest non-zero lattice vector. Exact SVP is NP-hard; approximating it within useful factors is believed hard too.
- **Closest Vector Problem (CVP)** — given a target point not on the lattice, find the nearest lattice vector. Also NP-hard.
- **Learning With Errors (LWE)** — the workhorse. There is a secret vector $\mathbf{s}$. You are given many samples $(\mathbf{a}_i, b_i)$ where $\mathbf{a}_i$ is random and $b_i = \langle \mathbf{a}_i, \mathbf{s} \rangle + e_i \bmod q$, with $e_i$ a small random error. Recover $\mathbf{s}$. Without the error this is just linear algebra; the error makes it hard, and provably as hard as worst-case lattice problems like SVP. It is believed hard for quantum computers.
- **Ring-LWE** — LWE over polynomial rings, which gives smaller keys and faster operations while keeping strong security. Most practical lattice schemes build on Ring-LWE or the related Module-LWE.

### Why lattices

- **Quantum resistance** — the core problems resist Shor's algorithm and other known quantum algorithms.
- **Versatility** — lattices yield public-key encryption, key encapsulation mechanisms, signatures, and almost all [fully homomorphic encryption](/citadel/cyber-security/public-key).
- **Security proofs** — many schemes reduce to worst-case lattice problems: breaking the scheme is at least as hard as solving a known-hard problem in the worst case.
- **Efficiency** — modern Ring-LWE and Module-LWE constructions are competitive with RSA and ECC in key size, speed, and bandwidth.

### Standardised schemes

NIST's post-quantum standardisation project selected several lattice schemes, published in 2024:

- **ML-KEM** (FIPS 203), based on **CRYSTALS-Kyber**, a key encapsulation mechanism built on Module-LWE.
- **ML-DSA** (FIPS 204), based on **CRYSTALS-Dilithium**, a signature scheme on Module-LWE.
- **Falcon** (standardisation as FN-DSA underway), a compact signature scheme on NTRU lattices, related to the Short Integer Solution problem.

**NTRUEncrypt** is one of the oldest and most studied lattice encryption schemes. (NIST also standardised the hash-based signature **SLH-DSA** in FIPS 205, which is not lattice-based.) Migration to these standards is a multi-year effort to protect data whose confidentiality must outlast the arrival of quantum computers.

## The one idea to keep

Zero-knowledge proofs decouple *convincing someone of a fact* from *revealing why it is true*, and SNARKs and STARKs make the proof small enough to put on a blockchain — the main difference between them is that STARKs need no trusted setup. Lattice cryptography bets that finding short vectors in a high-dimensional grid stays hard even for a quantum computer, and that bet is now standardised in ML-KEM and ML-DSA.
