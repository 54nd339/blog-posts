---
title: Computer Architecture - The Machine and Its Instruction Set
description: How a processor is built from functional units, how the fetch-execute cycle drives it, and how machine instructions name and reach their operands - memory addressing, instruction formats, addressing modes, program flow, and the RISC/CISC split.
date: 2022-02-11
draft: false
slug: /computer-architecture/coa
tags:
  - Computer Architecture
  - CPU
  - Instruction Sets
---

A processor does exactly one thing in a loop: read the next instruction from memory, work out what it means, carry it out. Instructions and the data they act on are both just binary in the same memory. Everything a CPU appears to *do* — run a browser, decode video, train a model — is that loop going fast over instructions someone (usually a compiler) laid down in advance.

This post is the machine's structure and the language it runs: the functional units, the fetch-execute cycle that ties them together, and then the part that trips people up — how a single instruction says *which* data it wants and *where* that data lives. It's the entry point to a series that then goes deep on [the datapath and control unit](/citadel/computer-architecture/processing-unit), [arithmetic](/citadel/computer-architecture/arithmetic), [memory technology](/citadel/computer-architecture/memory-system), [the cache and virtual-memory hierarchy](/citadel/computer-architecture/hierarchical-memory), [pipelining](/citadel/computer-architecture/pipelining), [instruction-level parallelism](/citadel/computer-architecture/ilp), [I/O](/citadel/computer-architecture/io-organisation), and [multiprocessors](/citadel/computer-architecture/multiprocessor).

## The functional units

Five kinds of block, wired together:

- **Input** and **output** units — the doorways for data coming from and going to keyboards, disks, network cards, screens.
- **Memory** — where instructions and their operands live while a program runs. *Main memory* (usually DRAM) is fast and directly reachable by the processor but volatile; *secondary storage* (disk, SSD) is larger, slower, and keeps its contents with the power off. Data moves in and out in fixed-size chunks called **words** (32 or 64 bits on most machines).
- **Arithmetic and logic unit (ALU)** — the part that actually computes: add, subtract, multiply, divide, AND, OR, NOT, XOR. Its operands come from small, fast storage cells called **registers** sitting right next to it.
- **Control unit** — the conductor. It fetches each instruction, decodes it, and raises the timing and control signals that move data between registers, through the ALU, and to or from memory. It's drawn as one block but its logic is spread across the whole processor.

## The fetch-execute cycle

Running one instruction is a fixed sequence of tiny register-to-register moves, using a handful of special registers: the **program counter (PC)** holding the address of the next instruction, the **instruction register (IR)** holding the current one, and the **memory address register (MAR)** / **memory data register (MDR)** pair that talk to memory.

1. **Fetch.** The PC goes to the MAR. The control unit issues `Read`. Memory returns the instruction into the MDR, which is copied to the IR. The PC is bumped to the next instruction: `PC ← [PC] + 4` for 4-byte instructions. (Square brackets mean "contents of".)
2. **Decode.** The control unit interprets the opcode in the IR.
3. **Fetch operands.** If the instruction needs data from memory, its address goes to the MAR, `Read` is issued, the operand arrives in the MDR and moves to the ALU or a register. Operands already in registers are just gated to the ALU inputs.
4. **Execute.** The ALU performs the operation.
5. **Store result.** The result goes back to a register, or out to memory (destination address to MAR, data to MDR, `Write`).

Normal sequencing can be broken by an **interrupt** — an I/O device raising a line to say it needs attention. The processor saves its state (PC, registers, status flags), runs an *interrupt-service routine*, then restores state and resumes. [I/O organisation](/citadel/computer-architecture/io-organisation) covers interrupts, priorities, and DMA in full.

## Buses

The units are connected by **buses**, each a bundle of three line types: **address lines** carry which location is being accessed, **data lines** carry the value, **control lines** carry `Read`/`Write`, clock, and interrupt signals.

