---
title: The Final Frontier - Generating Executable Object Code!
description: Explore the last phase of compilation - Object Code Generation! Learn about object code forms, machine-dependent optimization, register allocation, and code generation algorithms.
date: 2023-05-18
draft: false
slug: /pensieve/compiler-design/code-generation
tags:
  - Compiler Design
  - CS Basics
---

Hey Code Finishers! We've navigated the intricate pathways of lexical analysis, syntax structuring, semantic meaning-making, and the clever refinements of code optimization. Our program, now a polished Intermediate Representation (IR), stands at the threshold of its final transformation. It's time for the **Object Code Generation** phase – the grand finale where abstract instructions become concrete commands that a machine can understand and execute!

This is where the compiler translates the optimized IR into the target machine's language, or sometimes into assembly language. It's a critical step, as the quality of the generated code directly impacts the program's speed and efficiency. Think of it as taking a detailed, optimized architectural blueprint (the IR) and translating it into the precise materials and step-by-step construction sequence for a specific building site (the target machine).

Let's dive into how this final translation happens!

---
## What Does the Output Look Like? Object Code Forms

The code generator's output, the **target code**, can take several forms:

1.  **Absolute Machine Code**:
    * This is raw machine language, directly executable by the CPU. All memory addresses for code and data are fixed.
    * **Pros**: Simple to load and run.
    * **Cons**: The starting address of the program must be known at compile time. Difficult to use for programs composed of separately compiled modules.
    * **Use Case**: Small, standalone programs, or embedded systems where memory layout is fixed.

2.  **Relocatable Machine Code**:
    * This is the most common output format. It consists of machine instructions along with data, but with addresses specified relative to some starting point.
    * It also contains a list of places where the code needs to be "patched" by the linker (e.g., calls to external functions or references to data in other modules).
    * A **linker** combines multiple relocatable object files and library files to create a single executable. A **loader** then loads this executable into memory, adjusting relative addresses to absolute ones.
    * **Pros**: Supports separate compilation of program modules, allows for shared libraries.
    * **Cons**: Requires linking and loading steps.

3.  **Assembly Language**:
    * Some compilers first generate assembly language code specific to the target machine. An **assembler** then translates this assembly code into machine code.
    * **Pros**: Simplifies the compiler writing process (as the assembler handles some low-level details), makes the compiler more portable to different machines (just need a new assembler backend), and the assembly output can be human-readable for debugging.
    * **Cons**: Adds an extra step (assembly) to the compilation process.

---
## Tailoring for the Target: Machine-Dependent Code Optimization ️
While many optimizations are performed on the machine-independent IR, the code generation phase itself is inherently machine-dependent and provides opportunities for target-specific enhancements:

* **Instruction Selection**: This is a crucial task. The IR may have generic operations like "add" or "move." The code generator must choose the best (e.g., fastest, smallest, or lowest power consuming) sequence of machine instructions to implement each IR operation. Modern CPUs often have rich instruction sets with specialized instructions or addressing modes (e.g., `LEA` instruction for address calculation, auto-increment/decrement modes) that can perform complex operations efficiently. For example, `a = b + c*4` might be implemented by a single instruction on some architectures if `c` is in a register and `b` is a base address.
* **Instruction Scheduling**: To maximize performance on modern pipelined processors or those with multiple execution units, the order of instructions can matter significantly. The code generator might reorder instructions (where dependencies allow) to avoid pipeline stalls, keep functional units busy, and exploit instruction-level parallelism.
* **Peephole Optimization (on Target Code)**: As we've touched on before, this technique can be applied as a final pass over the generated machine code. It examines a small "window" of instructions and replaces sequences within the peephole with shorter, faster, or more efficient equivalents. This can catch redundancies or inefficiencies introduced by earlier, more general code generation steps. Techniques include:
    * Redundant load/store elimination.
    * Elimination of unreachable code.
    * Flow-of-control optimizations (e.g., simplifying jumps around jumps).
    * Exploiting machine idioms (specific efficient instruction sequences for common tasks).

---
## The Crown Jewels: Register Allocation and Assignment
CPUs have a small number of **registers**, which are extremely fast memory locations directly accessible by the processor. Using these registers effectively to store frequently accessed variables and temporaries is one of the most critical factors in generating fast code.

* **Register Allocation**: This phase decides *which* variables and intermediate results (temporaries) should reside in registers at each point in the program.
* **Register Assignment**: Once allocated, this phase assigns *specific* physical registers to these chosen items.

