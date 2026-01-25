---
title: Connecting to the World - A Deep Dive into Computer Input/Output (I/O) Organization
description: Exploring how computers manage communication with peripherals, from basic I/O operations and interrupts to DMA, interface circuits, and standard I/O buses like PCI, SCSI, and USB. We'll also touch on RISC/CISC and Flynn's Classification.
date: 2023-04-30
draft: false
slug: /pensieve/coa/io-organisation
tags:
  - Computer Architecture
  - CS Basics
---

Hello, fellow tech explorers! So far, we've dissected the computer's brain (CPU), its calculator (ALU), and its memory. But how does this self-contained world communicate with us and other devices? Keyboards, mice, displays, disks, network cards – they all need to talk to the processor and memory. This is where **Input/Output (I/O) Organization** comes into play. It's all about managing the flow of data between the central components and the vast array of peripherals.

## Basic Input and Output Operations

I/O operations are fundamental for transferring data between the computer and external devices. Think about typing on a keyboard (input) or seeing an image on a screen (output). A significant challenge in I/O is the speed disparity: processors and memory operate much faster than most I/O devices. This mismatch necessitates mechanisms to synchronize data transfers.

One of the simplest methods is **Program-Controlled I/O**. Here, the processor directly manages the I/O operation by repeatedly checking the status of the I/O device until it's ready for data transfer. This is also known as "polling."

For example, to read a character from a keyboard:
1.  The processor reads a status flag from the keyboard interface.
2.  If the flag indicates a character is ready (e.g., `SIN=1`), the processor reads the character from a data register in the interface.
3.  If not ready, the processor loops back to step 1, effectively "waiting" or "polling" the device.

While simple, program-controlled I/O can be inefficient as the processor spends a lot of time in wait loops.

## Accessing I/O Devices

I/O devices are connected to the processor and main memory, typically via a **bus**. This bus carries address, data, and control signals. There are two main ways I/O devices are addressed:

1.  **Memory-Mapped I/O:** I/O devices and memory share the same address space. A range of memory addresses is reserved for I/O device registers. Any machine instruction that can access memory (like `MOV`, `LOAD`, `STORE`) can be used to transfer data to or from the I/O device registers. This simplifies software as no special I/O instructions are needed.
2.  **I/O-Mapped I/O (or Port-Mapped I/O):** I/O devices have a separate address space from memory. Special input (`IN`) and output (`OUT`) instructions are used to communicate with the devices. This can sometimes simplify hardware as I/O devices might deal with fewer address lines, or a special control line on the bus indicates an I/O operation.

Regardless of the mapping scheme, each I/O device connects to the bus through an **I/O interface circuit** (also called an I/O module or port). This circuit typically includes:
* **Address Decoder:** Recognizes the device's assigned address on the address bus.
* **Data Register(s):** Temporary storage for data being transferred (e.g., `DATAIN` for input, `DATAOUT` for output).
* **Status Register(s):** Holds information about the device's state (e.g., data ready, device busy, errors).
* **Control Circuits:** Manages the transfer of data and responds to control signals from the bus.

## Interrupts: A More Efficient Way to Handle I/O

Continuously polling devices is wasteful. A more efficient approach is for the I/O device to alert the processor when it's ready for service. This is done using an **interrupt**.

### Interrupt Hardware and Process
* A dedicated **interrupt-request line** on the bus is used by devices to signal the processor.
* When the processor detects an active interrupt request, and if interrupts are enabled, it will:
    1.  Complete the currently executing instruction (usually).
    2.  Save the current state of the processor, minimally the **Program Counter (PC)** and the **Processor Status Word (PSW)** (which includes condition codes and control flags). This saved state is often pushed onto the processor stack.
    3.  Identify the interrupting device (more on this later).
    4.  Load the starting address of the specific **Interrupt Service Routine (ISR)** for that device into the PC.
    5.  The ISR is then executed. It performs the necessary operations to service the device (e.g., transfer data).
    6.  After the ISR completes, the saved processor state is restored from the stack.
    7.  Execution of the interrupted program resumes from where it left off.
* The processor may send an **interrupt-acknowledge signal** back to the device to confirm it's handling the request.

**Interrupt Latency** is the time delay between an interrupt request being asserted and the start of the ISR execution. Minimizing this is crucial for real-time systems.