A **single-bus** design is cheap and simple but serialises everything — one transfer at a time. **Multiple buses** allow concurrent transfers (fetching the next instruction while the current one executes), at more cost and complexity. Speed mismatches between the fast bus and slow devices are smoothed by **buffers** — small holding registers in the interface circuits.

## How memory is addressed

A $k$-bit address selects one of $2^k$ locations. The usual size prefixes are powers of two: 1K = $2^{10}$, 1M = $2^{20}$, 1G = $2^{30}$, 1T = $2^{40}$.

**Byte-addressable** memory gives every byte its own address, so a machine with 32-bit (4-byte) words has word addresses 0, 4, 8, 12, … Two wrinkles follow from packing multi-byte values into byte-addressed memory:

- **Endianness** — the order the bytes of a word are stored in. *Big-endian* (PowerPC, older Motorola) puts the most significant byte at the lowest address; *little-endian* (x86) puts the least significant byte there. Storing `0x12345678` at address 1000: big-endian has `0x12` at 1000, little-endian has `0x78` at 1000. It only matters when the same bytes are read at a different width, or moved between machines.
- **Alignment** — a 4-byte word is *aligned* when its address is a multiple of 4. Reading a misaligned word at address 1002 can force the hardware to do two reads (at 1000 and 1004) and splice the result, so aligned access is faster.

### Load and store

Only two operations move data between a register and memory. In register-transfer notation:

- **`Load R1, LOC`** — `MAR ← LOC`, `Read`, wait for memory, `MDR ← [memory]`, `R1 ← [MDR]`.
- **`Store R1, LOC`** — `MAR ← LOC`, `MDR ← [R1]`, `Write`, wait for memory.

## Instruction formats

How many operand addresses an instruction carries is a design choice with real consequences. Here is `X = (A + B) * (C + D)` in five styles.

**Three-address** — `OP dest, src1, src2`. Shortest programs, widest instructions:

```asm
ADD R1, A, B    ; R1 ← M[A] + M[B]
ADD R2, C, D    ; R2 ← M[C] + M[D]
MUL X,  R1, R2  ; M[X] ← R1 * R2
```

**Two-address** — `OP dest, src`, with `dest` also a source:

```asm
MOV R1, A
ADD R1, B       ; R1 ← R1 + M[B]
MOV R2, C
ADD R2, D
MUL R1, R2
MOV X,  R1
```

**One-address** — an implicit accumulator (`AC`) is always one operand and the destination:

```asm
LOAD  A         ; AC ← M[A]
ADD   B         ; AC ← AC + M[B]
STORE T
LOAD  C
ADD   D
MUL   T         ; AC ← AC * M[T]
STORE X
```

**Zero-address** — operands are implicit on a stack; `ADD` pops two and pushes their sum:

```asm
PUSH A
PUSH B
ADD             ; push (A + B)
PUSH C
PUSH D
ADD             ; push (C + D)
MUL             ; push the product
POP  X
```

**RISC load/store** — arithmetic touches registers only; memory access is explicit:

```asm
LOAD R1, A
LOAD R2, B
LOAD R3, C
LOAD R4, D
ADD  R1, R1, R2
ADD  R3, R3, R4
MUL  R1, R1, R3
STORE X, R1
```

The trend down the list: fewer operand fields per instruction means more instructions per task, but each instruction is smaller and easier to decode. That tension is the whole RISC/CISC argument below.

## Addressing modes

Within an instruction, the *addressing mode* says how to turn the operand field into the operand's actual location — its **effective address (EA)**.

