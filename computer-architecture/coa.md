---
title: A Deeper Dive into Computer Structure and Machine Language"
description: Explore the fundamental building blocks of computers, how they operate at a low level, and the intricacies of the language they understand – machine instructions and assembly.
date: 2023-04-26
draft: false
slug: /pensieve/coa
tags:
  - Computer Architecture
  - CS Basics
---

Hey tech enthusiasts!  Ready to peel back another layer of the onion and get *really* close to the metal? In our previous post, we sketched the basic blueprint of a computer. Now, we'll zoom in, adding more detail about how these components work together and truly understanding the language the processor speaks: machine instructions. Grab your virtual magnifying glass, and let's explore!

## The Blueprint Revisited: Basic Structure of Computers

Recall that computers, fundamentally, are information processors. They handle **instructions** (which dictate operations and data transfers) and **data** (the operands these instructions work on). Both are ultimately represented in binary. The core **Functional Units** orchestrate this processing.

### Functional Units: The Core Components In-Depth

1.  **Input Unit:** The gateway for information entering the computer.
2.  **Memory Unit:** The computer's filing cabinet.
    * **Primary Storage (Main Memory):** Fast, directly accessible by the processor, but typically *volatile* (contents lost when power is off) . This is where active programs and their data must reside . It's made of semiconductor cells, often DRAM (Dynamic RAM), which requires periodic refreshing . Data is accessed in chunks called **words** . Speed is measured by **access time** (time between address issue and data availability) and **cycle time** (minimum time between requests) . The **Memory Hierarchy** (Registers -> Cache -> Main Memory -> Secondary Storage) balances speed, size, and cost .
    * **Secondary Storage:** Slower, larger, non-volatile storage (like magnetic disks, SSDs) used for long-term storage of programs and data not actively in use.
3.  **Arithmetic and Logic Unit (ALU):** The computational core. Performs arithmetic (+, -, *, /) and logic (AND, OR, NOT, XOR) operations . Operands are typically loaded from memory into processor **registers**, processed by the ALU, and the result is stored back in a register or memory . Registers provide fast, temporary storage right next to the ALU.
4.  **Output Unit:** Presents the results of computation to the outside world.
5.  **Control Unit:** The conductor of the orchestra. It fetches instructions from memory, decodes them, and generates control signals and timing signals that dictate the sequence of operations, data transfers within the processor (register-to-register, register-to-ALU), and transfers between the processor and memory or I/O devices. Although often shown as a single block, its functionality is usually distributed throughout the processor.

### Basic Operational Concepts: The Fetch-Execute Cycle Detailed

Executing a single instruction involves a sequence of rudimentary steps, orchestrated by the control unit :

1.  **Fetch Instruction:**
    * The address in the **Program Counter (PC)** is sent to the **Memory Address Register (MAR)**.
    * The Control Unit issues a `Read` signal to memory.
    * Memory retrieves the instruction and places it in the **Memory Data Register (MDR)**.
    * The instruction is transferred from MDR to the **Instruction Register (IR)**.
    * The PC is incremented to point to the next instruction (e.g., `PC ← [PC] + 4` for 4-byte instructions).
2.  **Decode Instruction:** The control unit interprets the instruction held in the IR.
3.  **Fetch Operands:** If the instruction requires data from memory:
    * The operand address is placed in MAR.
    * A `Read` signal is issued.
    * The operand arrives in MDR.
    * Operand is transferred from MDR to the ALU or a general-purpose register.
    If operands are already in general-purpose registers, they are gated to the ALU inputs.
4.  **Execute:** The ALU performs the specified operation (e.g., addition, subtraction, logical AND).
5.  **Store Result:** The result from the ALU is stored:
    * Back into a general-purpose register.
    * Or into memory: The destination address goes to MAR, the result goes to MDR, and a `Write` signal is issued.

**Interrupts:** Normal execution can be temporarily suspended if an I/O device needs urgent attention. The device sends an **interrupt signal** . The processor saves its current state (PC, registers, status flags) and executes a special **interrupt-service routine** to handle the device's request before resuming the original program.

### Bus Structures: Connecting the Units

Communication relies on buses. A bus typically consists of three line types:

* **Address Lines:** Carry the address of the memory location or I/O device being accessed.
* **Data Lines:** Carry the data being transferred.
* **Control Lines:** Carry timing and command signals (Read/Write, Interrupt requests, Clock signals, etc.).

**Single-Bus:** Simple, cost-effective, but a bottleneck as only one transfer can happen at a time.

**Multiple-Bus:** Allows concurrent transfers, significantly improving performance (e.g., fetching the next instruction while executing the current one if caches are used).

**Buffers:** To handle speed mismatches between the fast bus/processor and slower I/O devices, interface circuits use buffers (temporary storage).

## The Processor's Language Deep Dive: Machine Instructions and Programs

Let's refine our understanding of how instructions access data and control the computer.

### Memory Location and Addressing Mechanics

