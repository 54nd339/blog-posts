---
title: Code Optimization - Making the Same Program Faster
description: How a compiler improves intermediate code without changing what it does - the scope levels from basic block to whole program, local techniques with DAGs and peephole passes, loop optimizations led by invariant code motion and strength reduction, and constant and copy propagation.
date: 2022-12-01
draft: false
slug: /compiler-design/code-optimisation
tags:
  - Compilers
  - Optimization
---

The [semantic analysis](/citadel/compiler-design/semantic-analysis) phase produces intermediate code that is correct but naive — it recomputes the same subexpression twice, evaluates `width * height` on every loop iteration, keeps assignments whose results are never read. **Code optimization** rewrites that intermediate representation to run faster, smaller, or with less power, *without changing what the program computes*.

This post covers where optimization is applied, the local transformations on a single block, the loop transformations that pay off most, and the propagation passes that feed them.

## Why, and what it costs

The goal is one of: **faster execution** (usually the priority), **smaller code** (loads quicker, less storage), **lower power** (mobile and embedded).

The one hard rule: **correctness first**. A faster program that returns the wrong answer is worse than a slow correct one.

The trade-offs:

- **Compile time.** Aggressive optimization can multiply build times. Worth it for release builds, not for the edit-compile-test loop.
- **Diminishing returns.** The 80/20 rule holds hard here — a few inner loops account for most of the runtime, so optimizing them is where the gains are.
- **Debuggability.** Optimized code no longer maps line-for-line to the source, which complicates stepping through it.
- **Portability.** Some transformations are machine-independent; others target one architecture's instructions and are done during [code generation](/citadel/compiler-design/code-genration).

## Scope

Optimizations apply over widening regions:

- **Local** — within one **basic block**, a straight-line instruction sequence with no branches in or out except at its ends. Simple and cheap.
- **Global (intraprocedural)** — across the basic blocks of one function, reasoning about how values flow between them. Needs [data flow analysis](/citadel/compiler-design/dataflow-analysis). Loop optimizations live here.
- **Interprocedural (whole-program)** — across function boundaries: inlining, cross-module dead-code elimination. Most powerful, most expensive.

## Local optimization

### DAG of a basic block

Represent the block's computations as a **directed acyclic graph**: interior nodes are operators, leaves are variables and constants, edges run from operands to operators.

```text
x = a + b
y = a + b
z = x * 2
```

`a + b` appears twice, so the DAG has *one* node for it with two parents — a **common subexpression**. Compute it once:

```text
t1 = a + b
x  = t1
y  = t1
z  = t1 * 2
```

The DAG also exposes safe instruction reorderings for better register use.

### Peephole optimization

Slide a small window over the instruction stream and replace short sequences with cheaper equivalents:

- **Redundant load/store** — `MOV R0, x` then `MOV x, R0` with `R0` untouched between: drop the second.
- **Constant folding** — `ADD R1, R1, #0` does nothing: remove it.
- **Strength reduction** — `MUL R1, R1, #2` becomes `SHL R1, R1, #1`.

## Loop optimization

Loop bodies run many times, so a small saving inside compounds.

### Loop-invariant code motion

Find computations inside the loop whose operands never change during it, and hoist them out:

```java
// before
for (int i = 0; i < n; i++) {
    int limit = width * height;   // same every iteration
    array[i] = i + limit;
}

// after
int limit = width * height;       // computed once
for (int i = 0; i < n; i++) {
    array[i] = i + limit;
}
```

`width * height` went from $n$ evaluations to one.

### Other loop transformations

- **Induction variable elimination** — variables that step by a constant each iteration (loop counters, `j = j + 4`) can often be collapsed to a single basic one, expressed as `4*i + c`.
- **Strength reduction in loops** — an array index `i * 4` recomputed each iteration becomes an add of `4` to the previous value.
- **Loop unrolling** — duplicate the body to cut loop-control overhead and expose parallelism, at the cost of code size.

## Folding and propagation

- **Constant folding** — evaluate constant expressions at compile time. `2 + 3` becomes `5`; `PI * SOME_CONST` folds if both are compile-time constants.
- **Constant propagation** — if `x` is assigned a constant and not reassigned before a use, substitute the constant. `currentUsers = 100; if (userCount < currentUsers)` becomes `if (userCount < 100)`, which may enable further folding.
- **Copy propagation** — after `y = x`, replace later uses of `y` with `x` (while neither is reassigned). This often makes `y` itself dead, so dead-code elimination removes the copy.

```java
int a = b;
int c = a + 5;   // → int c = b + 5;  and now 'a' may be dead
```

## The one idea to keep

Optimization is a stack of small, meaning-preserving rewrites applied over widening scopes — a peephole over three instructions, a DAG over one block, invariant motion over one loop, inlining over the whole program. The transformations themselves are easy to state; the machinery that makes the global ones *safe* — proving a value really is invariant, really is dead, really is a constant at that point — is [data flow analysis](/citadel/compiler-design/dataflow-analysis).
