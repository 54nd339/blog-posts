---
title: Turing Machines - Computability and Its Limits
description: The model that defines what "computable" means - the Turing machine and its instantaneous descriptions, why every reasonable variation has the same power, where recursive and recursively enumerable languages sit in the Chomsky hierarchy, and the diagonalization argument that makes the halting problem undecidable.
date: 2022-02-03
draft: false
slug: /automata/turing-machines
tags:
  - Theory of Computation
  - Computability
  - Automata
---

[Finite automata](/citadel/automata/regular-languages) have no memory beyond their state; [pushdown automata](/citadel/automata/context-free-grammar) have a stack, which is memory you can only reach from one end. Give a machine memory it can read and write anywhere, and you reach the ceiling: the **Turing machine**, Alan Turing's 1936 model of what a person following a fixed procedure with unlimited paper can compute. Nothing anyone has built or defined since computes more.

That is a strong claim, and this post is about both halves of it — why the model is as powerful as it is, and why that same power forces the existence of well-posed questions no machine can answer.

## The model

A deterministic Turing machine has:

1. A finite **control** holding a state $q \in Q$.
2. An **infinite tape** of cells, each holding one symbol from a finite tape alphabet $\Gamma$. The tape is input, output, and scratch memory all at once.
3. A **blank symbol** $\square \in \Gamma$ filling the tape outside the input. The input alphabet $\Sigma \subseteq \Gamma - \{\square\}$.
4. A **read-write head** over one cell, which each step reads that cell, writes a symbol, and moves one cell left or right.
5. A **transition function** $\delta : Q \times \Gamma \rightarrow Q \times \Gamma \times \{L, R\}$. It may be partial: if $\delta(q, a)$ is undefined, the machine **halts**.
6. A start state $q_0$ and a set of final states $F$.

An **instantaneous description** $x_1\,q\,x_2$ says the non-blank tape is $x_1 x_2$, the control is in state $q$, and the head is on the first symbol of $x_2$. A move rewrites the ID by one application of $\delta$: if $\delta(q_1, a) = (q_2, b, R)$ then $c\,q_1\,a\,d \vdash c\,b\,q_2\,d$. Write $\vdash^*$ for a run of zero or more moves.

### As a language accepter

Put $w \in \Sigma^+$ on an otherwise blank tape, start in $q_0$ with the head on the leftmost symbol. The machine **accepts** $w$ if it halts in a final state:

$$L(M) = \{w \in \Sigma^+ : q_0 w \vdash^* x_1\,q_f\,x_2,\ q_f \in F\}.$$

If it halts in a non-final state, or never halts at all, $w$ is not accepted. That third option — running forever — is new, and it is the whole story of this post.

A Turing machine can do everything the weaker models can and more. It accepts $00^*$ by walking right over `0`s to a blank. It accepts $\{a^n b^n : n \ge 1\}$ by repeatedly crossing off the leftmost `a` against the leftmost `b`, shuttling back and forth — more bookkeeping than a PDA, but no stack. And it accepts $\{a^n b^n c^n : n \ge 1\}$, which is [not context-free](/citadel/automata/context-free-grammar), by the same cross-off strategy across three blocks.

## The Church-Turing thesis

Chain enough basic Turing-machine operations together — copy a string, compare two numbers, add — and you can build up any procedure you could write in a programming language. This observation is elevated to a principle:

> **Church-Turing thesis.** Any computation that can be carried out by a mechanical procedure can be carried out by some Turing machine.

It is not a theorem — "mechanical procedure" is not a formal object — but a definition of what "algorithm" means. The evidence for adopting it: every other model anyone has proposed (Church's lambda calculus, Kleene's recursive functions, Post systems, register machines, real computers) turns out to have *exactly* Turing-machine power, able to simulate and be simulated by it. And in ninety years no one has described an intuitively effective procedure that no Turing machine can perform.

Accepting the thesis lets you define an **algorithm for a function** $f : D \rightarrow R$ as a Turing machine that, on any input $d \in D$, halts with $f(d)$ on the tape. Now "there is no algorithm for this problem" becomes a precise, provable statement.

## Every reasonable variation has the same power

You might hope to gain power by upgrading the machine. Almost nothing does:

- **Multiple tracks** — cells holding tuples — is just a bigger tape alphabet.
- **A "stay" option** for the head is $R$ then $L$.
- **A tape infinite in one direction only** simulates a two-way tape by folding the left half onto a second track.
- **A separate read-only input tape plus a work tape** (an off-line machine) is simulated with extra tracks tracking each head position.
- **$k$ tapes with $k$ independent heads** is simulated on one tape with $2k$ tracks — one for each tape's contents, one for each head's position. Efficiency drops; power does not.
- **A tape infinite in two dimensions** is simulated by storing `(contents, (x, y))` records on a linear tape.
- **Nondeterminism** — $\delta$ mapping to a *set* of moves, with acceptance if *any* branch halts in a final state — adds no power either. A deterministic machine does a breadth-first simulation: it keeps every reachable configuration after $k$ steps on its tape, and expands them all to get step $k + 1$. This is exponentially slower, but it accepts the same languages.

