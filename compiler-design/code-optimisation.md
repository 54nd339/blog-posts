---
title: Supercharging Your Code - A Tour of Compiler Optimizations!
description: Discover how compilers make your programs faster and smaller! Explore local and loop optimizations, constant folding, DAGs, and the art of frequency reduction.
date: 2023-05-16
draft: false
slug: /pensieve/compiler-design/code-optimsation
tags:
  - Compiler Design
  - CS Basics
---

Hey Code Speedsters! We've journeyed through the compiler's frontend, watching our source code transform from text into tokens (Lexical Analysis), then into structured parse trees (Syntax Analysis), and finally get its meaning checked and translated into Intermediate Representation (Semantic Analysis & IR Generation). Now, with a functionally correct but perhaps clunky intermediate version of our program, it's time for a pit stop at the **Code Optimization** phase!

This is where the compiler rolls up its sleeves and works like a master mechanic, tuning and refining the intermediate code. The goal? To make the final program more efficient – run faster, take up less space, or even consume less power – all *without changing what it actually does*.

Think of it like an expert editor taking a well-structured draft and making it more concise, impactful, and elegant. Let's explore how compilers achieve this performance boost!

---
## Why Bother? Considerations for Optimization
Before diving into *how* code is optimized, let's consider *why* and what the trade-offs are:

* **The Primary Goal**:
    * **Improve Performance**: Make the program execute faster. This is often the main objective.
    * **Reduce Code Size**: Smaller executables load faster and use less storage.
    * **Minimize Power Consumption**: Increasingly important for mobile and embedded devices.

* **The Golden Rule: Correctness is King!**
    An optimization technique is only useful if it preserves the meaning of the original program. A faster program that gives the wrong answer is worse than a slower, correct one.

* **The Trade-offs**:
    * **Compile-Time Cost**: More aggressive and complex optimizations can significantly increase how long it takes to compile the program. Developers need to balance this with the desired runtime improvements.
    * **Is it Worth It?**: Some optimizations offer huge gains, while others might provide marginal benefits at a high cost. Compilers often use heuristics to decide which optimizations to apply and how extensively. The Pareto Principle (80/20 rule) often applies: a small portion of the code (e.g., inner loops) might account for most of the execution time.
    * **Debugging**: Optimized code can sometimes be harder to debug because the structure of the generated code might not directly map back to the source code lines.
    * **Target Architecture**: Some optimizations are generic (machine-independent), while others are tailored to the specifics of the target processor (machine-dependent).

---
## Where to Optimize? The Scope of Optimization
Optimizations can be applied at different levels or "scopes" within the program:

1.  **Local Optimization**:
    * These transformations are applied to small segments of code, typically within a single **basic block**.
    * A **basic block** is a sequence of consecutive instructions where control flow enters at the beginning and exits at the end, with no branching in or out, except possibly at the very end.
    * Local optimizations are generally simpler to implement and computationally less expensive.

2.  **Global (Intraprocedural) Optimization**:
    * These optimizations operate over a larger scope, typically an entire function or procedure.
    * They consider how information (data flow) moves across different basic blocks within that function.
    * This often requires more complex analysis, like **data-flow analysis**. Loop optimizations are a prime example of global optimizations.

3.  **Interprocedural Optimization (Whole Program Optimization)**:
    * These optimizations look beyond individual function boundaries and analyze the interactions between different functions.
    * Examples include inlining (replacing a function call with the body of the function), dead code elimination across modules, etc.
    * These can be very powerful but are also the most complex and can significantly increase compile time.

For this post, we'll primarily focus on common local and global (especially loop) optimizations.

---
## Sharpening the Edges: Local Optimization Techniques
Local optimizations are about making individual basic blocks as efficient as possible.

### DAG Representation of Basic Blocks
A **Directed Acyclic Graph (DAG)** is a powerful way to represent the computations within a basic block:
* **Nodes**: Represent operators (e.g., `+`, `*`, array access). Interior nodes are operators; leaf nodes are variables or constants.
* **Edges**: Show the flow of values from operands to operators.

**How DAGs help:**
1.  **Identifying Common Subexpressions**: If the same computation appears multiple times within a block, the DAG will have a single node representing it, with multiple parents pointing to it. This allows the compiler to compute the value once and reuse it.
    * Example:
        ```
        x = a + b;
        y = a + b;
        z = x * 2;
        ```
        A DAG would show `a + b` computed once. Optimized code:
        ```
        t1 = a + b;
        x = t1;
        y = t1;
        z = x * 2; // or z = t1 * 2;
        ```