### Enabling and Disabling Interrupts
Processors provide mechanisms to **enable** or **disable** interrupts, either globally or for specific interrupt sources.
* Often, there's an interrupt-enable bit in the PSW. If clear, the processor ignores external interrupt requests.
* Specific devices or interrupt lines can often be individually enabled or disabled, usually by writing to control registers in their interface circuits.
* It's common practice for the processor to automatically disable further interrupts when it starts executing an ISR and re-enable them just before returning from the ISR. This prevents an ISR from being interrupted by a lower-priority or the same device.

### Handling Multiple Devices
When multiple devices can generate interrupts, the system needs to:
1.  **Identify the interrupting device:**
    * **Polling:** The ISR polls the status register of each potential device to find which one requested the interrupt. Simple but can be slow.
    * **Vectored Interrupts:** The interrupting device provides an identifier (a "vector") to the processor. This vector can be an address or an index into an **interrupt vector table** in memory, which contains the starting addresses of the ISRs. This is much faster.
2.  **Manage Simultaneous Requests (Priority Arbitration):** If multiple devices request an interrupt at the same time, or if a higher-priority device requests an interrupt while a lower-priority ISR is running, a priority system is needed.
    * **Separate Request/Acknowledge Lines:** Each device or priority level has its own interrupt request line. The processor has arbitration logic to select the highest-priority active request.
    * **Daisy Chaining:** Devices are connected in series. An interrupt-acknowledge signal propagates from the highest-priority device to the lowest. The first device in the chain that has an active request blocks the acknowledge signal from passing further and identifies itself. Priority is determined by electrical position.
    * Hybrid schemes combining multiple priority lines with daisy chains on each line are also common.

### Controlling Device Requests
Beyond processor-level enable/disable, each device interface typically has an **interrupt-enable bit**. The device can only generate an interrupt request if its own enable bit is set and the processor's interrupt system is also receptive.

### Exceptions
The term **exception** is a broader category for any event that causes an interruption of normal program execution. I/O interrupts are one type. Others include:
* **Errors:** Hardware malfunctions, memory errors (e.g., parity error), or errors during instruction execution (e.g., division by zero, illegal opcode).
* **Debugging Support:**
    * **Trace Mode:** An exception occurs after every instruction, allowing a debugger to single-step.
    * **Breakpoints:** An exception occurs when a specific instruction address (a breakpoint set by the debugger) is reached.
* **Privilege Violations:** Attempting to execute a privileged instruction (e.g., one that modifies system control registers) in user mode. The processor usually switches to a more privileged supervisor mode to handle such exceptions.

Exception handling is similar to interrupt handling: the processor state is saved, and a specific service routine is executed. For errors, the ISR might terminate the program or attempt recovery. For debugging exceptions, control typically passes to a debugger program.

## Direct Memory Access (DMA)

For transferring large blocks of data between an I/O device (like a disk) and main memory, interrupts can be inefficient because the processor would be involved in transferring every single word or byte. **Direct Memory Access (DMA)** allows an I/O device to transfer data directly to or from main memory, without continuous processor intervention.

* A **DMA Controller (DMAC)**, usually part of the I/O device's interface circuit, manages the transfer.
* **Initiation:** The processor initiates a DMA transfer by providing the DMAC with:
    * The starting memory address for the transfer.
    * The number of words/bytes to transfer.
    * The direction of transfer (I/O device to memory, or memory to I/O device).
    * The I/O device's address/port.
* **Transfer:** Once initiated, the DMAC takes over. It generates memory addresses, asserts bus control signals, and manages the data transfer directly with memory.
* **Completion:** When the entire block transfer is complete, the DMAC informs the processor by sending an interrupt signal.

### Bus Arbitration for DMA
Since the DMAC needs to control the memory bus to perform transfers, it must become the **bus master**. The process of deciding which device (processor or a DMAC) gets to be the bus master is called **bus arbitration**.
* **Cycle Stealing:** The DMAC requests the bus from the processor. When granted, it "steals" a few bus cycles to transfer one or a few words, then relinquishes the bus. This interleaves DMA transfers with processor activity.
* **Block or Burst Mode:** The DMAC gains exclusive control of the bus to transfer an entire block of data in one burst. This can temporarily halt the processor if it needs the bus.
* **Arbitration Schemes:**
    * **Centralized Arbitration:** A dedicated bus arbiter (which could be the processor itself or a separate chip) manages requests and grants bus mastership. Daisy-chaining is often used for grant signals.
    * **Distributed Arbitration:** All devices wanting to use the bus participate in a protocol to select the next bus master, often based on device IDs or priorities.