| Mode | Syntax | Effective address | Typical use |
| --- | --- | --- | --- |
| Immediate | `#Value` | operand *is* `Value` | constants |
| Register | `Ri` | operand is in `Ri` | fast temporaries |
| Absolute (direct) | `LOC` | EA = `LOC` | static variables |
| Register indirect | `(Ri)` | EA = `[Ri]` | pointers |
| Memory indirect | `(LOC)` | EA = `[LOC]` | pointer variables in memory |
| Index | `X(Ri)` | EA = `X + [Ri]` | array element |
| Base with index | `(Ri, Rj)` | EA = `[Ri] + [Rj]` | 2D arrays |
| Base with index + offset | `X(Ri, Rj)` | EA = `X + [Ri] + [Rj]` | struct field inside an array |
| Relative | `X(PC)` | EA = `[PC] + X` | branch targets |
| Autoincrement | `(Ri)+` | EA = `[Ri]`, then `Ri += 1` | stepping through an array |
| Autodecrement | `-(Ri)` | `Ri -= 1`, then EA = `[Ri]` | stack push/pop |

## Program flow

By default the PC just increments — **straight-line sequencing**. Two things change that:

- **Branches.** A conditional branch tests the **condition-code flags** — N (negative), Z (zero), V (overflow), C (carry) — that a prior `CMP` (subtract, keep only flags) or `TEST` (AND, keep only flags) set. `JE`, `JNE`, `JG`, `JL` and friends branch or don't based on those flags; the target is usually computed with relative addressing. Summing `N` numbers starting at address `NUM1`:

  ```asm
  CLR  R0            ; running sum
  LOOP:
      ADD  R0, (R2)+ ; add current number, advance pointer
      DEC  R1        ; count down
      BGT  LOOP      ; repeat while count > 0
  MOV  SUM, R0
  ```

- **Subroutines.** `CALL` pushes the return address onto the stack (`[ESP] ← [EIP]`, `ESP ← [ESP] − 4` on IA-32) and jumps; `RET` pops it back. Arguments are typically pushed before the call; local variables are carved out by decrementing the stack pointer further inside the routine. IA-32 layers concrete names on all of this — 32/16/8-bit register views (`EAX`/`AX`/`AL`), an `EFLAGS` register holding the condition codes, and assembler directives like `DB`/`DW`/`DD` to lay out data.

## Instruction encoding

An instruction in binary is an **opcode** field (which operation) plus fields for registers, addressing modes, and any immediate values or offsets. `Add R1, R2` might be an 8-bit opcode and two 4-bit register fields; `Move 24(R0), R5` needs opcode, two register fields, the mode, and room for the offset 24 — and if an immediate or address is too big for the base instruction, it spills into an extra word.

Two philosophies:

- **Variable length (CISC, e.g. x86)** — instructions run from 1 byte to a dozen-plus. Rich operations and addressing modes in one instruction; decoding is hard.
- **Fixed length (RISC)** — every instruction is the same width (typically 32 bits). Trivial to decode and pipeline; more instructions needed per task.

## RISC vs CISC

**CISC** aims to shrink the number of instructions per program by making each instruction do more — memory operands, complex addressing, a single instruction that loads, adds, and stores. It leans on [microprogrammed control](/citadel/computer-architecture/processing-unit) and variable-length encoding.

**RISC** aims to make each instruction execute in one fast pipeline cycle: a small operation set, few addressing modes, fixed length, load/store separation, hardwired control. The compiler builds complex behaviour from simple pieces.

The performance equation makes the trade-off precise. Execution time $T = \dfrac{N \times S}{R}$, where $N$ is instructions per program, $S$ is average cycles per instruction, and $R$ is clock rate. CISC pushes $N$ down; RISC pushes $S$ down and lets $R$ go up because the logic is simpler. Modern x86 chips blend both — a hardware front end cracks complex instructions into RISC-like micro-ops.

## The one idea to keep

The processor's job is a three-step loop over binary, and the instruction set is the contract for that loop: an opcode picks the operation, an addressing mode turns an operand field into a real location, and the condition-code flags let the next instruction depend on the last one's result. Every later topic in this series — pipelines, caches, out-of-order execution, multiple cores — is an optimisation layered on top of that contract without changing what the contract says.
