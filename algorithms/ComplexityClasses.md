---
title: P, NP, NP-Complete, NP-Hard - A Map of Problem Difficulty
description: Complexity classes sort decision problems by how hard they are, not how clever your algorithm is. P is solvable fast, NP is checkable fast, NP-complete is the hard core of NP, and whether P equals NP is a million-dollar open question.
date: 2021-04-02
draft: false
slug: /algorithms/ComplexityClasses
tags:
  - Algorithms
  - Complexity
---

Big-O measures an *algorithm*. Complexity classes measure a *problem* — the best any algorithm could do. Some problems have fast solutions; some seem not to, and we can't prove it either way. The vocabulary for this — P, NP, NP-complete, NP-hard — is what tells you whether to keep looking for an efficient algorithm or switch to approximating.

These classes are defined for **decision problems**: questions with a yes/no answer ("does this graph have a Hamiltonian cycle?", "is this formula satisfiable?").

## P: solvable fast

**P** (polynomial time) is the set of decision problems a deterministic machine can *solve* in time bounded by a polynomial in the input size — $O(n)$, $O(n^2)$, $O(n^3)$, and so on. These are the problems we call tractable: sorting, [shortest paths](/citadel/algorithms/PathFinding), primality testing, [matching](/citadel/algorithms/MinimumSpanningTree). If a problem is in P, you can run it at scale.

## NP: checkable fast

**NP** (nondeterministic polynomial time) is the set of decision problems where, given a proposed "yes" answer plus a **certificate** (a witness), you can *verify* it in polynomial time. You may not be able to *find* the answer quickly — only check one you're handed.

A Hamiltonian cycle is the example: finding one seems to need exponential search, but if someone hands you a vertex ordering, checking that it's a valid cycle takes linear time. Boolean satisfiability (SAT) is another: verifying a satisfying assignment is trivial, finding one is not.

Every problem in P is also in NP — if you can solve it fast you can certainly check it fast (just solve it again). So $P \subseteq NP$. Whether the containment is strict is the open question.

## NP-complete: the hard core of NP

A problem is **NP-complete** if it is (1) in NP, and (2) **NP-hard** — every problem in NP reduces to it in polynomial time. NP-complete problems are the hardest in NP, and they're all equivalent: a polynomial algorithm for *any one* of them would give a polynomial algorithm for *all* of NP, proving $P = NP$.

Stephen Cook and Leonid Levin established the concept around 1971, proving SAT is NP-complete (the Cook–Levin theorem). Richard Karp then showed 21 more problems NP-complete by reducing SAT to them. The list now runs to thousands, including [the Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle), [graph coloring](/citadel/algorithms/GraphColoring) (decision version), [0/1 knapsack](/citadel/algorithms/01Kanpsack), vertex cover, and the travelling salesman decision problem.

![Two Euler diagrams: on the left, for P not equal to NP, P sits inside NP, NP-complete is the top of NP, and NP-hard extends beyond; on the right, for P equal to NP, the three collapse together](../images/complexity-classes-euler.png "The landscape under the two hypotheses P ≠ NP (left) and P = NP (right). Source: Wikimedia Commons")

## NP-hard: at least as hard as all of NP

A problem is **NP-hard** if every NP problem reduces to it — but it need not be in NP itself, and need not be a decision problem. The *optimization* version of the travelling salesman ("find the shortest tour", not "is there a tour under length $k$") is NP-hard. The halting problem is NP-hard *and undecidable* — strictly harder than anything in NP. If a problem is both NP-hard and in NP, it's NP-complete.

## P versus NP

Is $P = NP$? Can every quickly-checkable problem also be quickly solved? It's one of the Clay Institute's Millennium Prize Problems, worth a million dollars. Almost everyone believes $P \ne NP$ — that there really are problems you can verify but not find — but no proof exists. A proof of $P = NP$ would break most modern cryptography (which relies on certain problems being hard) and collapse a huge swath of optimization into tractability.

## Why it matters in practice

Recognising that your problem is NP-complete is useful information: it tells you to stop searching for a fast exact algorithm (assuming $P \ne NP$) and instead:

- **Approximate** — a polynomial algorithm with a proven bound on how far from optimal it can be (an [MST gives a 2-approximation](/citadel/algorithms/MinimumSpanningTree) for metric TSP).
- **Use heuristics** — [greedy](/citadel/algorithms/ActivitySelection), [local search, simulated annealing, genetic algorithms](/citadel/algorithms/AISearch) — good answers, no guarantee.
- **Exploit structure** — the general case is hard, but your instances may have a small parameter (fixed-parameter tractable) or a special shape (trees, planar graphs) that is not.
- **Solve small instances exactly** — [backtracking](/citadel/algorithms/NQueens) with good pruning handles surprisingly large cases before the exponential bites.

## Key takeaways

- Complexity classes rank *problems* by inherent difficulty, independent of any particular algorithm.
- **P**: solvable in polynomial time. **NP**: a proposed answer is checkable in polynomial time. $P \subseteq NP$.
- **NP-complete**: in NP and NP-hard; the hardest problems in NP, all polynomially equivalent.
- **NP-hard**: at least as hard as everything in NP, possibly harder, possibly not even decidable.
- **P vs NP** is open; the consensus is $P \ne NP$.
- Identifying an NP-complete problem redirects effort from exact solutions to approximation, heuristics, and special cases.