* **Address Space:** A $k$-bit address allows the processor to access $2^k$ unique locations. Common prefixes: 1K = $2^{10}$, 1M = $2^{20}$, 1G = $2^{30}$, 1T = $2^{40}$.
* **Byte Addressability:** Assigning an address to each byte is the most common approach . For a 32-bit (4-byte) word size, word addresses are typically 0, 4, 8, 12...
* **Endianness:** Affects how multi-byte data (like integers or multi-character strings) is interpreted when read from memory.
    * Big-Endian (e.g., Motorola 68k, PowerPC): `0x12` at address 1000, `0x34` at 1001...
    * Little-Endian (e.g., Intel x86): `0x78` at address 1000, `0x56` at 1001...
* **Alignment:** Accessing a 32-bit word at address `1002` (misaligned) might require the hardware to perform two separate 4-byte reads (at 1000 and 1004) and then extract/combine the desired bytes, slowing down the access compared to an aligned read at `1000` or `1004`.

### Memory Operations: Load and Store (RTN View)

Using Register Transfer Notation (RTN) :

* **Load R1, LOC:** `MAR ← [LOC_address]`, `Read`, `wait for MFC (Memory Function Complete)`, `MDR ← [Memory]`, `R1 ← [MDR]` (Simplified)
* **Store R1, LOC:** `MAR ← [LOC_address]`, `MDR ← [R1]`, `Write`, `wait for MFC` (Simplified)

### Instruction Formats: Encoding Operations

The way instructions specify operands impacts code size and complexity. Consider evaluating `X = (A+B) * (C+D)`:

* **Three-Address:** `OP Dest, Src1, Src2`
    ```assembly
    ADD R1, A, B  ; R1 ← M[A] + M[B] 
    ADD R2, C, D  ; R2 ← M[C] + M[D]
    MUL X, R1, R2  ; M[X] ← [R1] * [R2]
    ```
* **Two-Address:** `OP Dest, Src` (Dest often doubles as a source)
    ```assembly
    MOV R1, A    ; R1 ← M[A]
    ADD R1, B    ; R1 ← [R1] + M[B]
    MOV R2, C    ; R2 ← M[C]
    ADD R2, D    ; R2 ← [R2] + M[D]
    MUL R1, R2   ; R1 ← [R1] * [R2]
    MOV X, R1    ; M[X] ← [R1]    
    ```
* **One-Address:** Uses an implicit register (Accumulator - AC)
    ```assembly
    LOAD A       ; AC ← M[A]
    ADD B        ; AC ← [AC] + M[B]
    STORE T      ; M[T] ← [AC]
    LOAD C       ; AC ← M[C]
    ADD D        ; AC ← [AC] + M[D]
    MUL T        ; AC ← [AC] * M[T]
    STORE X      ; M[X] ← [AC]
    ```
* **Zero-Address (Stack):** Operands are implicitly on the stack (TOS=Top Of Stack)
    ```assembly
    PUSH A       ; TOS ← M[A]
    PUSH B       ; TOS ← M[B]      
    ADD          ; TOS ← [TOS] + [TOS-1]
    PUSH C       ; TOS ← M[C]
    PUSH D       ; TOS ← M[D]      
    ADD          ; TOS ← [TOS] + [TOS-1]
    MUL          ; TOS ← [TOS] * [TOS-1]
    POP X        ; M[X] ← [TOS]    
    ```
    (See also Reverse Polish Notation )
* **RISC (Load/Store):** Operations primarily on registers; explicit Load/Store for memory access.
    ```assembly
    LOAD R1, A   ; R1 ← M[A]
    LOAD R2, B   ; R2 ← M[B]      
    LOAD R3, C   ; R3 ← M[C]
    LOAD R4, D   ; R4 ← M[D]      
    ADD R1, R1, R2 ; R1 ← [R1] + [R2]
    ADD R3, R3, R4 ; R3 ← [R3] + [R4]
    MUL R1, R1, R3 ; R1 ← [R1] * [R3]
    STORE X, R1  ; M[X] ← [R1]    
    ```

**(Think:** Which instruction format likely leads to the shortest programs? Which might be easiest for a processor to decode quickly?)

### Addressing Modes: Finding the Operands

How an instruction specifies the *location* of its operand(s) is critical.

| Mode             | Assembler Syntax | Addressing Function          | Example Use                       |
| :--------------- | :--------------- | :--------------------------- | :-------------------------------- |
| Immediate        | `#Value`         | Operand = Value              | Constants (`ADD R1, R1, #10`)     |
| Register         | `Ri`             | EA = Ri (Operand is in Ri)   | Fast access (`ADD R1, R2, R3`)    |
| Absolute (Direct)| `LOC`            | EA = LOC                     | Static variables (`MOVE A, R1`)   |
| Register Indirect| `(Ri)`           | EA = [Ri]                    | Pointers (`MOVE (R1), R2`)        |
| Indirect (Mem)   | `(LOC)`          | EA = [LOC]                   | Pointer variables in memory       |
| Index            | `X(Ri)`          | EA = X + [Ri]                | Array access (`MOVE A(R1), R2`)   |
| Base w/ Index    | `(Ri, Rj)`       | EA = [Ri] + [Rj]             | 2D arrays, complex structures     |
| Base w/Idx+Off   | `X(Ri, Rj)`      | EA = X + [Ri] + [Rj]         | Structure fields within arrays    |
| Relative         | `X(PC)`          | EA = [PC] + X                | Branch targets (`BEQ LOOP`)       |
| Autoincrement    | `(Ri)+`          | EA = [Ri]; Increment Ri      | Step through arrays (`ADD (R2)+, R0`)|
| Autodecrement    | `-(Ri)`          | Decrement Ri; EA = [Ri]      | Stack operations (push/pop)       |

