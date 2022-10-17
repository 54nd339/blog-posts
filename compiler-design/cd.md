---
title: Compilers - The Phases That Turn Source Into an Executable
description: What a compiler is, how it differs from an interpreter, and the assembly line of phases - lexical, syntax, semantic, intermediate code, optimization, code generation - that a single statement passes through on its way from text to machine code, with the symbol table threaded through all of them.
date: 2022-10-17
draft: false
slug: /compiler-design/cd
tags:
  - Compilers
  - Parsing
  - Code Generation
---

You write `position = initial + rate * 60` and the machine runs a sequence of register loads, a multiply, an add, and a store. Nothing about the source text says *which* registers, *what* order, or that `60` needs converting to a float first. A **compiler** is the program that works all of that out — it translates code in a **source language** (C, Rust, Swift) into a **target language**, usually the machine code or assembly for one processor.

This post is the map: the phases a compiler runs, what each one produces, and how one statement changes shape as it moves down the line. Each phase has its own post; this is the overview that connects them.

## Compiler versus interpreter

Both take source code and run it; they differ in *when* the translation happens.

- A **compiler** translates the whole program to machine code (or an intermediate form) once, ahead of time. Execution is then fast because the CPU runs native instructions directly. The cost is a build step, and errors surface only after the whole program is analysed.
- An **interpreter** walks the program and executes it construct by construct, translating as it goes. Startup is immediate and errors point at the line that failed, but every execution re-does the translation work, so it runs slower.

Many modern systems blend the two: a compiler emits bytecode, and a virtual machine interprets it or JIT-compiles hot paths at runtime.

A compiler rarely works alone. The surrounding **language processing system** includes a **preprocessor** (expands `#include` and macros), an **assembler** (turns emitted assembly into machine code), and a **linker** and **loader** (stitch compiled modules and libraries together and place the result in memory).

## Why high-level languages need them

Three generations of language explain the job. **Machine language** — raw numeric opcodes like `C706 0000 0002` for "store 2 at address 0" — is what the CPU executes and what early programmers wrote by hand. **Assembly language** replaces the numbers with mnemonics (`MOV x, 2`) but stays tied to one instruction set. **High-level languages** (FORTRAN onward) use algebraic, English-like syntax and are deliberately **machine-independent**: the same C program compiles for many architectures. The compiler is what bridges that abstraction gap, and it is why the gap is affordable.

## The phases

Compilation is an assembly line. Each **phase** consumes the previous phase's output, does one transformation, and passes it on.

```text
source text
  → lexical analysis      → token stream
  → syntax analysis       → syntax tree
  → semantic analysis     → annotated syntax tree
  → intermediate code gen → intermediate representation (e.g. three-address code)
  → machine-independent optimization → better IR
  → code generation       → target assembly / machine code
  → machine-dependent optimization   → optimized target code
```

Threaded through every phase is the **symbol table** — a data structure holding each identifier's name, type, scope, and eventual memory location. The lexer creates entries, the semantic analyser fills in types, the code generator reads offsets from it. Its own post covers [how it is built](/citadel/compiler-design/symbol-tables).

Trace `position = initial + rate * 60`, with `position`, `initial`, `rate` declared as floats.

### Lexical analysis

Group the character stream into **lexemes** and emit a **token** for each, discarding whitespace and comments. The identifiers go into the symbol table.

```text
<id,1> <=> <id,2> <+> <id,3> <*> <number,60>
```

Tokens are the "words" of the language; [lexical analysis](/citadel/compiler-design/lexical-analysis) specifies them with regular expressions and recognises them with finite automata.

### Syntax analysis

Check the token sequence against the language's grammar and build a **syntax tree** that captures its structure — here, that `rate * 60` is a subexpression, added to `initial`, and the sum assigned to `position`.

```text
        =
       / \
   <id,1> +
         / \
     <id,2> *
           / \
       <id,3> <number,60>
```

[Syntax analysis](/citadel/compiler-design/syntax-analysis) covers the context-free grammars and the top-down and bottom-up parsers that do this.

### Semantic analysis

Check what grammar cannot: are the types compatible, are names declared before use, is the assignment legal. Because `rate` is a float and `60` an integer, the analyser inserts a coercion node.

```text
        =
       / \
   <id,1> +
         / \
     <id,2> *
           / \
       <id,3> inttofloat
                 |
             <number,60>
```

[Semantic analysis](/citadel/compiler-design/semantic-analysis) covers type checking and syntax-directed translation.

### Intermediate code generation

Emit a low-level, machine-independent form. A common choice is **three-address code**, where each instruction has one operator and at most three operands:

```text
t1 = inttofloat(60)
t2 = id3 * t1
t3 = id2 + t2
id1 = t3
```

### Optimization

**Machine-independent** optimization improves the IR without knowing the target CPU — fold constants, eliminate common subexpressions, drop dead code, hoist loop-invariant computations. Here `t1` is a compile-time constant, so `inttofloat(60)` collapses to `60.0` and one instruction disappears. [Code optimization](/citadel/compiler-design/code-optimisation) covers the local and loop transformations; [data flow analysis](/citadel/compiler-design/dataflow-analysis) covers the program-wide analysis they depend on.

### Code generation

Translate the IR to target instructions, choosing machine instructions for each IR operation and assigning variables to registers:

```asm
LDF  R2, id3
MULF R2, R2, #60.0
LDF  R1, id2
ADDF R1, R1, R2
STF  id1, R1
```

**Machine-dependent** optimization then tunes this for the specific architecture — special instructions, scheduling for the pipeline, a final peephole pass. [Code generation](/citadel/compiler-design/code-genration) covers instruction selection and register allocation.

## Front end and back end

The phases group into two halves:

- **Analysis (front end):** lexical, syntax, semantic, IR generation. It breaks the source down, checks it, and produces the IR. Largely source-language-dependent and target-independent.
- **Synthesis (back end):** the optimizers and code generator. It builds the target program from the IR and symbol table. Largely target-dependent.

The IR is the seam. Because it is neutral, you can pair one front end with several back ends to target different machines, or several front ends with one back end to compile several languages for the same machine — which is exactly how toolchains like LLVM are organised.

## The one idea to keep

A compiler is not one clever translation but a pipeline of narrow ones, each turning a slightly lower-level representation into a slightly lower one still — characters to tokens to tree to IR to instructions. The symbol table is the shared memory that lets a late phase use a fact an early phase discovered, and the IR seam in the middle is what lets front ends and back ends be built and reused independently.
