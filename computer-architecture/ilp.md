---
title: Doing More, Faster - The World of Instruction-Level Parallelism
description: An exploration of Instruction-Level Parallelism (ILP), including basic concepts, techniques to increase it, and a look at superscalar, super-pipelined, VLIW, array, and vector processor architectures.
date: 2023-05-03
draft: false
slug: /pensieve/coa/instruction-level-parallelism
tags:
  - Computer Architecture
  - CS Basics
---

Hey performance seekers! We've seen how pipelining acts like an assembly line to improve instruction throughput. But modern processors push the boundaries even further by exploiting **Instruction-Level Parallelism (ILP)**. This refers to the ability to execute different instructions from the *same program thread* simultaneously, squeezing even more performance out of each clock cycle. Let's dive into how this is achieved.

## Basic Concepts of Instruction-Level Parallelism

ILP is about finding and exploiting opportunities for parallel execution among instructions. While pipelining overlaps different *stages* of sequential instructions, advanced ILP techniques aim to overlap the execution of *entire instructions* or their independent parts.

The main limiting factors to ILP are **dependencies** between instructions:
* **Data Dependencies (RAW - Read After Write or True Dependencies):** An instruction needs the result of a previous instruction. These are fundamental and must be respected to ensure correct execution. For example, `ADD R1, R2, R3` followed by `SUB R4, R1, R5` has a true dependency on `R1`.
* **Name Dependencies (False Dependencies):** These occur when instructions use the same register or memory location name, but there's no actual flow of data between them.
    * **WAR (Write After Read or Anti-Dependency):** An instruction writes to a location that a *previous* instruction reads. The write must not occur before the earlier read.
    * **WAW (Write After Write or Output Dependency):** Two instructions write to the same location. The writes must occur in the original program order to ensure the final correct value.
    Name dependencies can often be resolved.
* **Control Dependencies:** The execution of an instruction depends on the outcome of a previous branch instruction. This dictates the flow of instructions and what can be executed in parallel.

The **latency** of an instruction (the time it takes from start to finish) is another factor. ILP techniques try to overlap these latencies.

## Techniques for Increasing ILP

Both hardware and software (compiler) techniques are employed to increase the amount of ILP that can be extracted:

### Hardware Techniques:
1.  **Pipelining:** The foundational technique for ILP, allowing different stages of multiple instructions to execute concurrently.
2.  **Multiple Issue:** Processors that can fetch, decode, and issue (begin execution of) more than one instruction per clock cycle.
3.  **Dynamic Scheduling:** Hardware rearranges the order of instruction execution at runtime (out-of-order execution) to reduce stalls caused by dependencies. When an instruction is stalled, other independent instructions can proceed.
    * **Scoreboarding:** A centralized hardware mechanism that tracks instruction dependencies and resource availability to decide when instructions can execute.
    * **Tomasulo's Algorithm:** A distributed approach using **reservation stations** (buffers at functional units) and a **Common Data Bus (CDB)** for broadcasting results. It inherently performs **register renaming** by using tags that point to reservation stations producing results, which helps eliminate WAR and WAW hazards.
4.  **Branch Prediction:** Hardware predicts the outcome of branches to keep fetching and speculatively executing instructions along the predicted path, minimizing stalls from control dependencies.
5.  **Speculative Execution:** Executing instructions before it's certain they are on the correct execution path (e.g., instructions after a predicted branch or a predicted memory access). Results are usually held in a temporary buffer and only committed to the architectural state (registers/memory) once the prediction is confirmed correct. If incorrect, results are flushed.
6.  **Hardware Register Renaming:** The processor has more physical registers than architectural registers visible to the programmer. During execution, architectural registers in instructions are mapped to physical registers. This helps eliminate WAR and WAW hazards by providing unique physical storage for results of different instructions that might target the same architectural register.

