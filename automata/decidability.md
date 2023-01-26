---
title: Decidability and Reducibility - The Halting Problem and Beyond
description: Some questions have no algorithm - not because we are not clever enough, but provably. This is how you show it: reduce a problem already known to be unsolvable to the one in front of you, lean on Rice's theorem for anything about program behaviour, and place the rest on the ladder of the arithmetical hierarchy.
date: 2023-01-26
draft: false
slug: /automata/decidability
tags:
  - Theory of Computation
  - Computability
  - Automata
---

The [halting problem](/citadel/automata/turing-machines) is undecidable: no program can take an arbitrary program and input and correctly say whether it halts. Turing proved it by diagonalization in 1936. That single result is a lever. Once you have *one* problem you can't solve, you can show a whole landscape of others are just as hopeless — without redoing the diagonalization each time.

This post is the toolkit for that. The central move is **reduction**: if solving problem `B` would let you solve a known-unsolvable problem `A`, then `B` is unsolvable too. Then two big consequences — Rice's theorem, which kills essentially every question about what a program *does*, and the arithmetical hierarchy, which sorts the unsolvable problems by *how* unsolvable they are.

## Three tiers of language

Fix an alphabet and think of a decision problem as a **language** — the set of strings for which the answer is "yes."

- **Decidable** (recursive) — some Turing machine halts on *every* input and answers correctly. This is what "algorithm exists" means.
- **Recognizable** (recursively enumerable, r.e.) — some machine halts and accepts on every "yes" input, but on a "no" input it may reject *or run forever*. You can confirm membership; you can't always confirm non-membership.
- **Co-recognizable** (co-r.e.) — the complement is recognizable. You can confirm non-membership but not always membership.

The relationships:

- Decidable = recognizable **and** co-recognizable. If you can semi-confirm both "yes" and "no," run both semi-deciders in parallel; one halts, and you have your answer.
- The halting language `A_TM = { ⟨M, w⟩ : M halts on w }` is recognizable (simulate `M` on `w`; accept if it stops) but **not** decidable, so by the line above it is **not co-recognizable** either. Its complement — "M does *not* halt on w" — is not even recognizable.

That last fact is the source of most undecidability lower down: you cannot enumerate the non-halting computations.

## Reduction, precisely

A **mapping reduction** from `A` to `B`, written `A ≤_m B`, is a *computable* function `f` such that `w ∈ A ⟺ f(w) ∈ B`. It transforms questions about `A` into equivalent questions about `B`.

The two ways to use it:

- **Undecidability** — if `A ≤_m B` and `A` is undecidable, then `B` is undecidable. (A decider for `B` plus `f` would decide `A`.)
- **Non-recognizability** — if `A ≤_m B` and `A` is not recognizable, then `B` is not recognizable.

The craft is designing `f`. To show `E_TM = { ⟨M⟩ : L(M) = ∅ }` (does this machine accept nothing?) is undecidable, reduce `A_TM` to its complement: given `⟨M, w⟩`, build a new machine `M_w` that ignores its own input, runs `M` on `w`, and accepts iff `M` accepts. Then `L(M_w)` is everything if `M` accepts `w`, and empty otherwise. A decider for `E_TM` would now tell you whether `M` accepts `w`. Contradiction.

Almost every "this property of a program is undecidable" proof is a variant of that gadget: wrap the input machine so that its acceptance behaviour hinges on a halting question you're not allowed to answer.

## Rice's theorem: give up on behaviour

**Rice's theorem.** Any property of the *language* a Turing machine recognizes — any property that is not trivially true of all machines or false of all machines — is undecidable.

"Does `M` accept a finite set?" "Does `M` recognize a regular language?" "Does `M` accept the string `hello`?" "Are `M₁` and `M₂` equivalent?" All undecidable, immediately, with no new proof — they're non-trivial semantic properties.

The theorem is exactly about **semantic** properties (about `L(M)`), not **syntactic** ones (about the description `⟨M⟩`). "Does `M` have 15 states?" or "does `M` ever write a blank?" are syntactic and can be decidable. The dividing line is whether the property could differ between two machines that recognize the same language.

The practical reading: **static analysis cannot be both sound and complete** for any interesting behavioural question. Every real linter, type checker, and verifier is an approximation — it gives up on some inputs (rejecting safe programs) or accepts some bad ones, because Rice says it must.

## Problems that don't mention Turing machines

Undecidability isn't confined to self-referential machine questions.

- **Post's Correspondence Problem.** Given a set of domino tiles, each with a top string and a bottom string, is there a sequence of tiles (repeats allowed) whose top concatenation equals its bottom concatenation? Undecidable — and it reduces cleanly to grammar and automaton questions, which is how you prove *those* undecidable (e.g. "do two context-free grammars generate a common string?").
- **Hilbert's tenth problem.** Does a given multivariate polynomial equation have an integer solution? Undecidable (Matiyasevich, 1970), closing a problem posed in 1900.
- **The word problem for groups**, **tiling the plane with a given set of tiles**, **whether a given program is a virus under a precise behavioural definition** — all undecidable.

Once the halting problem is in the room, undecidability leaks into number theory, combinatorics, and geometry.

## The arithmetical hierarchy

Undecidable problems aren't all equally hard. Classify them by how many alternating unbounded quantifiers over a *decidable* predicate you need to express membership:

- **Σ⁰₁** — `∃y. R(x, y)` with `R` decidable. Exactly the recognizable languages. `A_TM` lives here.
- **Π⁰₁** — `∀y. R(x, y)`. The co-recognizable languages. "M halts on *every* input."
- **Σ⁰₂** — `∃y ∀z. R(x, y, z)`. "L(M) is finite" sits here (there exists a bound such that for all longer strings, M rejects).
- **Π⁰₂** — `∀y ∃z. R`. "L(M) is infinite," "M halts on infinitely many inputs."

Each level is strictly harder than the ones below; the hierarchy doesn't collapse. `Δ⁰₁ = Σ⁰₁ ∩ Π⁰₁` is exactly the decidable languages.

**Turing reduction** (`A ≤_T B`) is the looser tool that makes this precise: `A` is decidable *given an oracle* — a black box that answers `B` for free. An oracle for the halting problem lets you decide everything in Σ⁰₁ and Π⁰₁; but "does `M` halt on all inputs" (Π⁰₂) is still undecidable *even with* that oracle. Relative to a halting oracle there's a new, harder halting problem — the hierarchy is a tower of "halting problems for machines with the previous oracle."

## Where the line actually is

Undecidability is not a wall between "practical" and "theoretical." It's why compilers can't always tell you if a loop terminates, why you can't write a perfect deadlock detector, why `make` can't know in general whether a build script will finish, and why every security scanner is heuristic. The engineering response is the same everywhere: restrict the input language until the question *becomes* decidable (total functional languages, typed configuration, bounded model checking), or accept an approximation and be honest about which direction it errs.

## The one idea to keep

You prove a problem unsolvable by reducing a known-unsolvable one to it — build a computable gadget that turns "does this machine halt?" into an instance of your problem, and you're done. Rice's theorem hands you that reduction for free for anything about a program's behaviour, which is why sound-and-complete static analysis is impossible in principle. The arithmetical hierarchy then says the unsolvable problems still have a structure: they get strictly harder as you stack `∃∀∃…` quantifiers, and each level is the halting problem for the level below.
