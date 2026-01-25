---
title: The Assembly Line of Computation - Understanding CPU Pipelining
description: A detailed exploration of CPU pipelining, from basic concepts and types of pipelines to hazards (structural, data, control), handling techniques, exceptions, and optimization strategies.
date: 2023-05-01
draft: false
slug: /pensieve/coa/pipelining
tags:
  - Computer Architecture
  - CS Basics
---

Hey tech explorers! We've seen how the CPU fetches and executes instructions. But how do modern processors achieve such incredible speeds? One of the key answers is **pipelining**. Think of it like an assembly line in a factory: instead of building one car completely before starting the next, different stages of multiple cars are worked on simultaneously. Let's break down how this "assembly line" works for instructions.

## Basic Concepts of Pipelining

Pipelining is a technique that overlaps the execution of multiple instructions by breaking down the instruction processing into a sequence of independent **stages**. Each stage performs a part of the instruction's execution. As one instruction moves from one stage to the next, the previous stage becomes free to accept the next instruction in the sequence.

The primary goal of pipelining is to increase **instruction throughput** – the number of instructions completed per unit of time – rather than reducing the execution time of a single instruction (which actually slightly increases due to pipeline overhead).

### Pipeline Stages and Cycle Time
A common way to break down instruction execution is into five stages (like in many MIPS processors):
1.  **IF (Instruction Fetch):** Fetch the instruction from memory.
2.  **ID (Instruction Decode & Register Fetch):** Decode the instruction and read operands from the register file.
3.  **EX (Execute/Address Calculation):** Perform the operation in the ALU or calculate an effective memory address.
4.  **MEM (Memory Access):** Access memory for load or store operations.
5.  **WB (Write Back):** Write the result back to the register file.

**Interface latches** or **pipeline registers** are placed between stages to hold the intermediate results and control information as an instruction moves from one stage to the next. These latches are crucial for isolating stages and allowing them to operate somewhat independently on different instructions.

The time required for an instruction to pass through one stage is called a **pipeline cycle** (or processor cycle). In a **synchronous pipeline**, this cycle time ($\tau$) is determined by the slowest stage plus the delay introduced by the pipeline latch ($d$):
$\tau = \text{max}(\text{stage\_delay}_m) + d$
The **pipeline frequency** is then $f = 1 / \tau$.

An **asynchronous pipeline** uses handshaking signals between stages to coordinate transfers, allowing for variable stage delays, but is more complex to design.

### Performance Metrics
* **Ideal Speedup:** For a $k$-stage pipeline processing $n$ tasks (instructions):
    The first task takes $k$ cycles to complete. Each subsequent task completes one cycle after the previous one.
    Total time $T_k = [k + (n-1)] \tau$.
    A non-pipelined processor would take $T_1 = nk\tau$ (assuming each of the $k$ sub-operations takes $\tau$ time).
    So, ideal speedup $S_k = T_1 / T_k = nk / (k + n - 1)$.
    As $n$ becomes much larger than $k$, the speedup approaches $k$.
* **Efficiency ($\eta$):** The percentage of the pipeline's time-space span that is busy.
    $\eta = n / (k + n - 1)$. As $n \rightarrow \infty$, $\eta \rightarrow 1$.
* **Throughput ($W$):** The number of tasks completed per unit time.
    $W = n / ([k + n - 1]\tau) = \eta / \tau$. Ideally, if efficiency is 1, throughput is $1/\tau$, or one instruction per cycle.

Pipelining doesn't reduce the execution time of a single instruction (latency); in fact, it slightly increases it due to overhead from latches and potential clock skew (variations in clock signal arrival times at different latches). However, by processing multiple instructions in parallel stages, it dramatically increases the overall instruction completion rate.

**(Think Point:** If a 5-stage pipeline has stage delays of 10ns, 8ns, 12ns, 10ns, and 7ns, and latch delay is 1ns, what is the pipeline cycle time?)
*Answer: Longest stage (12ns) + latch delay (1ns) = 13ns.*

## Instruction and Arithmetic Pipelines

