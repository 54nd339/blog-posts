---
title: Logical Agents - Propositional and First-Order Reasoning
description: How an agent reasons instead of reacting - knowledge bases and the TELL/ASK interface, the Wumpus World, entailment and sound and complete inference, propositional logic with resolution and chaining, and the jump to first-order logic with objects, relations, and quantifiers.
date: 2023-02-04
draft: false
slug: /artificial-intelligence/logical-agents
tags:
  - Artificial Intelligence
  - Knowledge Representation
  - Logic
---

A reflex agent maps the current percept straight to an action. A **knowledge-based agent** does something else: it holds a set of sentences describing the world, and when it needs to act, it *derives* new sentences from them and acts on what it derived. Give it "squares next to a pit are breezy" and "I feel a breeze here", and it can conclude "a neighbouring square has a pit" — a fact nobody told it directly.

This post builds that machinery up: the knowledge-base interface, what it means for one sentence to *follow* from others, the two workhorse inference methods for propositional logic, and why serious knowledge representation needs the extra power of first-order logic.

## Knowledge-based agents

Two components:

- A **knowledge base** (KB) — a set of sentences in a formal language, representing what the agent knows. Facts, rules, and heuristics for when the rules run out.
- An **inference engine** — derives new sentences from the KB and decides what to do with them.

The agent runs on a **declarative** style: you `TELL` it what's true, and it `ASK`s itself what follows. `TELL` adds a sentence; `ASK` queries what the KB entails. You can describe such an agent at the **knowledge level** (what it knows) without committing to the **implementation level** (the data structures and algorithms underneath).

## The Wumpus World

The standard testbed. A 4×4 grid of caves holds one wumpus (deadly), some bottomless pits, and a heap of gold. PEAS:

- **Performance** — +1000 for climbing out with the gold, −1000 for dying, −1 per move, −10 for shooting the one arrow.
- **Environment** — squares adjacent to the wumpus smell (a *stench*); squares adjacent to a pit have a *breeze*; the gold's square *glitters*. Shooting along the row/column you face kills the wumpus if it's there.
- **Actuators** — turn left, turn right, move forward, grab, release, shoot.
- **Sensors** — stench, breeze, glitter, bump (walked into a wall), scream (wumpus died).

It's partially observable (you only sense your own square), deterministic, sequential, static, discrete, and single-agent.

The reasoning looks like this. Start in [1,1], sense nothing — so [1,2] and [2,1] are pit-free and wumpus-free. Move to [2,1], feel a breeze — a pit is in [1,1] (known safe), [2,2], or [3,1]. Back up, go to [1,2], feel no breeze — so [2,2] is safe, and the pit must be in [3,1]. The agent never *saw* the pit; it combined percepts with rules.

## What "follows from" means

Every logic has a **syntax** (which strings are well-formed sentences) and a **semantics** (which sentences are true in which "worlds"). A world, made precise, is a **model** — an assignment that fixes the truth of everything. We write $m \models \alpha$ when $\alpha$ is true in model $m$.

**Entailment**, $KB \models \alpha$: the KB entails $\alpha$ if $\alpha$ is true in *every* model where the KB is true. If the KB says "the Giants won" and "the Reds won", it entails "the Giants won or the Reds won", because there's no way for the KB to hold and that disjunction to fail.

**Inference**, $KB \vdash_i \alpha$: $\alpha$ can be *derived* from the KB by procedure $i$. Two properties matter:

- **Soundness** — $i$ only derives entailed sentences: $KB \vdash_i \alpha$ implies $KB \models \alpha$. It never lies.
- **Completeness** — $i$ derives every entailed sentence: $KB \models \alpha$ implies $KB \vdash_i \alpha$. It never misses.

## Propositional logic

The simplest useful logic. The world is a set of facts, each true or false.

- **Proposition symbols** — $P_{1,1}$ ("pit in [1,1]"), $B_{2,1}$ ("breeze in [2,1]"). Each is a sentence.
- **Connectives** — $\lnot S$ (not), $S_1 \land S_2$ (and), $S_1 \lor S_2$ (or), $S_1 \Rightarrow S_2$ (implies — false only when $S_1$ is true and $S_2$ false), $S_1 \Leftrightarrow S_2$ (iff — same truth value).

A **model** assigns true or false to every proposition symbol; truth tables then fix the truth of every compound sentence. Wumpus knowledge in this language:

$$\lnot P_{1,1}, \quad \lnot B_{1,1}, \quad B_{2,1}$$
$$B_{1,1} \Leftrightarrow (P_{1,2} \lor P_{2,1}), \quad B_{2,1} \Leftrightarrow (P_{1,1} \lor P_{2,2} \lor P_{3,1})$$

Three notions built on top:

- **Logical equivalence** — $\alpha \equiv \beta$ if they're true in exactly the same models. De Morgan's laws, commutativity, distributivity all hold.
- **Validity** — true in every model ($A \lor \lnot A$). The **deduction theorem**: $KB \models \alpha$ iff $(KB \Rightarrow \alpha)$ is valid.
- **Satisfiability** — true in at least one model. The link to inference: $KB \models \alpha$ iff $(KB \land \lnot \alpha)$ is *unsatisfiable*. Assume the opposite and derive a contradiction.

The brute-force inference method is **model checking** — enumerate all $2^n$ models and check. Correct, exponential. The alternatives apply sound inference rules instead.

## Resolution

**Resolution** is a single inference rule that is sound *and complete* for propositional logic, provided sentences are in **conjunctive normal form** (CNF) — a conjunction of *clauses*, each clause a disjunction of *literals* (a symbol or its negation).