## Interface Circuits: The Bridge to Peripherals

An I/O interface circuit acts as the bridge between the computer's system bus and the I/O device. It translates bus signals into device-specific signals and vice-versa.
Key components include address decoders, data registers (like `DATAIN` for input, `DATAOUT` for output), status registers (with flags like `SIN` - input ready, `SOUT` - output ready), and control logic.

### Parallel Port
A parallel port transfers multiple bits of data (e.g., 8 or 16) simultaneously over parallel data lines.
* **Input (e.g., Keyboard):** When a key is pressed, an encoder generates a code. A `Valid` signal might indicate data is ready in the `DATAIN` register of the interface, setting a status flag (`SIN`). The processor reads the status, then the data.
* **Output (e.g., Printer):** The processor checks a status flag (`SOUT`) set when the printer is `Idle` (ready for data). If ready, the processor writes a character to the `DATAOUT` register. The interface then asserts a `Valid` signal to the printer along with the data.

### Serial Port
A serial port transfers data one bit at a time over a single data line (or a pair for full-duplex). This is suitable for longer distances due to fewer wires and less skew issues.
* **Operation:** Requires **shift registers** for parallel-to-serial conversion (for output) and serial-to-parallel conversion (for input).
* **Input:** Bits arrive serially and are clocked into an input shift register. Once a full byte/word is assembled, it's transferred in parallel to a `DATAIN` buffer register for the processor to read.
* **Output:** The processor writes a byte/word in parallel to a `DATAOUT` buffer register. This is then loaded in parallel into an output shift register, from which bits are clocked out serially.
* **Double-Buffering:** Using both a shift register and a data buffer (e.g., DATAIN for input) allows the serial port to start receiving the next character while the processor is still reading the previous one from the buffer, improving efficiency.
* **Bit Rate:** The speed of serial transmission. Interfaces support various configurable clock speeds to match different devices.

## Standard I/O Interfaces

To simplify connecting a wide variety of peripherals from different manufacturers, standard interface protocols and bus architectures have been developed. Modern PCs often have a **processor bus** (high-speed, close to CPU) and one or more **expansion buses** (for other devices), connected by a **bridge circuit** that translates signals and protocols.

### PCI Bus (Peripheral Component Interconnect)
A popular, processor-independent expansion bus known for its plug-and-play capabilities.
* Supports burst data transfers efficiently.
* Uses multiplexed address and data lines (AD lines) to save pins.
* An **initiator** (master) starts a transaction with a **target** (slave).
* Key signals include `FRAME#` (duration of transaction), `C/BE#` (command/byte enables), `IRDY#` (Initiator Ready), `TRDY#` (Target Ready), and `DEVSEL#` (Device Select by target).
* Each PCI device has a **configuration ROM** storing its information, which allows the system to identify and configure it during boot-up (plug-and-play).

### SCSI Bus (Small Computer System Interface)
A standard primarily for connecting storage devices like disks and tape drives, though it can support other peripherals.
* Devices on the SCSI bus are *not* directly in the processor's memory-mapped address space. A **SCSI controller** (host adapter) connects the SCSI bus to the system bus, typically using DMA for transfers.
* Communication involves **initiators** (e.g., SCSI controller) and **targets** (e.g., disk drive).
* Uses a packet-based protocol for commands, data, and status information.
* A key feature is its ability to manage multiple pending operations. An initiator can select a target, send a command, and then the target can disconnect from the bus (releasing it for other devices) while it processes the command (e.g., performing a disk seek). When ready, the target re-selects the initiator to transfer data. This allows for high throughput with multiple devices.
* Data transfers are controlled by the target controller.