### Instruction Pipeline
As described, the 5-stage IF-ID-EX-MEM-WB is a classic example of an instruction pipeline.
* **IF stage:** Needs memory access (for instruction cache) and an adder (to update PC).
* **ID stage:** Needs register file access (to read operands) and potentially an adder (for branch target calculation).
* **EX stage:** Needs the ALU.
* **MEM stage:** Needs memory access (for data cache).
* **WB stage:** Needs register file access (to write results).

### Arithmetic Pipeline
Complex arithmetic operations can also be pipelined. For instance, floating-point addition might be broken into stages like:
1.  Compare exponents.
2.  Align mantissas (shift the mantissa of the smaller number).
3.  Add mantissas.
4.  Normalize the result.
Each of these steps can be a pipeline stage, allowing multiple floating-point additions to be in progress simultaneously.

## Pipeline Hazards: When the Flow Breaks

Ideally, a new instruction enters the pipeline every cycle. However, **hazards** are situations that prevent this, forcing the pipeline to stall or take corrective action.

### 1. Structural Hazards
* **Cause:** Two (or more) instructions in the pipeline require the same hardware resource at the same time.
* **Example:** A single, unified memory unit for both instruction fetch (IF stage) and data access (MEM stage). If one instruction is in MEM reading data, another instruction cannot simultaneously be in IF fetching from the same memory unit.
* **Resolution:**
    * **Stall:** One of the conflicting instructions is paused (a "bubble" is inserted into the pipeline).
    * **Duplicate Resources:** Provide separate hardware units. For example, using separate instruction caches and data caches allows IF and MEM stages to access memory concurrently. Similarly, having multiple ALUs can help.
    * **Pipelined Functional Units:** If a functional unit is used for multiple cycles (e.g., a complex multiplier), it can be pipelined internally so it can accept a new operation before the previous one is fully complete.

### 2. Data Hazards
* **Cause:** An instruction depends on the result of a preceding instruction that is still in the pipeline and has not yet written its result back.
* **Types:**
    * **Read After Write (RAW) / True Dependency:** An instruction tries to read an operand before a preceding instruction writes to it.
        *Example:*
        ```assembly
        ADD R1, R2, R3  ; R1 is written in WB stage
        SUB R4, R1, R5  ; R1 is needed in ID stage
        ```
        The SUB instruction would read the old value of R1 if no measures are taken.
    * **Write After Read (WAR) / Anti-Dependency:** An instruction tries to write to a destination before a preceding instruction reads its original value. This is usually not an issue in simple in-order pipelines but can become problematic in out-of-order execution scenarios.
        *Example:*
        ```assembly
        SUB R4, R1, R5  ; Reads R1
        ADD R1, R2, R3  ; Writes R1
        ```
        If ADD somehow completes and writes to R1 before SUB reads it, SUB gets the wrong R1.
    * **Write After Write (WAW) / Output Dependency:** Two instructions write to the same location, and they complete out of order, leaving the wrong value in the destination. Also mainly an issue with out-of-order completion.
        *Example:*
        ```assembly
        ADD R1, R2, R3  ; Writes R1
        SUB R1, R4, R5  ; Also writes R1
        ```
        If ADD (a longer operation) finishes after SUB, the result of SUB in R1 is overwritten.

### Techniques for Handling Data Hazards

1.  **Stalling (Pipeline Interlock):**
    The simplest solution. The hardware detects the hazard and stalls the dependent instruction (and subsequent instructions) in the pipeline until the required data is available. This introduces "bubbles" and reduces throughput. For a load instruction followed immediately by an instruction using its result (a "load-use" hazard), a stall is often unavoidable even with forwarding because data is only available after the MEM stage.

2.  **Operand Forwarding (Bypassing):**
    Instead of waiting for a result to be written back to a register in the WB stage, the result can be routed (forwarded) directly from the output of the functional unit (e.g., ALU output at end of EX, or memory output at end of MEM) to the input of the ALU for a subsequent instruction that needs it in its EX stage.
    * This can eliminate stalls for many RAW hazards between ALU operations.
    * For a load-use hazard (`LOAD R1, addr; ADD R2, R1, R3`), forwarding can pass the loaded value from the MEM/WB pipeline register to the EX stage of the ADD. However, if the ADD needs R1 in its ID stage, and the LOAD produces it in its MEM stage, a one-cycle stall might still be necessary unless the ID stage can also receive forwarded data.