The lesson: *what* can be computed is astonishingly robust against changes to the machine. Only *how fast* moves.

## Where the languages sit

Turing machines carve out two language classes by how carefully they halt:

- **Recursively enumerable (RE), or Type 0.** $L$ is RE if some Turing machine accepts exactly $L$: it halts and accepts on every $w \in L$, and on $w \notin L$ it either rejects or loops forever. RE languages are precisely those generated by **unrestricted grammars** (productions $u \rightarrow v$ with $u$ containing at least one variable).
- **Recursive (decidable).** $L$ is recursive if some Turing machine *decides* it — accepts $L$ and **always halts**, accepting or rejecting every input. A recursive language has a genuine membership algorithm.

The relationships:

- Every recursive language is RE.
- If both $L$ and $\overline{L}$ are RE, then $L$ is recursive — run both machines in parallel; one must halt and tell you the answer.
- There are RE languages that are **not** recursive (the halting problem, below).
- There are languages that are **not even RE** — a counting argument: there are only countably many Turing machines, hence countably many RE languages, but uncountably many languages over $\Sigma$.

Between the context-free and the recursive languages sit the **context-sensitive languages (CSL, Type 1)**: generated by grammars whose productions never shrink ($|x| \le |y|$), equivalently accepted by a **linear bounded automaton** — a nondeterministic Turing machine confined to the tape cells its input started on. Every CSL is recursive, and some recursive languages are not context-sensitive.

Stacking these gives the **Chomsky hierarchy**:

| Type | Languages | Machine | Grammar |
| --- | --- | --- | --- |
| 3 | Regular | Finite automaton | Right/left-linear |
| 2 | Context-free | Pushdown automaton | Context-free |
| 1 | Context-sensitive | Linear bounded automaton | Non-contracting |
| 0 | Recursively enumerable | Turing machine | Unrestricted |

Each class is a proper subset of the next, with the recursive languages sitting strictly between Type 1 and Type 0.

## The halting problem

> **Halting problem.** Given a Turing machine $M$ and an input $w$, will $M$ halt when started on $w$?

There is no algorithm that answers this for all $(M, w)$. The proof is diagonalization — a machine turned against its own description.

1. Suppose a decider $H$ exists: given encodings $w_M\,w$, it halts in state $q_y$ if $M$ halts on $w$, and in $q_n$ if $M$ loops on $w$.
2. Build $H'$ from $H$: where $H$ would answer $q_y$, make $H'$ loop forever; where $H$ would answer $q_n$, make $H'$ halt.
3. Build $H''$: on input $w_M$, first copy it to $w_M\,w_M$, then run $H'$. So $H''$ **loops** on $w_M$ exactly when $M$ **halts** on its own description, and $H''$ **halts** on $w_M$ exactly when $M$ **loops** on its own description.
4. Run $H''$ on its own description $w_{H''}$. If $H''$ halts on $w_{H''}$, then by step 3 it must loop — contradiction. If it loops, then it must halt — contradiction.

The assumption in step 1 is the only free choice, so $H$ does not exist. The halting problem is undecidable; its language is RE (simulate $M$ on $w$ and accept if it stops) but not recursive.

## Reduction spreads the undecidability

Once you have one undecidable problem, you get many by **reduction**: if a decider for problem $B$ would let you build a decider for the known-undecidable $A$, then $B$ is undecidable too. Reducing from the halting problem knocks over a long list:

- **State-entry:** does $M$ ever enter state $q$ on input $w$? (Modify $M$ to enter a fresh state just before halting.)
- **Blank-tape halting:** does $M$ halt when started on a blank tape? (Given $(M, w)$, build $M_w$ that writes $w$ then runs $M$.)
- **Emptiness, finiteness, equality** for RE languages: is $L(M) = \emptyset$? is it finite? is $L(M_1) = L(M_2)$?
- **Rice's theorem** generalises all of these: *any* property of the language $L(M)$ that holds for some RE languages and fails for others is undecidable. "Is $L(M)$ regular?", "is $L(M)$ context-free?", "does $L(M)$ contain $\lambda$?" — all undecidable, for free.

The [context-free languages](/citadel/automata/context-free-grammar) inherit some of this: whether a CFG is ambiguous, whether two CFGs are equivalent, and whether a CFG's language is regular are all undecidable, several by reduction from the Post Correspondence Problem rather than directly from halting.

## The one idea to keep

The Turing machine is powerful enough to simulate any mechanical procedure — which is exactly why it can encode a question about itself and be trapped by the answer. Undecidability is not a gap waiting for a cleverer algorithm or a faster computer; it is a structural fact about any system expressive enough to describe its own computations. Knowing where that wall is — and that problems like [P versus NP](/citadel/algorithms/ComplexityClasses) live on the *decidable* side of it, asking not "can this be computed" but "can it be computed quickly" — is as much a part of the theory as the machines themselves.
