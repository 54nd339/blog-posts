---
title: CPU Pipelining - Overlapping Instructions and the Hazards It Creates
description: Why processors run instructions assembly-line style - the five classic stages and the cycle-time and speedup formulas, then the three hazard types (structural, data, control) and how stalls, forwarding, branch prediction, and delayed branches deal with them, plus precise exceptions.
date: 2022-04-01
draft: false
slug: /computer-architecture/pipelining
tags:
  - Computer Architecture
  - Pipelining
  - CPU
---

Run one instruction all the way through the [datapath](/citadel/computer-architecture/processing-unit) before starting the next, and most of the hardware is idle most of the time — the ALU sits unused while an instruction is being fetched, the fetch logic sits unused while the ALU works. **Pipelining** fixes that the way a factory does: break the work into stages and keep every stage busy on a different instruction. Throughput goes up by roughly the number of stages.

The cost is that nearby instructions often *depend* on each other, and the assembly line doesn't wait. Most of what makes pipelining interesting is the machinery for coping with those dependencies — this post covers the pipeline itself and its performance, the three kinds of **hazard**, and how hardware and compilers handle each.

## The pipeline

The classic breakdown, from MIPS, is five stages:

1. **IF** — instruction fetch (needs the instruction cache and a PC adder).
2. **ID** — decode and read registers (register file; an adder for the branch target).
3. **EX** — the ALU operation, or an effective-address calculation.
4. **MEM** — data cache access, for loads and stores.
5. **WB** — write the result back to the register file.

Between every pair of stages sits a **pipeline register** (interface latch) holding that instruction's partial results and control bits, so each stage works on its own instruction independently.

A **pipeline cycle** $\tau$ is set by the slowest stage plus the latch delay $d$:

$$\tau = \max_m(\text{stage delay}_m) + d$$

and the clock frequency is $1/\tau$. Stage delays of 10, 8, 12, 10, 7 ns with a 1 ns latch give $\tau = 12 + 1 = 13$ ns — the fast stages wait for the slow one every cycle, which is why balanced stages matter.

### What you gain

For a $k$-stage pipeline running $n$ instructions: the first takes $k$ cycles to emerge, then one completes every cycle, so

$$T_k = [k + (n - 1)]\,\tau$$

against $T_1 = nk\tau$ for the unpipelined version. Speedup:

$$S_k = \frac{T_1}{T_k} = \frac{nk}{k + n - 1} \xrightarrow{n \gg k} k$$

Efficiency (fraction of pipeline slots doing work) is $\eta = n / (k + n - 1) \to 1$, and throughput is $W = \eta / \tau \to 1/\tau$ — one instruction per cycle in the limit. A single instruction's *latency* actually gets slightly worse, from latch overhead and clock skew; pipelining trades that for throughput.

The same idea works inside an arithmetic unit: [floating-point addition](/citadel/computer-architecture/arithmetic) splits into compare-exponents, align-mantissas, add, normalise — four stages that let several FP adds be in flight at once.

## Hazards

A hazard is anything that stops a new instruction entering the pipeline every cycle.

### Structural hazards

Two instructions need the same hardware in the same cycle. The classic case: a single unified memory, where an instruction in MEM and another in IF both want it. Fixes: **stall** one (insert a bubble), **duplicate** the resource (separate instruction and data caches, so IF and MEM never collide), or internally pipeline a multi-cycle functional unit so it can accept new work before finishing.

### Data hazards

An instruction needs a value a nearby earlier instruction hasn't produced yet.

```asm
ADD R1, R2, R3   ; R1 written in WB
SUB R4, R1, R5   ; R1 read in ID, a few cycles too early
```

This is a **read-after-write (RAW)**, or true, dependency — a real flow of data that must be respected. There are also **write-after-read (WAR)** and **write-after-write (WAW)** hazards, but those only cause trouble when instructions *complete* out of order, which a simple in-order pipeline doesn't do; [ILP](/citadel/computer-architecture/ilp) deals with them.

Handling RAW hazards, cheapest first:

- **Stall (interlock).** Hardware detects the dependency and freezes the dependent instruction until the value is ready. Bubbles, lost throughput.
- **Forwarding (bypassing).** Route the result straight from the ALU output (end of EX) or the memory output (end of MEM) to the ALU input of the waiting instruction, without waiting for WB. This removes almost all stalls between back-to-back ALU operations. A **load-use** hazard — a load immediately followed by a use of the loaded value — still costs one stall cycle, because the value isn't available until the end of MEM.
- **Compiler scheduling.** Reorder independent instructions between the dependent pair so the gap fills itself; where that's impossible, insert NOPs — a software stall.

### Control hazards

A branch doesn't reveal its outcome and target until late (EX or MEM). Instructions fetched sequentially behind a branch that turns out **taken** are wrong and must be flushed — the **branch penalty**.

Handling them:

- **Stall / flush.** Stop fetching when a branch is decoded until the outcome is known. A fixed penalty every branch.
- **Static prediction.** *Predict not taken* — keep fetching sequentially, flush if wrong. *Predict taken* — needs the target early, flush if the branch falls through.
- **Dynamic prediction.** Hardware learns per-branch behaviour:
  - A **1-bit predictor** (branch history table) stores "taken last time?" and predicts the same. It mispredicts twice for every loop (the first and last iteration) and scores 0% on an alternating T-NT-T-NT pattern.
  - A **2-bit saturating counter** is a four-state machine (strongly/weakly not-taken, weakly/strongly taken); it takes two wrong predictions in a row to flip direction. That hysteresis fixes the loop case.
  - A **branch target buffer (BTB)** caches the target address of recently taken branches, so a predicted-taken branch can redirect fetch immediately.
- **Delayed branch.** The instruction(s) in the **delay slot(s)** right after the branch always execute, taken or not. The compiler fills the slot with something safe: an independent instruction from before the branch, or from the target, or from the fall-through path — a NOP only as a last resort. One delay slot hides one cycle of penalty.

## Precise exceptions

An **exception** — an I/O interrupt, arithmetic overflow, an illegal opcode, a page fault, a trap — disrupts execution, and in a pipeline several instructions are mid-flight when one occurs. A later instruction might fault before an earlier one finishes, and instructions behind the faulting one may already have started.

The goal is **precise exceptions**: every instruction before the faulting one has completed, no instruction after it has changed processor state, the faulting instruction is identified, and the saved PC points at it (or the next instruction to run). On detection, the pipeline stops fetching, flushes the instructions behind the fault, lets the ones ahead drain, saves state, and jumps to the handler. Deep or out-of-order pipelines need extra hardware — reorder buffers, history buffers — to undo speculative state and restore a precise point.

## Beyond one pipeline

Several techniques push further, all covered in [instruction-level parallelism](/citadel/computer-architecture/ilp):

- **Superscalar** — several pipelines side by side, issuing multiple instructions per cycle.
- **Superpipelining** — split stages finer for a faster clock.
- **Dynamic scheduling** — hardware reorders execution at runtime (scoreboarding, Tomasulo's algorithm) to work around stalls, with register renaming to erase WAR and WAW hazards.

Compilers help too, with loop unrolling and software pipelining to expose more independent work.

## The one idea to keep

Pipelining buys throughput by overlapping stages, and then spends most of its complexity buying that throughput *back* from the dependencies between instructions. Forwarding handles the common data hazard for free; branch prediction handles control hazards well enough that a good predictor is worth a lot of silicon; and precise exceptions are the constraint that keeps all this speculation from corrupting the programmer's model of a strictly sequential machine.