3.  **Compiler Scheduling (Instruction Reordering / NOP Insertion):**
    The compiler can try to mitigate data hazards:
    * **Instruction Reordering:** The compiler can reorder instructions to separate dependent instructions by inserting independent instructions between them.
    * **NOP (No-Operation) Insertion:** If reordering isn't possible, the compiler can insert NOP instructions to create the necessary delay, effectively implementing a software stall.

### 3. Control Hazards (Branch Hazards)
* **Cause:** Occur with branch instructions (and other control flow changing instructions like jumps, calls, returns). The pipeline doesn't know whether to fetch instructions sequentially or from the branch target address until the branch condition is evaluated and the target address is computed, which typically happens late in the pipeline (e.g., EX or MEM stage).
* **Effect:** If the pipeline fetches instructions sequentially after a branch, and the branch is then determined to be "taken" (i.e., execution should jump to the target address), the already fetched sequential instructions are incorrect and must be flushed. This creates "branch penalty" cycles.

### Techniques for Handling Control Hazards

1.  **Stall (Flush Pipeline):**
    The simplest approach. When a branch is decoded, stall the pipeline (stop fetching further instructions) until the branch outcome and target address are known. If instructions were fetched speculative_ly, flush them if the branch is taken. This incurs a fixed penalty.

2.  **Branch Prediction:**
    Try to guess the outcome of the branch ahead of time.
    * **Predict Not Taken:** Always assume the branch will not be taken and continue fetching sequentially. If the branch *is* taken, the incorrectly fetched instructions must be flushed, and the pipeline restarted from the branch target. This works well if most branches are not taken.
    * **Predict Taken:** Always assume the branch will be taken. This requires knowing the branch target address early (ideally in ID stage). If the branch is *not* taken, the speculatively fetched instructions from the target path must be flushed.
    * **Dynamic Branch Prediction:** Hardware learns from the past behavior of branches.
        * **1-bit Predictor (Branch History Table - BHT):** Store one bit per branch, indicating whether it was taken or not taken last time. Predict the same way next time. Can mispredict twice for a loop's first and last iteration. Fails badly for alternating patterns (0% accuracy for T-NT-T-NT...).
        * **2-bit Predictor (Saturating Counter):** Uses two bits per branch to implement a 4-state finite state machine (e.g., Strongly Not Taken, Weakly Not Taken, Weakly Taken, Strongly Taken). A prediction must be wrong twice in a row to change from strongly one way to weakly the other. This adds "hysteresis" and improves prediction accuracy, especially for loops.
        * **Branch Target Buffer (BTB):** A small cache that stores the branch instruction address and its *target address* if it was recently taken. If a fetched instruction address hits in the BTB and is predicted taken, the target address can be fetched immediately.