### Software (Compiler) Techniques:
1.  **Loop Unrolling:** The compiler replicates the body of a loop multiple times within a single iteration of the modified loop. This reduces the overhead of loop control instructions (branches, counter updates) and increases the number of instructions in the loop body, providing more opportunities for the hardware to find ILP.
2.  **Software Pipelining (Symbolic Loop Unrolling):** The compiler restructures a loop such that each iteration of the *new* loop body consists of instructions from *different* iterations of the *original* loop. This aims to keep all stages of the hardware pipeline busy by overlapping operations from different original iterations. It typically involves a prologue to fill the pipeline, a steady-state kernel, and an epilogue to drain it.
3.  **Instruction Scheduling:** The compiler reorders instructions within a program (respecting dependencies) to minimize stalls and maximize the parallel execution capability of the hardware. This can involve moving independent instructions to fill delay slots or to separate dependent instructions.
4.  **Compiler-Based Register Renaming:** Similar to hardware register renaming, the compiler can use different registers for otherwise conflicting variable assignments to break false dependencies.

**(Question:** Why is register renaming particularly effective at increasing ILP?)
*Answer: It eliminates WAR and WAW hazards, which are "false" dependencies that arise from the limited number of architectural registers, not from the actual flow of data. Removing them gives more freedom to reorder instructions for parallel execution.*

## Superscalar Processor Architectures

A **superscalar** processor can issue and execute more than one instruction per clock cycle from a single program thread. It exploits ILP by having multiple independent functional units (e.g., multiple integer ALUs, floating-point units, load/store units) and the logic to dispatch instructions to them simultaneously.

* **Key Features:**
    * Multiple instruction pipelines.
    * Ability to fetch, decode, and issue several instructions at once.
    * Often employ dynamic scheduling to manage dependencies and resource allocation among the parallel instructions.
    * Heavy reliance on branch prediction and speculative execution to keep the multiple pipelines fed.
* **Operation:** The processor fetches a block of instructions, determines dependencies between them, and dispatches those that are ready and for which functional units are available. Instructions may complete out of program order, but mechanisms (like a reorder buffer) are often used to ensure they are *committed* (results written to architectural state) in program order to maintain precise exceptions.
* **Challenges:** The hardware complexity for dependency checking among multiple instructions, resource allocation, and managing out-of-order execution with precise exceptions is significant.

## Super-pipelined Processor Architectures

**Super-pipelining** achieves ILP by making the pipeline itself "deeper" – that is, breaking down the existing pipeline stages into more, smaller sub-stages.
* By having more, shorter stages, the clock cycle time can be reduced, allowing the processor to run at a higher clock frequency.
* If an instruction takes $k$ original-length cycles and each stage is divided into $m$ sub-stages, the new pipeline depth is $mk$ and the clock cycle can be $1/m$ of the original (ideally).
* This approach still completes one instruction per (faster) clock cycle in the ideal case, but because the clock is faster, the overall instruction throughput increases.
* It's a form of temporal parallelism, pushing instructions through more, quicker stages.
* Super-pipelining can be combined with superscalar designs: a processor might have multiple, deep pipelines.

## VLIW (Very Long Instruction Word) Processor Architectures

**VLIW** architectures take a different approach to ILP, relying heavily on the compiler.
* **Concept:** The compiler analyzes the program, identifies multiple primitive operations that can be executed in parallel, and packs them into a single, very long instruction word.
* Each part of this VLIW instruction explicitly targets a specific, independent functional unit (e.g., one operation for an integer ALU, one for a floating-point unit, one for a memory access unit, all within the same VLIW).
* **Static Scheduling:** All decisions about which operations to execute in parallel are made by the compiler at compile time. The hardware is simpler because it doesn't need complex dynamic scheduling logic to detect dependencies or issue instructions. It simply dispatches the pre-scheduled operations from the VLIW to their respective functional units.
* **Characteristics:**
    * Multiple functional units.
    * The "instruction" fetched from memory is very wide, containing fields for each functional unit.
    * Compiler is responsible for detecting hazards and ensuring that operations packed into one VLIW are independent or that their dependencies are correctly handled by the schedule across multiple VLIWs.