2.  **Reordering Instructions**: The DAG can reveal opportunities to reorder instructions for better target code, perhaps to improve register usage or instruction scheduling.

### Peephole Optimization
This technique involves examining a small, sliding "window" of instructions (the peephole) in the generated code (intermediate or target) and replacing instruction sequences within the peephole with shorter or faster ones.
* **Redundant Instruction Elimination**: `MOV R0, x` followed by `MOV x, R0` might be removed if `R0` is not used in between.
* **Constant Folding**: If an instruction computes a constant, like `ADD R1, R1, #0`, it can be eliminated.
* **Strength Reduction**: Replacing an expensive operation with an equivalent cheaper one. E.g., `MUL R1, R1, #2` can be replaced by `LSL R1, R1, #1` (logical shift left by 1).

---
## The Powerhouse: Loop Optimizations
Loops are critical hotspots in many programs. Even small improvements within a loop can lead to significant overall speedups because the loop body executes many times.

### Frequency Reduction (Code Motion / Loop-Invariant Code Motion)
This is one of the most effective loop optimizations.
* **Concept**: Identify computations inside a loop whose operands do not change their values during the loop's execution (these are called **loop-invariant computations**).
* **Action**: Move these computations to be performed only once, *before* the loop begins.
* **Example**:
    ```java
    // Original code
    for (int i = 0; i < n; i++) {
        int limit = width * height; // Loop-invariant
        array[i] = i + limit;
    }

    // Optimized code (Frequency Reduction)
    int limit = width * height; // Moved out!
    for (int i = 0; i < n; i++) {
        array[i] = i + limit;
    }
    ```
    Here, `width * height` is computed `n` times in the original, but only once in the optimized version.

### Other Important Loop Techniques (Briefly)
* **Induction Variable Elimination**: Simplifies or eliminates variables that are updated by a constant amount in each iteration (e.g., loop counters).
* **Strength Reduction**: Within loops, replacing expensive operations with cheaper ones. For example, if an array index is `i * 4` inside a loop, and `i` increments by 1, `i * 4` can be updated by adding 4 in each iteration instead of re-multiplying.
* **Loop Unrolling**: Reduces loop control overhead by duplicating the loop body and decreasing the number of iterations. Trade-off: increases code size.

---
## Smart Calculations: Folding and Propagation
These techniques simplify expressions and data flow.

### Constant Folding
* **Concept**: If an expression involves only constant values, evaluate it at compile time instead of generating code to compute it at runtime.
* **Example**:
    ```java
    int radius = 10;
    double circumference = 2 * 3.14159 * radius; // 3.14159 * radius is not constant folding
                                                // if radius is not const itself.
    int twoPlusThree = 2 + 3;                   // This IS constant folding
    ```
    The compiler can replace `2 + 3` directly with `5`. If `PI = 3.14159; AREA_CALC = PI * SOME_CONST;`, then `PI * SOME_CONST` can be folded.

### Constant Propagation
* **Concept**: If a variable is assigned a constant value, subsequent uses of that variable (where its value hasn't changed) can be replaced by the constant itself.
* **Example**:
    ```java
    final int MAX_USERS = 100;
    int currentUsers = MAX_USERS;
    if (userCount < currentUsers) { /* ... */ }
    ```
    The compiler might propagate `100` for `currentUsers`, transforming the condition to `if (userCount < 100)`. This can then enable further constant folding if `userCount` is also known.

### Copy Propagation
* **Concept**: After an assignment like `y = x;`, subsequent uses of `y` can be replaced by `x`, as long as neither `x` nor `y` has been reassigned.
* **Example**:
    ```java
    int a = b;
    int c = a + 5;
    ```
    Could become:
    ```java
    int a = b;
    int c = b + 5; // 'a' propagated to 'b'
    ```
    This can sometimes eliminate the variable `a` altogether if it's not used elsewhere (dead code elimination).

---
## Polishing for Performance!

Code optimization is a fascinating and complex area of compiler design, transforming well-behaved intermediate code into a lean, mean, executing machine. By carefully considering correctness and applying a variety of techniques—from local tweaks using DAGs and peephole optimization to powerful global strategies like loop optimizations (frequency reduction being a star player) and intelligent simplification through folding and propagation—compilers significantly enhance the efficiency of the programs we write every day.

While we've only scratched the surface, it's clear that optimization is not just an afterthought but an integral step in producing high-quality software.
