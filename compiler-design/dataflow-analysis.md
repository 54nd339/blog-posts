---
title: Data Flow Analysis - How Compilers Reason About Whole Functions
description: The analysis that makes global optimization safe - control flow graphs of basic blocks, data flow equations with gen and kill sets solved to a fixed point, and the four standard problems (reaching definitions, available expressions, live variables, copy propagation) classified as forward or backward, may or must.
date: 2022-12-03
draft: false
slug: /compiler-design/dataflow-analysis
tags:
  - Compilers
  - Optimization
  - Static Analysis
---

[Local optimizations](/citadel/compiler-design/code-optimisation) see one basic block at a time. To hoist a computation out of a loop or delete an assignment whose result is never used, the compiler has to know things that span many blocks: *is this value still the same when control reaches here?* *could this variable ever be read again?* **Data flow analysis** (DFA) computes those facts. It changes nothing itself — it is the intelligence the transformations run on.

This post covers the graph the analysis runs over, the equations it solves, and the four analyses that between them enable most global optimization.

## The control flow graph

First, a map of where execution can go. The **control flow graph** (CFG) has:

- **Nodes** — **basic blocks**: maximal straight-line instruction sequences. Control enters only at the first instruction (the *leader*) and leaves only at the last; no branches or halts in the middle.
- **Edges** — a directed edge `B1 -> B2` whenever `B2` can run immediately after `B1` (a jump to `B2`, or fall-through).
- **Entry and exit** — a distinguished entry node and one or more exit nodes.

For

```c
x = 5;
y = x + 2;
if (y > 0)  z = y * 10;
else        z = y / 2;
print(z);
```

the blocks are `B1: {x=5; y=x+2}`, `B2: {if y>0}`, `B3: {z=y*10}`, `B4: {z=y/2}`, `B5: {print(z)}`, with edges `entry->B1->B2`, `B2->B3`, `B2->B4`, `B3->B5`, `B4->B5`, `B5->exit`. The graph is the structure over which data flow facts propagate.

## Data flow equations

For each block `B`, the analysis computes two sets:

- $in[B]$ — the facts holding when control enters `B`.
- $out[B]$ — the facts holding when control leaves `B`.

A **transfer function** $f_B$ models what the block's instructions do to the facts:

$$out[B] = f_B(in[B]) \quad \text{(forward analysis)} \qquad in[B] = f_B(out[B]) \quad \text{(backward analysis)}$$

The transfer function is usually built from two per-block sets: $gen[B]$, the facts the block *creates*, and $kill[B]$, the facts it *invalidates*. A typical forward form is

$$out[B] = gen[B] \cup (in[B] - kill[B]).$$

Facts arriving from several predecessors are combined with a **meet operator**:

$$in[B] = \bigsqcap_{P \,\in\, pred(B)} out[P] \quad \text{(forward)} \qquad out[B] = \bigsqcap_{S \,\in\, succ(B)} in[S] \quad \text{(backward)}$$

The meet is **union** for a *may* analysis (a fact holds if it holds on *some* path) and **intersection** for a *must* analysis (a fact holds only if it holds on *every* path).

These equations are solved by an **iterative algorithm**: initialise all $in$/$out$ sets, repeatedly recompute every block's sets from the equations, and stop when nothing changes — a **fixed point**. Because each set only grows (or only shrinks) monotonically toward a bound, the iteration is guaranteed to terminate.

## The four standard analyses

### Reaching definitions

A definition `d` of variable `v` **reaches** point `p` if some path from `d` to `p` does not redefine `v`.

- Forward, *may* (meet = $\cup$).
- $gen[B]$: definitions in `B` that survive to its end. $kill[B]$: definitions elsewhere of variables `B` redefines.
- Enables **constant propagation** and detection of possibly-uninitialised variables.

### Available expressions

An expression `x op y` is **available** at `p` if *every* path from entry to `p` computes it and nothing since has redefined `x` or `y`.

- Forward, *must* (meet = $\cap$).
- $gen[B]$: expressions computed in `B` and still valid at its end. $kill[B]$: expressions whose operands `B` redefines.
- Enables **global common-subexpression elimination** — if `x op y` is available where it is recomputed, reuse the earlier value.

### Live variables

Variable `v` is **live** at `p` if some path from `p` reaches a use of `v` before any redefinition — its current value might still be read.

- Backward, *may* (meet = $\cup$); facts flow from uses back toward definitions.
- $use[B]$: variables read in `B` before any redefinition there. $def[B]$: variables written in `B` before any use.
- $in[B] = use[B] \cup (out[B] - def[B])$, with $out[B] = \bigcup_{S \in succ(B)} in[S]$.
- Enables **dead-code elimination** (an assignment to a variable not live afterward is dead) and **register allocation** (a dead variable's register is free).

### Copy propagation

After `x = y`, a later use of `x` can become `y` if neither `x` nor `y` is redefined in between. It relies on reaching-definitions information plus a check that the copy is the only definition of `x` that reaches the use.

### Induction variables

A related loop analysis identifies **induction variables** — those forming an arithmetic progression each iteration (`i` in `for (i=0; ...)`, or `j = j + 4` inside it) — and their relationships (`j = 4*i + c`). This drives **strength reduction** (replace `i * 4` with repeated addition), **induction variable elimination** (express several in terms of one basic variable), and loop-termination test replacement.

## The taxonomy

Every DFA problem is one of four kinds:

| | Forward | Backward |
| --- | --- | --- |
| **May** (meet $\cup$) | reaching definitions | live variables |
| **Must** (meet $\cap$) | available expressions | very busy expressions |

- **Forward** propagates with control flow; $out$ depends on $in$.
- **Backward** propagates against it; $in$ depends on $out$.
- **May** answers "on some path"; **must** answers "on all paths".

Knowing which cell a problem sits in tells you the direction to iterate and the operator to combine paths with.

## The one idea to keep

Global optimization is only safe if the compiler can prove a fact holds on *every* relevant path — that a value is truly dead, truly invariant, truly a known constant. Data flow analysis is how it gets that proof: set up gen/kill equations over the control flow graph, pick union or intersection for how paths merge, and iterate to a fixed point. The four analyses here cover most of what the [optimizer](/citadel/compiler-design/code-optimisation) and [code generator](/citadel/compiler-design/code-genration) need.
