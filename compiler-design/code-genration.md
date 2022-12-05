---
title: Code Generation - From Intermediate Code to Machine Instructions
description: The compiler's back end - the forms target code can take (absolute, relocatable, assembly), machine-dependent choices like instruction selection and scheduling, register allocation by graph colouring of an interference graph, the descriptor-based generation loop, and Sethi-Ullman labelling for expression trees.
date: 2022-12-05
draft: false
slug: /compiler-design/code-genration
tags:
  - Compilers
  - Code Generation
  - Optimization
---

By the time the [optimizer](/citadel/compiler-design/code-optimisation) is done, the program is a clean stream of three-address code: `t2 = id3 * t1`, `t3 = id2 + t2`. **Code generation** is the last phase — turning that into instructions a specific processor runs. It is where the compiler commits to *which* machine instructions, *which* registers, and *what* order, and those choices decide how fast the program runs.

This post covers the forms the output takes, the target-specific decisions, and the one that matters most: fitting values into a handful of registers.

## Forms of target code

- **Absolute machine code** — directly executable, all addresses fixed. Simple to load and run, but the load address must be known at compile time and separate modules cannot be combined easily. Used for small standalone programs and fixed-layout embedded systems.
- **Relocatable machine code** — instructions and data with addresses relative to a start point, plus a list of spots the **linker** must patch (calls to external functions, references to other modules' data). The linker combines relocatable object files and libraries; the **loader** places the result in memory, fixing relative addresses to absolute ones. This is the common format — it is what makes separate compilation and shared libraries possible.
- **Assembly language** — some compilers emit assembly and let an **assembler** produce machine code. It adds a step but keeps the compiler simpler and more portable, and the output is human-readable for debugging.

## Machine-dependent decisions

Code generation is inherently target-specific, and that is an opportunity:

- **Instruction selection.** The IR has generic operations; the target has a rich instruction set, often with addressing modes or compound instructions that do several IR operations at once (`a = b + c*4` in one instruction if the architecture has scaled-index addressing). Pick the fastest/smallest sequence for each IR operation.
- **Instruction scheduling.** On a pipelined processor, instruction order affects stalls. Reorder instructions (where data dependencies allow) to keep functional units busy and avoid pipeline bubbles.
- **Peephole optimization on target code.** A final sliding-window pass over the generated instructions: remove redundant loads and stores, drop unreachable code, simplify jump-to-jump chains, and swap in machine idioms.

## Register allocation

CPUs have very few **registers** — the fastest storage, directly addressable by instructions — typically 8 to 32. Keeping the right values in them is the single biggest lever on generated-code speed.

- **Register allocation** decides *which* variables and temporaries live in registers at each program point.
- **Register assignment** then binds each to a *specific* physical register.

When more values want registers than there are registers, some must be **spilled** to memory and **filled** (reloaded) later, at a cost. Allocation can be **local** (within a basic block) or **global** (across a whole function); global is harder but much better.

### Graph colouring

The standard global method turns allocation into [graph colouring](/citadel/algorithms/GraphColoring):

1. Build an **interference graph**: one node per variable/temporary; an edge between two nodes if they are ever **live** at the same time (from [live-variable analysis](/citadel/compiler-design/dataflow-analysis)) and so cannot share a register.
2. Colour the graph with `k` colours, `k` = number of available registers, so no two adjacent nodes get the same colour. Each colour is a register.
3. If no `k`-colouring exists, pick a node to **spill**, remove it, and retry.

Graph `k`-colouring is NP-hard in general, so real allocators use a heuristic (Chaitin's: repeatedly remove a node with fewer than `k` neighbours, colour on the way back, spill when stuck). Simpler allocators just use **usage counts** — keep the variables referenced most often in a loop or block in registers.

## The generation loop

Lowering three-address code is systematic. For `x = y op z`:

1. **Locate operands.** Where are the current values of `y` and `z`? If not in suitable registers, emit **load** instructions. A `getReg()` routine picks a register, spilling another value if necessary.
2. **Select the instruction** for `op` — `ADD` for `+`, and so on.
3. **Place the result.** If `x` is a temporary used soon, keep it in a register; if it is a program variable, it may need storing back to memory eventually.

To decide well, the generator maintains two tables:

- **Register descriptors** — for each register, which variables currently hold their value there.
- **Address descriptors** — for each variable, every location (a register, a memory address, a stack slot) where its current value can be found.

After emitting `MOV R0, y` and `ADD R0, R0, z` for `x = y + z` (with `x` in `R0`): the register descriptor for `R0` now says `x`, and the address descriptor for `x` says `R0`. These are consulted and updated instruction by instruction; the detailed scheme is the one in the Dragon Book (Aho et al.), section 8.6.

## Sethi-Ullman labelling

For evaluating one expression tree within a block, the **Sethi-Ullman algorithm** computes the minimum number of registers needed and an evaluation order that achieves it, with no spills.

Label the tree bottom-up:

- A leaf that is a left operand (needs a register): label 1. A right-child constant that can be an immediate operand: label 0.
- An interior node with children labelled $l_1$ (left) and $l_2$ (right): label $l_1 + 1$ if $l_1 = l_2$, otherwise $\max(l_1, l_2)$.

The root's label is the register count. The rule for generating code: evaluate the child with the *larger* label first (so its result occupies a register while the smaller side is computed), then apply the operator. If registers run short, the labels tell you which intermediate results to park in memory.

## The one idea to keep

The front end's job was analysis; the back end's is a sequence of resource-allocation problems on a specific machine — which instruction, which register, which order. Register allocation is the crux, and framing it as colouring an interference graph is what connects a compiler phase to the [complexity theory](/citadel/algorithms/ComplexityClasses) that says why it is hard and the heuristics that make it practical anyway.