### USB (Universal Serial Bus)
A widely adopted standard for connecting a vast range of typically lower-to-medium speed peripherals (keyboards, mice, printers, external drives, cameras, etc.).
* **Topology:** Uses a tiered-star (tree) structure. The **host controller** (root hub) is at the top, connecting to **hubs**, which in turn connect to other hubs or I/O devices.
* **Addressing:** Each device on the USB tree is assigned a unique 7-bit address by the host when it's connected (plug-and-play).
* **Communication:** The host polls hubs to detect new devices and manages all communication. Data is transferred in **packets** (control packets for addressing/acknowledging, data packets for actual information).
* **Isochronous Traffic:** Supports devices requiring guaranteed bandwidth and delivery times (like audio or video) by dividing transmission into 1ms **frames**. The host schedules transfers for isochronous devices within these frames.
* **Cables:** Four wires: two for power (+5V, Ground) allowing bus-powered devices, and two for differential data signals.

## Flynn's Classification of Computer Architectures

Michael J. Flynn proposed a classification for computer architectures based on the number of concurrent instruction streams and data streams:

1.  **SISD (Single Instruction stream, Single Data stream):** Traditional uniprocessor systems. One instruction operates on one piece of data at a time.
2.  **SIMD (Single Instruction stream, Multiple Data streams):** A single instruction is executed simultaneously on multiple data elements. Common in array processors and vector processors, good for tasks with high data parallelism (e.g., graphics, scientific computing).
3.  **MISD (Multiple Instruction streams, Single Data stream):** Multiple instructions operate on a single data stream. This category has few, if any, practical commercial examples.
4.  **MIMD (Multiple Instruction streams, Multiple Data streams):** Multiple processors execute different instructions on different data streams concurrently. This includes multiprocessors and multicomputer systems.
    * **Shared Memory MIMD:** Processors share a common global address space. Subcategories include UMA (Uniform Memory Access, like bus-based SMPs) and NUMA (Non-Uniform Memory Access, where access time varies depending on memory location relative to the processor).
    * **Distributed Memory MIMD:** Each processor has its own private memory. Communication occurs via message passing over a network.

## RISC vs CISC Architectures Revisited

We've touched on this before, but it's relevant to overall system design including I/O handling.

* **CISC (Complex Instruction Set Computer):**
    * **Philosophy:** Aim to make hardware powerful by including complex instructions that can perform multiple low-level operations (e.g., a single instruction to load from memory, add to a register, and store back to memory). Goal was to reduce the "semantic gap" between high-level languages and machine language, and reduce the number of instructions per program.
    * **Characteristics:** Many instructions, complex addressing modes, variable instruction lengths, often microprogrammed control.
* **RISC (Reduced Instruction Set Computer):**
    * **Philosophy:** Simplify the instruction set to a smaller number of basic operations that can execute very quickly, usually in a single clock cycle within a pipeline. Complex operations are built by the compiler using sequences of these simple instructions.
    * **Characteristics:** Few instructions, simple addressing modes, fixed instruction length, load/store architecture (only load/store instructions access memory, ALU operations are register-to-register), emphasis on pipelining, often hardwired control.

**Performance Equation ($T = (N \times S) / R$):**
* CISC aims to reduce N (number of instructions).
* RISC aims to reduce S (average cycles per instruction) and enable a higher R (clock rate) due to simpler logic.
The role of the compiler is more critical in RISC to optimize instruction sequences. Modern high-performance processors often blend ideas from both.

## Conclusion: The Computer's Dialogue with the World

Input/Output organization is a vast and vital area of computer architecture. It encompasses how the CPU communicates with a diverse array of peripherals operating at different speeds, manages asynchronous events through interrupts, and offloads bulk data transfers using DMA. Standard interfaces like PCI, SCSI, and USB have been crucial in enabling the rich ecosystems of devices we use today. Understanding these mechanisms, alongside architectural choices like RISC/CISC and parallel processing models, provides a complete picture of how computers function as integrated systems.

**Key Takeaways:**

* I/O operations are managed through interfaces, with methods like program-controlled I/O, interrupts, and DMA.
* Interrupts allow devices to signal the CPU, improving efficiency over polling. Handling multiple interrupting devices involves identification and priority arbitration.
* DMA allows direct data transfer between I/O and memory, freeing up the CPU. Bus arbitration manages access to the shared bus.
* Interface circuits (parallel/serial ports) bridge the CPU bus and device-specific needs.
* Standard I/O buses (PCI, SCSI, USB) provide common frameworks for device connectivity, each with unique features and protocols.
* Flynn's classification helps categorize parallel computer architectures.
* RISC and CISC represent different philosophies in instruction set design with distinct trade-offs.