The rule: from two clauses containing complementary literals ($\ell$ in one, $\lnot \ell$ in the other), infer the clause made of all the other literals. From $(P_{1,3} \lor P_{2,2})$ and $(\lnot P_{2,2})$, infer $P_{1,3}$.

Any sentence converts to CNF by a fixed procedure:

1. Eliminate $\Leftrightarrow$: replace $\alpha \Leftrightarrow \beta$ with $(\alpha \Rightarrow \beta) \land (\beta \Rightarrow \alpha)$.
2. Eliminate $\Rightarrow$: replace $\alpha \Rightarrow \beta$ with $\lnot \alpha \lor \beta$.
3. Push $\lnot$ inward with De Morgan and $\lnot\lnot\alpha \equiv \alpha$.
4. Distribute $\lor$ over $\land$.

The **resolution algorithm** proves $KB \models \alpha$ by contradiction: convert $KB \land \lnot \alpha$ to CNF and resolve pairs of clauses repeatedly. If you ever derive the empty clause, $KB \land \lnot \alpha$ is unsatisfiable, so $KB \models \alpha$.

## Forward and backward chaining

For KBs restricted to **Horn clauses** (at most one positive literal per clause — essentially "rules" $A_1 \land \dots \land A_k \Rightarrow B$ plus facts), two cheaper methods work, both sound and complete for that fragment:

- **Forward chaining** (data-driven) — start from the known facts, fire any rule whose premises are all satisfied, add its conclusion, repeat until the query appears or nothing new can be derived. It computes every consequence of the data.
- **Backward chaining** (goal-driven) — start from the query, find rules that conclude it, make their premises new sub-goals, recurse until every sub-goal is a known fact. It only touches facts relevant to the query, and it avoids loops by checking whether a sub-goal is already on the goal stack.

A Wumpus agent using propositional logic would `TELL` its KB rules like $B_{x,y} \Leftrightarrow (P_{x,y+1} \lor P_{x,y-1} \lor P_{x+1,y} \lor P_{x-1,y})$ and its current percepts, then `ASK` whether a given square is safe.

## First-order logic

Propositional logic's ceiling is expressiveness. "Every square next to a pit is breezy" has to be written out separately for all sixteen squares. **First-order logic** (FOL, predicate calculus) fixes this by assuming the world contains **objects** (people, numbers, squares), **relations** among them (`Brother`, `>`, `Adjacent`), and **functions** from objects to objects (`FatherOf`, `LeftLegOf`).

Syntax:

- **Constants** name objects (`KingJohn`, `2`). **Variables** stand for unspecified objects (`x`, `y`).
- **Functions** map terms to a term (`Sqrt(x)`); **predicates** map terms to true/false (`Brother(x, y)`, `IsRed(block)`).
- A **term** is a constant, a variable, or a function applied to terms. An **atomic sentence** is a predicate applied to terms, or an equality `term1 = term2`.
- The same connectives as propositional logic build complex sentences.
- **Quantifiers**: $\forall x$ ("for all") usually pairs with $\Rightarrow$, as in $\forall x\; \text{At}(x, NUS) \Rightarrow \text{Smart}(x)$ ("everyone at NUS is smart"); $\exists x$ ("there exists") usually pairs with $\land$, as in $\exists x\; \text{At}(x, NUS) \land \text{Smart}(x)$ ("someone at NUS is smart").

A sentence is true relative to a **model** (a set of objects, plus which relations and functions hold) and an **interpretation** (what each symbol refers to). $\forall x\; P$ is true if $P$ holds for every object; $\exists x\; P$ if it holds for at least one.

Order matters between different quantifiers: $\exists x\, \forall y\; \text{Loves}(x, y)$ ("someone loves everyone") is not $\forall y\, \exists x\; \text{Loves}(x, y)$ ("everyone is loved by someone"). Same-type quantifiers commute. The **duality**: $\forall x\; P \equiv \lnot \exists x\; \lnot P$.

FOL says a lot compactly:

- "Brothers are siblings": $\forall x, y\; \text{Brother}(x, y) \Leftrightarrow \text{Sibling}(x, y)$.
- "One's mother is one's female parent": $\forall m, c\; \text{Mother}(c) = m \Leftrightarrow (\text{Female}(m) \land \text{Parent}(m, c))$.
- "Every house has an owner": $\forall h\; \text{House}(h) \Rightarrow \exists o\; \text{Owner}(o, h)$.
- Wumpus causal rule: $\forall r\; \text{Pit}(r) \Rightarrow [\forall s\; \text{Adjacent}(r, s) \Rightarrow \text{Breezy}(s)]$ — one sentence, all squares.

You still `TELL` and `ASK`, but an answer to `ASK(KB, ∃a BestAction(a, 5))` comes back as a **substitution** binding the variable, like `{a/Shoot}`.

## Knowledge engineering

Building a real FOL knowledge base is a discipline of its own:

1. Identify the task the agent must perform.
2. Assemble the relevant domain knowledge.
3. Decide on a vocabulary of constants, predicates, and functions.
4. Encode general axioms about the domain.
5. Encode the specific problem instance.
6. Pose queries to the inference procedure.
7. Debug — refine axioms and facts against the answers you get.

## The one idea to keep

A logical agent separates *what it knows* from *how it reasons*. Entailment defines the target — the sentences that genuinely follow — and an inference procedure is judged by whether it hits that target without overshooting (sound) and without falling short (complete). Resolution and chaining do this for propositional logic; first-order logic keeps the same framework but lets you quantify over objects, so one axiom covers a whole domain instead of one square at a time. The next step, turning that knowledge into a plan of action, is [knowledge representation and planning](/citadel/artificial-intelligence/knowledge-rep).
