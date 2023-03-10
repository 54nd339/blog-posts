---
title: Out-of-Order Execution, Branch Prediction, and Speculative Side Channels
description: A modern CPU core does not run instructions in program order. It renames registers to break false dependencies, executes whatever is ready, retires in order for precise exceptions, and guesses branch outcomes to keep the pipeline fed - and that last guess is what Spectre and Meltdown turned into a data leak.
date: 2023-03-10
draft: false
slug: /computer-architecture/branch-prediction-ooo
tags:
  - Computer Architecture
  - CPU
  - Pipelining
---

An in-order [pipelined](/citadel/computer-architecture/pipelining) CPU stalls the moment an instruction needs a result that isn't ready yet — a cache miss, a slow multiply — and every instruction behind it waits too, even ones that could run right now. A load that misses to DRAM costs ~200 cycles; stalling for all of them wastes an enormous amount of work the core could be doing.

Out-of-order execution is the fix: run instructions as their inputs become available, not as they appear in the program, while still *presenting* the illusion of sequential execution to the programmer. Doing that needs register renaming, a reorder buffer, and — because branches would otherwise force the core to wait to know which instructions come next — a branch predictor good enough to be right ~99% of the time. This post is those mechanisms and the security hole speculation opened.

## The dependency problem

Consider:

```asm
mul  r1, r2, r3     ; slow
add  r4, r1, r5     ; needs r1 — must wait (true dependency)
sub  r6, r7, r8     ; independent — could run now
mov  r1, r9         ; reuses r1 — but only a NAME clash (false dependency)
```

The `add` genuinely depends on the `mul` (a **read-after-write**, or true, dependency). The `sub` depends on nothing here and should run immediately. The `mov` "depends" on the `mul` only because both write `r1` — a **write-after-write** naming collision, not a data flow.

## Register renaming

The machine has, say, 16 architectural registers but a much larger pool of **physical registers** (100+). Every instruction that writes a register is given a *fresh* physical register, and a **register alias table** maps architectural names to current physical ones. Now the `mov r1, r9` gets a new physical register for its `r1`, so it no longer collides with the `mul`'s `r1` — the false dependency vanishes. Only true data dependencies remain, and those form the real schedule.

## Tomasulo's algorithm

The classic out-of-order engine (IBM 360/91, 1967; the core idea in every big core since):

- Instructions are decoded, renamed, and dropped into **reservation stations** — buffers in front of the execution units that hold an instruction plus its operand values (or tags for operands not ready yet).
- When an execution unit finishes, it broadcasts its result and tag on the **common data bus**; every reservation station waiting on that tag grabs the value.
- An instruction whose operands are all present **issues** to its execution unit. Ready instructions run regardless of program order.

This is **dataflow** execution: an instruction fires when its data arrives.

## The reorder buffer and precise exceptions

Out-of-order *execution* would break two things the programmer relies on: exceptions that name the exact faulting instruction, and a consistent architectural state. The **reorder buffer** (ROB) restores them.

Every instruction enters the ROB in program order and leaves (**retires** / **commits**) in program order. Results computed out of order sit in the ROB until every earlier instruction has retired; only at retirement does a result become part of the visible architectural state. If an instruction raises an exception, the core waits until it reaches the head of the ROB, then discards it and everything after it — so the exception appears **precise**, as if execution had stopped exactly there. Speculative work that turns out to be wrong is discarded the same way: it never retired, so architecturally it never happened.

## Branch prediction

A conditional branch isn't resolved until it executes, which might be dozens of cycles after fetch. Waiting means a bubble in the pipeline every branch — and roughly one instruction in five is a branch. So the fetch unit **predicts** the outcome and speculatively fetches down the predicted path; if the prediction was right (the common case), there's no penalty; if wrong, the speculative instructions are squashed (they haven't retired) and fetch restarts, costing the full pipeline depth — 15–20 cycles on a big core.

Predictors, in rough order of sophistication:

- **Static** — always-taken, or backward-taken/forward-not-taken (loops go back).
- **Bimodal** — a table of 2-bit saturating counters indexed by branch address; needs two wrong guesses in a row to flip its prediction, so a loop's single exit doesn't poison it.
- **Two-level / gshare** — index the counter table by the branch address XORed with a **global history register** (the taken/not-taken pattern of the last N branches), capturing correlations between branches.
- **TAGE** — several tagged tables indexed by geometrically increasing history lengths; the longest-history table that matches wins. State of the art, and why real branch prediction accuracy exceeds 99%.

Alongside: a **branch target buffer** caches where taken branches go (so the target is known at fetch), and a **return address stack** predicts function returns.

## Speculative side channels

Speculation is architecturally invisible — squashed instructions never retire — but it is *not microarchitecturally* invisible. A speculatively executed load still brings a line into the cache, and that leaves a timing footprint even after the speculation is undone. This is the basis of **Spectre** and **Meltdown** (2018).

- **Spectre v1 (bounds-check bypass).** Train the branch predictor so it speculates "in bounds" on `if (i < array_len) x = array[i];`, then call it with an out-of-bounds `i`. The core speculatively reads out-of-bounds memory and uses the value to index a second array, pulling a specific line into cache. The bounds check eventually fails and the work is squashed — but the attacker then times accesses to the second array to learn which line was fetched, and thereby the secret byte.
- **Spectre v2 (branch target injection).** Poison the branch target buffer from one context so a victim's indirect branch speculatively jumps to an attacker-chosen gadget.
- **Meltdown.** On affected Intel cores, a user-mode load of a kernel address executes speculatively and forwards the value to dependent instructions *before* the privilege check faults — the same cache-timing exfiltration follows.

Mitigations: **retpoline** (replace indirect branches with a construct the predictor can't speculate through), speculation barriers (`lfence`) after sensitive bounds checks, **KPTI** (unmap the kernel from user page tables so Meltdown has nothing to read), and hardware fixes in later silicon. The general cost has been a few percent of performance and a permanent asterisk on "speculation is invisible."

## Performance framing

The metric is **IPC** (instructions per cycle) — a wide out-of-order core sustains 3–5 on good code. It's dragged down by mispredicts (the pipeline-flush penalty), by the **memory wall** (a load that misses all caches stalls dependents for hundreds of cycles, and even a deep ROB can't hide all of it), and by long dependency chains with no independent work to overlap.

## The one idea to keep

A modern core runs instructions in dataflow order — whatever's ready — after register renaming removes the false dependencies, then retires them in program order through a reorder buffer so exceptions stay precise and wrong speculation is simply never committed. Branch prediction (TAGE-class, >99% accurate) keeps the fetch stream ahead of branch resolution. The catch is that squashed speculation still perturbs the cache, and Spectre/Meltdown weaponise that timing residue to read memory the program was never allowed to touch.
