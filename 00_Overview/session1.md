# What is computer architecture?

It is, in very simple terms, the structure of a computer system based on component parts.

## Layers of Abstraction

 1. Microarchitecture (Implementation)

 This explains how the ISA is physically realised in hardware. It explains how different chips are optimized to the maximum (by using less wires and registers, to make it more energy efficient and cost effective). In simple terms, how gates are connected to do to make it work.

2. Instruction Set Architecture (ISA)

The ISA is the interface between hardware and software. It specifies:
 - The set of instructions
 - Data types and registers.
 - Memory addressing modes.

 Some examples include, x86 (used in most PCs, introduced by Intel), ARM (common in mobile devices), RISC-V (common open source ISA).


 3. System Architecture

 This specifies the full computer -> CPU + RAM + I/O etc.
---

 > You design from the bottom up, but program from the top down.

You start with physics, think about constraint to make the physical chip to build one, but when you program the abstraction goes the other way about.


Think of it like layers in a game engine:

- Logic is the physics engine (rules of reality),

- Microarchitecture is the mechanics (how motion happens),

- ISA is the API the player uses,

- System is the whole game world.

> FPGAs are perfect for architectural exploration

Field-programmable Gate Arrays (FPGAs) are reconfigurable chips that allow us to implement and test the real hardware without fabricating silicon.

This is perfect for prototyping and experimenting custom CPU designs, Accelerators, because you can modify and re-synthesize your logic instantly.

### CPU vs Accelerator vs Peripheral

| Component                         | Role                                                                                 | Typical Function                      |
| --------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------- |
| **CPU (Central Processing Unit)** | General-purpose executor. Executes instructions sequentially or in limited parallel. | Arithmetic, control, memory access    |
| **Accelerator**                   | Specialized compute unit for one type of workload.                                   | AI, DSP, image processing, encryption |
| **Peripheral**                    | External interface for I/O or timing.                                                | UART, SPI, GPIO, Timer, AXI bridges   |


if you don't understand don't worry, further sessions will help you understand all of them.