*(EA = Effective Address)*

### IA-32 Specifics Deep Dive (General Concepts)

* **Registers & Flags:** IA-32 extends these general concepts with specific register names and sizes (e.g., 32-bit EAX, 16-bit AX, 8-bit AL/AH) and a dedicated EFLAGS register holding the N, Z, V, C flags along with others controlling processor behavior.
* **Assembly Language:** Uses syntax like `OPCODE destination, source`. Directives like `DB` (Define Byte), `DW` (Define Word), `DD` (Define Doubleword), `EQU` (Equate) are used to define data and constants.
* **Program Flow Control:**
    * **Straight-Line Sequencing:** PC incremented sequentially.
    * **Branching:** `JMP` (unconditional), `JE`, `JNE`, `JG`, `JL`, etc. (conditional based on flags set by `CMP` or `TEST`). The target address is often calculated using **Relative Addressing**.
    * **Loop Example (Summing n numbers):**
        ```assembly
        ; Assume N is in R1, address of first number (NUM1) in R2
        CLR R0          ; R0 = 0 (Sum accumulator)
        LOOP:
            ADD R0, (R2)+ ; Add current number to R0, increment pointer R2
            DEC R1          ; Decrement counter N
            BGT LOOP        ; Branch back to LOOP if R1 > 0 (using N flag implicitly)
        MOV SUM, R0     ; Store result
        ```
        *(Note: Uses generic mnemonics and autoincrement)*
* **Logic and Shift/Rotate:** Essential for low-level control, data packing/unpacking, and fast multiplication/division by powers of 2. The diagrams clearly show how bits move and how the Carry flag (C) might be involved (especially in rotates like RCL/RCR). `CMP` performs a subtraction just to set flags, `TEST` performs an AND just to set flags (e.g., checking if a bit is set).
* **Subroutines:** Crucial for modular programming. The stack is the standard mechanism in IA-32 for managing calls. When `CALL` executes, `[ESP] ← [EIP]` (return address pushed), `ESP ← [ESP] - 4`, `EIP ← subroutine_address`. When `RET` executes, `EIP ← [ESP]`, `ESP ← [ESP] + 4`. Parameters are often pushed onto the stack before the call, and local variables allocated by decrementing ESP further within the subroutine.

### Instruction Encoding: From Assembly to Binary

* **Opcode:** Specifies the operation (ADD, MOVE, etc.) and often implies operand types/locations.
* **Operands/Addressing:** Bits are allocated to specify registers, addressing modes, and any immediate values or offsets.
* **Variable Length (CISC like IA-32):** Instructions can range from 1 byte to many bytes. Allows complex operations and addressing modes in a single instruction, but makes decoding complex.
* **Fixed Length (RISC):** All instructions are the same size (e.g., 32 bits). Simpler decoding and pipelining, but may require more instructions for the same task (e.g., explicit Load/Store).
* **Encoding Example:** An instruction like `Add R1, R2` might have an 8-bit opcode, 4 bits for R1 (Dest), and 4 bits for R2 (Source). An instruction `Move 24(R0), R5` would need bits for the opcode, R5 (Dest), R0 (Base Register), the offset 24, and the addressing mode (Index). If the address or immediate value is too large, it might require extra words/bytes following the main instruction part.

## Conclusion: Mastering the Machine's Tongue

We've delved much deeper into the operational heart of the computer and the specific ways it represents and executes commands. Understanding memory addressing, instruction formats, addressing modes, and control flow is fundamental to grasping how software translates into hardware action. While IA-32 specifics are vast, the general principles illustrated here apply broadly across different computer architectures.

**Key Takeaways:**

* The Fetch-Execute cycle involves precise register transfers (PC->MAR, MDR->IR, etc.) managed by the Control Unit.
* Memory addressing involves concepts like byte-addressability, endianness, and alignment.
* Instruction formats (3-addr, 2-addr, 1-addr, 0-addr, Load/Store) represent different trade-offs.
* Addressing modes (Immediate, Register, Direct, Indirect, Indexed, Relative, Auto-inc/dec) provide flexibility in accessing operands.
* Control flow relies on jumps and conditional branches driven by condition code flags (N, Z, V, C).
* Subroutines use CALL/RET and the stack mechanism for modularity.
* Instruction encoding translates assembly into binary, with CISC (variable-length) and RISC (fixed-length) being major approaches.
