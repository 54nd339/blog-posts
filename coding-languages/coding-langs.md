---
title: Compiled, Interpreted, and Bytecode Languages - How Code Runs
description: Source code is for people; a CPU runs machine code. The translation happens ahead of time (C, Go, Rust), at runtime line by line (Python, Ruby), or in two stages through portable bytecode and a JIT (Java, C#). What each path costs in speed, portability, and how early bugs surface.
date: 2020-10-31
draft: false
slug: /coding-languages/coding-langs
tags:
  - Programming Languages
  - Compilers
  - Runtimes
---

A CPU executes **machine code** — binary instructions for one specific processor architecture. The code you write is text designed for people. Something has to bridge the two, and the three common ways of doing it are what people mean by "compiled", "interpreted", and "bytecode" languages. The choice affects how fast the program runs, whether the same build works on another machine, and how early a mistake shows up.

The categories are about *implementation*, not the language itself. CPython interprets Python; a different implementation could compile it. But each language has a dominant implementation, and that is what the label refers to.

## Compiled: translate everything first

A **compiler** translates the whole program to machine code before anyone runs it.

1. You write source — say, C++.
2. A compiler for the target platform (a given OS and CPU) reads it, checks it for errors, and applies optimisations.
3. If it succeeds, it emits an **executable file** of native machine code.
4. To run the program, the OS loads that file into memory and the CPU executes it directly.

**What you get:**

- **Speed.** Translation and optimisation happen once, ahead of time. At runtime there is no translation overhead — the CPU runs native instructions.
- **Early error detection.** Syntax errors, and in statically typed languages like C++, Go, or Rust, type errors, are caught at compile time — before the program reaches a user.

**What it costs:**

- **Platform lock.** The executable targets one OS and CPU architecture. Running it elsewhere means recompiling with a compiler for that platform.
- **A build step.** Compilation takes time, and for a large project that slows the edit–run–debug loop.

Examples: C, C++, Go, Rust, Swift.

## Interpreted: translate as you go

An **interpreter** reads the source and executes it at runtime, without producing a separate machine-code file.

1. You write source — say, Python or JavaScript.
2. When the program runs, the interpreter reads it, typically statement by statement.
3. It performs each statement's operations immediately, then moves to the next.

**What you get:**

- **Portability.** The same script runs anywhere there is a compatible interpreter — Windows, macOS, Linux. Browsers ship a JavaScript engine, so JavaScript runs on all of them.
- **A fast loop.** Change the code, run it — no compile step in between.
- **Dynamic typing (usually).** Types are checked at runtime, which is flexible during development.

**What it costs:**

- **Speed.** Reading, translating, and executing all happen at runtime, every run. That overhead adds up.
- **Late error detection.** A type error or a syntax error in a branch that has not run yet surfaces only when the interpreter reaches that line.

Examples: Python, JavaScript, Ruby, PHP, Perl.

## Bytecode: two stages, and a virtual machine

Java and C# split the difference to get both portability and decent speed.

1. You write source — a `.java` file.
2. A compiler (`javac`) translates it not to machine code but to **bytecode** — a compact, platform-independent instruction set — in a `.class` file.
3. A **virtual machine** executes the bytecode: the Java Virtual Machine (JVM) for Java, the Common Language Runtime (CLR) for C#. The VM is a program built for each OS and CPU.
4. The VM reads the bytecode, and translates it to native machine code the CPU can run.

**Just-in-time (JIT) compilation** is how the VM gets its speed. Rather than interpret every instruction forever, the VM watches which parts of the bytecode run often — the "hot spots" — and compiles those to native machine code on the fly. Subsequent calls to that code run at native speed. Modern JavaScript engines like V8 do the same thing, which is why the compiled/interpreted line is blurry in practice.

**What you get:**

- **"Write once, run anywhere."** The bytecode is portable; only the VM is platform-specific.
- **Good performance.** After an initial warm-up, JIT-compiled hot paths approach native speed, which matters most for long-running programs.
- **A managed runtime.** The VM provides [garbage collection](/citadel/interview/garbage-collection) and a security sandbox.

Examples: Java, C#, Scala, Kotlin.

## Side by side

![Three execution paths: compiled languages go source to compiler to machine code to OS to hardware; bytecode languages go source to compiler to bytecode to a virtual machine with a JIT and interpreter; interpreted languages go source straight to an interpreter at runtime.](../images/compile.png "The compiled, bytecode, and interpreted paths from source code to running program, split across development time and runtime.")

| | Compiled (C++, Go) | Interpreted (Python, JS) | Bytecode (Java, C#) |
| --- | --- | --- | --- |
| Translation happens | before runtime (AOT) | during runtime | to bytecode before, to native during (VM/JIT) |
| First-step output | machine code | none — direct execution | bytecode |
| Execution speed | fastest | slowest | between; near-native with JIT warm-up |
| Platform dependence | high — build per platform | low — interpreter hides it | low — bytecode portable, VM per platform |
| Syntax errors caught | compile time | runtime | compile time (source to bytecode) |
| Edit–run loop | slower (build step) | fastest | build to bytecode, then run |

## The one idea to keep

The question is *when* source becomes machine code: all of it before the program runs (compiled — fastest, least portable, errors caught early), a piece at a time while it runs (interpreted — most portable, fast to iterate, errors caught late), or in two stages through portable bytecode that a virtual machine JIT-compiles as it warms up (bytecode — a deliberate middle). Modern interpreters bundle JIT compilers, so real implementations sit on a spectrum rather than in three boxes.