**Challenges and Techniques**:
* **Limited Registers**: The number of available registers is small (e.g., 8, 16, 32 general-purpose registers). If more values need to be in registers than available, some values must be "spilled" to main memory and "filled" (reloaded) when needed, incurring performance costs.
* **Global vs. Local**: Register allocation can be done locally (within a basic block) or globally (across an entire function). Global allocation is more complex but generally yields better results.
* **Graph Coloring**: A classic approach for global register allocation.
    1.  Construct an **interference graph**: Nodes represent variables/temporaries that are candidates for registers. An edge connects two nodes if their corresponding items are "live" (their values are in use) at the same time, meaning they cannot share the same register.
    2.  The problem then becomes trying to "color" the graph with `k` colors (where `k` is the number of available registers) such that no two adjacent nodes (connected by an edge) have the same color. If a `k`-coloring is not possible, some variables must be spilled.
* **Usage Counts**: Simpler methods might prioritize keeping variables that are frequently used within a basic block or loop in registers.

Efficient register allocation and assignment minimize memory traffic, which is a major bottleneck in program execution.

---
## The Recipe: Generic Code Generation Algorithms

Translating the optimized Intermediate Representation (like Three-Address Code - TAC) into target machine instructions is a systematic process. A common high-level algorithm processes each TAC instruction. For a typical TAC statement like `x = y op z`:

1.  **Operand Location**: Determine where the current values of `y` and `z` are located (register, memory). If they are not in registers suitable for the `op` instruction, generate **load** instructions to bring them into registers. The `getReg()` function is often used here to select an appropriate register, possibly spilling another variable if necessary.
2.  **Instruction Selection**: Choose the target machine instruction(s) that correspond to the `op`. For instance, if `op` is `+`, generate an `ADD` instruction.
3.  **Result Location**: Determine where the result `x` should be stored. If `x` is a temporary value that will be used by subsequent instructions, it's often best to keep it in a register. If `x` is a program variable, it might eventually need to be stored back to its memory location.

**Managing Information: Descriptors**
To make intelligent decisions, the code generator maintains:
* **Register Descriptors**: For each register, this tracks which variable(s) currently hold their value in that register.
* **Address Descriptors**: For each program variable, this tracks the location(s) (a register, a memory address, a stack location) where its current value can be found.

These descriptors are consulted and updated as each TAC instruction is processed and target code is emitted. For example, after generating `MOV R0, y` and `ADD R0, R0, z` (for `x = y+z` where `x` gets R0):
* The register descriptor for R0 would indicate it now holds `x`.
* The address descriptor for `x` would indicate its value is in R0 (and potentially no longer just in memory if it was loaded and modified).

The lecture notes mention a specific algorithm detailed in the Dragon Book (Aho et al.), typically found in section 8.6.2, which elaborates on such a scheme using `getReg` to manage register usage and minimize costs.

---
## DAGs to the Rescue (Again!): DAG for Register Allocation
Directed Acyclic Graphs (DAGs) are not just for identifying common subexpressions in local optimization; they also play a role in efficient register allocation, especially for evaluating expressions within a basic block.

The **Sethi-Ullman algorithm** is a classic example. It labels nodes in an expression tree (which is a form of DAG where common subexpressions are not merged) to determine the minimum number of registers required to evaluate the expression without spills, and helps in generating optimal code for expression evaluation.

* **Labeling Scheme**:
    * Leaf nodes (variables or constants): Label 1 if the operand is a left child (or if it's a general operand that needs a register). Label 0 if it's a right child constant that can be an immediate operand in an instruction.
    * Interior operator nodes `N` with children `N1` (left) and `N2` (right) having labels `l1` and `l2`:
        * If `l1 == l2`, the label of `N` is `l1 + 1`.
        * If `l1 != l2`, the label of `N` is `max(l1, l2)`.
* **Interpretation**: The label of the root node of the expression tree indicates the minimum number of registers needed to evaluate the entire expression tree by first evaluating the subtree that requires more registers, storing its result (if necessary), then evaluating the other subtree, and finally applying the root operator.

This helps in ordering the evaluation of subexpressions to make the best use of available registers and to decide which intermediate values might need to be temporarily stored in memory if registers are scarce.

---
## From Blueprint to Reality!

Object code generation is the critical final act in the compiler's performance. It's where all the preceding analysis, translation, and optimization efforts culminate in a program that the target machine can actually execute. Choosing appropriate object code forms, performing machine-dependent optimizations like astute instruction selection and scheduling, and, most importantly, managing scarce CPU registers with sophisticated allocation and assignment strategies are all vital for producing efficient, high-performance code.

While the details can be highly complex and specific to each target architecture, the fundamental goal remains: to translate the high-level abstractions of our source code into the most effective sequence of low-level operations.
