---
title: Instruction-Level Parallelism - Superscalar, VLIW, and Vectors
description: How processors run several instructions from one thread at once - the dependencies that limit it, hardware out-of-order scheduling with Tomasulo's algorithm and register renaming, compiler techniques like loop unrolling and software pipelining, and the superscalar, VLIW, and vector architectures built on them.
date: 2022-04-07
draft: false
slug: /computer-architecture/ilp
tags:
  - Computer Architecture
  - Parallelism
  - CPU
---

[Pipelining](/citadel/computer-architecture/pipelining) overlaps the *stages* of consecutive instructions but still finishes about one per cycle. **Instruction-level parallelism (ILP)** goes further: issue several instructions per cycle from a single thread, and let them finish in whatever order their inputs allow. A modern core sustains three to four instructions per cycle this way.

The ceiling is dependencies — an instruction that needs a result another hasn't produced can't run early. Everything in this post is a way to see around dependencies, or to remove the ones that were never real: hardware that reorders execution at runtime, compilers that rearrange code to expose independent work, and three architecture styles — superscalar, VLIW, and vector — that pick different splits of that labour.

## What limits ILP

Three kinds of dependency between instructions:

- **True data dependency (RAW).** An instruction needs the value a previous one writes: `ADD R1, R2, R3` then `SUB R4, R1, R5`. Real; must be respected.
- **Name dependency (WAR, WAW).** Two instructions reuse the same register *name* with no data flowing between them — write-after-read and write-after-write. False; an artefact of having only so many architectural registers, and removable.
- **Control dependency.** Whether an instruction runs depends on a branch outcome.

On top of that, each instruction has a **latency** — multi-cycle operations like loads and floating-point divides. ILP techniques try to fill those latency gaps with other useful work.

## Finding parallelism in hardware

**Multiple issue.** Fetch, decode, and begin executing more than one instruction per clock.

**Dynamic (out-of-order) scheduling.** When an instruction stalls on an operand, let independent later instructions execute past it:

- **Scoreboarding** — a central table tracks every in-flight instruction's status and the availability of operands and functional units, releasing instructions to execute when both are ready. It handles RAW dynamically but *stalls* on WAR and WAW hazards.
- **Tomasulo's algorithm** — each functional unit has **reservation stations** (small buffers) and results are broadcast on a **common data bus**. A reservation station waiting on an operand holds a *tag* naming the station that will produce it, not a register number — which is **register renaming in hardware**. WAR and WAW hazards disappear (each pending value has its own tag), and RAW is handled by waiting for the tagged broadcast.

**Speculative execution.** Run instructions past a predicted branch (or a predicted memory address) before it's certain they should run. Results are held in a buffer and **committed** to the architectural registers only once the prediction is confirmed; on a misprediction they're flushed.

**Hardware register renaming.** The chip has many more physical registers than the architectural ones a program names. Each instruction's destination is mapped to a fresh physical register, so instructions that target the same architectural register no longer collide. A **reorder buffer** then commits results in program order, which is what keeps exceptions [precise](/citadel/computer-architecture/pipelining) despite out-of-order execution.

All of this is fed by [branch prediction](/citadel/computer-architecture/pipelining) — without a good predictor the window of instructions to reorder stays tiny.

## Finding parallelism in software

- **Instruction scheduling** — the compiler reorders instructions (within a basic block, or across blocks) to separate dependent pairs and fill latency gaps.
- **Loop unrolling** — replicate the loop body $N$ times per iteration. Fewer branch and counter instructions per unit work, and a bigger pool of independent instructions for the hardware to overlap. See [code optimisation](/citadel/compiler-design/code-optimisation).
- **Software pipelining** — restructure the loop so each iteration of the *new* body contains instructions drawn from *different* iterations of the original, keeping every pipeline stage busy. It needs a **prologue** to fill the pattern, a steady-state **kernel**, and an **epilogue** to drain it — effective for loops without carried dependencies.
- **Compiler register renaming** — assign distinct registers to variables that would otherwise create WAR or WAW hazards, giving the scheduler more freedom.

## Superscalar

A **superscalar** processor has several independent functional units — multiple integer ALUs, a floating-point unit, load/store units — and issues several ready instructions to them each cycle. It leans on dynamic scheduling to sort out dependencies among the instructions in flight, and on branch prediction and speculation to keep the units fed. Instructions finish out of program order but **commit** in order through a reorder buffer, preserving precise exceptions. The cost is hardware complexity: dependency checking across a wide issue window, resource allocation, and out-of-order bookkeeping all grow fast.

## Superpipelining

Instead of widening, go deeper: split each pipeline stage into $m$ shorter sub-stages. Depth becomes $mk$ and the cycle time drops toward $\tau/m$, so the clock runs faster and — hazards permitting — throughput rises. It composes with superscalar: several deep pipelines side by side.

## VLIW

**Very long instruction word** machines move the parallelism-finding entirely to the compiler. The compiler identifies independent primitive operations and packs them into one wide instruction, with a slot per functional unit — one integer op, one FP op, one memory op, issued together. Scheduling is **static**: the hardware just dispatches the slots, so it's much simpler than a superscalar.

The trade-offs land on the compiler and the binary:

- the compiler carries the full burden of dependence analysis and scheduling;
- if it can't fill every slot, it pads with NOPs, hurting code density;
- in early VLIWs, one stalled operation (a cache miss) stalls the whole word, since all slots issue in lock-step;
- a binary is compiled for a specific number and mix of functional units and doesn't port to a different one.

Superscalar versus VLIW is really one question — *who finds the parallelism* — answered "hardware, at runtime" or "compiler, at compile time".

## Data parallelism: array and vector processors

When the *same operation* applies to many data elements, that's **SIMD** in [Flynn's taxonomy](/citadel/computer-architecture/multiprocessor), and two architectures specialise in it.

**Array processors** are a grid of identical **processing elements**, each with its own ALU and local memory. A single control unit runs scalar code itself and *broadcasts* data-parallel instructions to all PEs, which execute them in lock-step on their local data. An interconnection network moves data between PEs when the computation needs it.

**Vector processors** have instructions that operate on whole one-dimensional arrays: `VADD V1, V2, V3` adds every element of two vector registers. Those registers hold many elements (64–128); the functional units are deeply pipelined so that, once filled, they produce roughly one result per cycle for a long vector. Vector load/store instructions handle strided access and scatter/gather, and the compiler's job is **vectorising** scalar loops into these instructions. Today's SIMD instruction-set extensions and [GPUs](/citadel/parallel-computing/parallel-architect) are direct descendants.

## The one idea to keep

ILP is the art of pretending a sequential instruction stream is more parallel than it looks. True data dependencies are the hard floor; name dependencies are just register-naming and get erased by renaming; control dependencies get predicted around. The three architecture families differ only in where the reordering happens — superscalar does it in hardware every run, VLIW does it in the compiler once, and vector machines sidestep the question by encoding the parallelism explicitly in the instruction.
