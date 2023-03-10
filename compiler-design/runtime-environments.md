---
title: Runtime Environments and Register Allocation
description: Before a compiler can emit code it has to decide how memory is laid out at run time - where locals live, how a call passes arguments and returns, how closures capture variables - and then squeeze the program's values into a handful of registers. Activation records and graph-colouring allocation are the two halves.
date: 2023-03-10
draft: false
slug: /compiler-design/runtime-environments
tags:
  - Compilers
  - Code Generation
---

The front end of a compiler turns source into an intermediate representation that talks about *variables* and *function calls* as if memory were infinite and abstract. The back end has to make that real: a running program has a fixed set of registers, a stack that grows and shrinks, a heap, and a calling convention that two separately compiled functions must both obey. The **runtime environment** is the set of decisions about that layout, and **register allocation** is the optimisation that decides which values get to live in registers at all.

Get either wrong and the program crashes in ways that don't point back to any one line of source.

## Storage at run time

A process's address space is carved into regions the compiler and OS agree on:

- **Code** — the machine instructions, read-only.
- **Static / global** — variables whose size and address are known at compile time and which live for the whole program. One fixed slot each.
- **Stack** — grows and shrinks with function calls; holds one **activation record** per active call.
- **Heap** — explicitly (or GC-) managed, for data whose lifetime doesn't match any call's.

The compiler decides, for every variable, which region it belongs in — a decision driven by its **lifetime** (whole program → static; one call → stack; outlives its call → heap) and whether its size is known statically.

## The activation record

Each function call gets a frame on the stack — its **activation record** (or stack frame). A typical layout, from high address to low:

- **Incoming arguments** beyond what fit in registers.
- **Return address** — where to jump back to when this call finishes.
- **Saved frame pointer** of the caller (the **control link**), so the callee can restore it on return.
- **Saved registers** the callee must preserve.
- **Local variables** and compiler temporaries.
- **Outgoing arguments** for calls this function makes.

Two registers navigate it: the **frame pointer** (`rbp`) points at a fixed spot in the current frame so locals are addressed at constant offsets even as the stack pointer moves, and the **stack pointer** (`rsp`) points at the current top. (Optimising compilers often omit the frame pointer and address everything off `rsp`, freeing a register.)

The **calling sequence** splits work between caller and callee:

- Caller: evaluate arguments, place them (registers first, then the outgoing-args area), push the return address (the `call` instruction does this), jump.
- Callee prologue: save the caller's frame pointer, set up its own, allocate space for locals, save any callee-saved registers it will use.
- Callee epilogue: put the return value in the designated register, restore callee-saved registers and the frame pointer, `ret` (pop the return address and jump).

An **ABI** (System V AMD64, AArch64 AAPCS) nails down every detail — which registers pass the first six integer arguments, which are caller- vs callee-saved, how structs are passed, stack alignment — so that code from different compilers, and the OS, all interoperate.

## Parameter passing modes

- **By value** — copy the argument into the callee's frame. The callee's changes don't affect the caller. C, Java (primitives), Go.
- **By reference** — pass the address; the callee reads and writes the caller's variable. C++ references, `&` in C, `out`/`ref` in C#.
- **By value-result (copy-in/copy-out)** — copy in on call, copy back on return. Fortran, Ada `in out`.
- **By name** — re-evaluate the argument expression each time it's used (Algol 60, and the essence of lazy evaluation). Rare and surprising.

Variadic functions (`printf`) need a convention for where the extra arguments sit and how the callee walks them, since their count and types aren't in the signature.

## Nested functions and closures

If a language allows a function defined inside another to *outlive* its enclosing call (return it, store it), the inner function's references to the outer's locals can't just be stack offsets — the stack frame is gone. Options:

- **Access link (static link)** — each frame points to the frame of its lexically enclosing function; an inner function walks links to reach an outer variable. Works while everything's on the stack.
- **Displays** — an array indexed by nesting depth, holding a pointer to the most recent frame at each level. Faster than walking links.
- **Heap-allocated closures** — when the function can escape, the captured variables are moved to a heap record (the **closure environment**) that the function value carries a pointer to. This is what JavaScript, Python, Rust (`move`), and functional languages do. The trade is an allocation and GC pressure for the freedom to return functions.

## Register allocation

The IR uses unlimited **virtual registers**; the machine has ~16 general-purpose ones. Allocation assigns virtual registers to physical registers, and **spills** the rest to stack slots.

### Liveness and interference

A virtual register is **live** at a point if its current value will be used later. Two virtual registers **interfere** if they're ever live at the same time — they can't share a physical register. Build the **interference graph**: a node per virtual register, an edge for each interference (computed from a backward dataflow pass over the IR).

### Graph colouring

Assigning `k` physical registers so that no two interfering registers get the same one is exactly **graph colouring** with `k` colours — NP-hard in general, but Chaitin's heuristic works well:

1. **Simplify** — repeatedly remove any node with fewer than `k` neighbours (it can always be coloured later) and push it on a stack.
2. If only nodes with `≥ k` neighbours remain, pick one to **spill** (using a cost heuristic: prefer to spill something used rarely or in no loop), remove it, continue.
3. **Select** — pop nodes off the stack, giving each a colour different from its already-coloured neighbours. A node marked for spilling that turns out colourable ("optimistic colouring") is kept; otherwise its value is loaded/stored around each use, new virtual registers are introduced for those, and the whole process reruns.

**Coalescing** merges the source and destination of a `mov` into one node when they don't interfere, eliminating the copy — the main source of register-allocation payoff, balanced against not making the graph harder to colour.

### Linear scan

Full graph colouring is too slow for a JIT compiling on the fly. **Linear-scan allocation** sorts live intervals by start point and sweeps once, keeping an active set and evicting the interval that ends latest when it runs out of registers. Much faster, slightly worse code — the right trade for HotSpot's C1 and for V8's baseline tiers. SSA form makes both approaches cleaner because SSA interference graphs are chordal and colourable in polynomial time.

## The one idea to keep

The runtime environment is the contract the back end commits to: each call gets a stack frame with a fixed layout (return address, saved frame pointer, locals, outgoing args), navigated by a frame pointer and a stack pointer, with an ABI pinning every register and offset so separately compiled code fits together. Functions that can escape their caller move their captured variables to a heap closure. Then register allocation models "which values can share a register" as graph colouring — simplify low-degree nodes, spill when stuck, colour on the way back — with a fast linear-scan variant for JITs.