3.  **Delayed Branch:**
    The instruction(s) immediately following the branch instruction (in the "delay slot(s)") are *always* executed, regardless of whether the branch is taken or not. The compiler's job is to fill this delay slot with useful instructions that are safe to execute in either case.
    * **Filling the slot:**
        * From an independent instruction *before* the branch.
        * From the *target* of the branch (if the instruction is safe to execute even if the branch isn't taken).
        * From the *fall-through* path (if the instruction is safe to execute even if the branch is taken).
        * If no useful instruction can be found, a NOP is inserted.
    A single delay slot can hide one cycle of branch penalty. The effectiveness depends on the compiler's ability to find useful instructions.

## Exception Handling in Pipelined Systems

**Exceptions** are events that disrupt the normal flow of execution (e.g., I/O interrupts, arithmetic overflow, undefined opcode, page faults, software traps). Handling them in a pipelined processor is complex:
* Multiple instructions are in various stages of execution simultaneously.
* An instruction later in the pipeline might cause an exception *before* an earlier instruction has completed.
* Instructions fetched after the exception-causing instruction might have already begun execution or even modified the processor state.

The goal is usually to provide **precise exceptions**, where:
1.  All instructions before the faulting instruction have completed.
2.  The faulting instruction itself is identified.
3.  No instructions after the faulting instruction have modified the processor state.
4.  The saved PC points to the faulting instruction (or the instruction to restart after handling).

Achieving precise exceptions in deeply pipelined or out-of-order processors can be complex, often requiring hardware support like history buffers or reorder buffers to allow the state to be correctly restored or rolled back. When an exception is detected, the pipeline typically stops fetching, flushes later instructions, allows earlier ones to complete if possible, saves the relevant state, and then branches to an exception handler routine.

## Pipeline Optimization Techniques

Beyond basic hazard handling, several techniques aim to extract more Instruction-Level Parallelism (ILP):

* **Superscalar Execution:** The processor has multiple independent pipelines (e.g., two integer pipelines, one floating-point pipeline) and can fetch, decode, and issue multiple instructions per clock cycle.
* **Superpipelining:** Increases the depth of the pipeline by breaking existing stages into smaller, faster stages. This allows for a higher clock frequency, potentially increasing throughput if hazards don't become a bottleneck.
* **Dynamic Scheduling:** Hardware rearranges instruction execution order at runtime to reduce stalls, going beyond static compiler scheduling.
    * **Scoreboarding:** A centralized control mechanism that tracks instruction status and dependencies, allowing instructions to issue and execute out-of-order when their operands and required functional units are available. It handles RAW hazards dynamically and can detect WAW and WAR hazards, typically by stalling.
    * **Tomasulo's Algorithm:** A more advanced dynamic scheduling technique using distributed **reservation stations** at each functional unit and a **Common Data Bus (CDB)** for broadcasting results. It effectively performs **register renaming** in hardware by using tags in reservation stations to refer to pending results, thus eliminating WAR and WAW hazards without stalling, and efficiently handling RAW hazards.

## Compiler Techniques for Improving Performance

The compiler plays a crucial role in exposing and enhancing ILP for pipelined processors:

1.  **Instruction Scheduling:** The compiler reorders instructions within a basic block (or across blocks if using more advanced techniques) to separate dependent instructions, fill delay slots, and maximize the utilization of pipeline stages.
2.  **Loop Unrolling:** The compiler replicates the loop body multiple times within a single iteration of the modified loop. This reduces the relative overhead of loop control instructions (incrementing counters, branching) and increases the number of instructions available within the loop body for parallel scheduling and pipelining.
3.  **Software Pipelining (Symbolic Loop Unrolling):** A more sophisticated loop optimization technique. The compiler restructures the loop such that each iteration of the *new* loop body contains instructions from *different* iterations of the *original* loop. This allows the pipeline to be kept full by overlapping the execution of multiple original iterations. It involves a "prologue" (to fill the software pipeline), a "steady-state" new loop body, and an "epilogue" (to drain the pipeline). This technique can be very effective for loops without complex loop-carried dependencies.
4.  **Register Renaming (by compiler):** When possible, the compiler assigns different registers to variables that would otherwise cause false dependencies (WAR, WAW), allowing for more scheduling flexibility.

**(Quick Thought:** How does loop unrolling help a pipelined processor more than a non-pipelined one?)
*Answer: It increases the number of independent instructions available within a block, giving the pipeline (especially a superscalar one) more opportunities for parallel execution and hiding latencies.*

## Conclusion: The Power of Parallel Stages

Pipelining is a cornerstone of modern processor design, enabling significant performance gains by overlapping instruction execution. While hazards (structural, data, and control) pose challenges, a combination of hardware techniques (stalling, forwarding, branch prediction, dynamic scheduling) and compiler optimizations (instruction scheduling, loop unrolling, software pipelining) effectively mitigates their impact. The result is a CPU that can sustain a high throughput of instructions, bringing us the speed and responsiveness we expect from today's computers.

**Key Takeaways:**

* Pipelining breaks instruction processing into stages, increasing throughput.
* Structural, data, and control hazards can stall the pipeline and degrade performance.
* Forwarding, stalling, and branch prediction are key hardware techniques to handle hazards.
* Delayed branching is a software-hardware cooperative approach for control hazards.
* Precise exception handling is important in pipelined systems.
* Advanced techniques like superscalar execution and dynamic scheduling (Scoreboarding, Tomasulo's) further enhance ILP.
* Compilers are crucial for optimizing code for pipelined execution through techniques like loop unrolling and software pipelining.