* **Challenges:**
    * **Compiler Complexity:** The compiler bears a heavy burden.
    * **Code Density:** If the compiler cannot find enough independent operations to fill all slots in a VLIW, it must insert NOPs (No-Operations), potentially leading to larger code size.
    * **Lock-Step Execution (in early VLIWs):** If one operation in a VLIW stalls (e.g., due to a cache miss), all other operations in that VLIW might also have to stall, as they are issued together.
    * **Binary Compatibility:** A VLIW program compiled for a machine with a certain number and type of functional units may not run (or run efficiently) on a VLIW machine with a different configuration.

**(Think about it:** What's the main philosophical difference between superscalar and VLIW approaches regarding who finds the parallelism?)
*Answer: Superscalar relies primarily on hardware at runtime to find and manage parallelism. VLIW relies primarily on the compiler at compile time.*

## Array and Vector Processors: Specialized ILP for Data Parallelism

While superscalar and VLIW focus on ILP within a general instruction stream, **array** and **vector processors** are specialized architectures that excel at exploiting **data parallelism**. This is a form of ILP where the *same operation* is performed concurrently on many different data elements. This falls under the **SIMD (Single Instruction, Multiple Data)** category of Flynn's Classification.

### Array Processors
* Consist of a large number of identical **Processing Elements (PEs)**, each typically having its own ALU and local memory.
* A single **Control Unit (CU)** fetches and decodes instructions.
* Scalar or control instructions are executed by the CU.
* **Vector instructions** (or instructions intended for parallel execution) are broadcast by the CU to all (or a selected subset of) PEs. Each PE then executes the same instruction on the data element(s) stored in its own local memory, all in lock-step synchronization.
* An interconnection network is needed for data routing and communication between PEs if required by the computation.

### Vector Processors (Often Pipelined)
* These processors have instructions that operate on entire **vectors** (one-dimensional arrays of data) rather than single data elements. For example, a single `VADD V1, V2, V3` instruction would add corresponding elements of vector registers V2 and V3 and store the results in vector register V1.
* They typically feature:
    * **Vector Registers:** Capable of holding multiple data elements (e.g., 64 or 128 elements).
    * **Pipelined Functional Units:** Highly pipelined ALUs, multipliers, etc., designed to stream vector elements through them. Once the pipeline is filled, a result can be produced nearly every clock cycle for long vectors.
    * Specialized instructions for loading/storing vectors, handling strides in memory access, and performing operations like scatter/gather.
* The compiler plays a key role in **vectorization**, which involves transforming scalar loops in high-level language code into vector instructions.

Array and vector processors are highly efficient for scientific computing, image processing, and other domains with regular, large-scale data parallelism.

## Conclusion: The Quest for More Concurrent Operations

Instruction-Level Parallelism is a fundamental pillar of modern high-performance computing. From the foundational concept of pipelining to the aggressive multi-issue capabilities of superscalar processors, the clever compile-time scheduling of VLIW machines, and the data-parallel prowess of array/vector architectures, the goal remains the same: to keep the processor's functional units as busy as possible, executing more useful work in each clock cycle. Each approach comes with its own set of complexities and trade-offs, pushing the boundaries of both hardware design and compiler technology.

**Key Takeaways:**

* ILP aims to execute multiple instructions from a single thread concurrently.
* Dependencies (data, name, control) are the primary limiters of ILP.
* Techniques like pipelining, multiple issue, dynamic scheduling (Scoreboarding, Tomasulo's), branch prediction, and register renaming are used in hardware to boost ILP.
* Compilers contribute through loop unrolling, software pipelining, and instruction scheduling.
* **Superscalar** processors dynamically issue multiple instructions to multiple functional units.
* **Super-pipelined** processors use deeper pipelines and faster clocks.
* **VLIW** processors rely on the compiler to statically schedule multiple operations in long instruction words.
* **Array and Vector processors** exploit data parallelism using SIMD principles.
