---
layout: default
title: Session 1 – What is Computer Architecture?
parent: "00 – Overview"
nav_order: 2
---

# What is computer architecture?

In very simple terms, computer architecture is the structure and behaviour of a computer system as seen through its major components and the rules that connect them. It sits between raw devices (wires, gates, memories) and full systems (CPUs, SoCs, boards, and software stacks).

## Layers of Abstraction

We will keep returning to three main layers:

1. **Microarchitecture (Implementation)**  
   This explains how the ISA is physically realised in hardware. It covers how different chips are optimised (fewer wires and registers, better pipelines, smarter control) to meet performance, power, and area constraints. In simple terms: how gates are connected to make the ISA actually work.

2. **Instruction Set Architecture (ISA)**  
   The ISA is the interface between hardware and software. It specifies:
   - the set of instructions,
   - the visible registers and data types, and
   - the memory addressing modes and calling conventions.

   Examples include x86 (most PCs), ARM (common in mobile and embedded), and RISC‑V (an open ISA we will use in this repository).

3. **System Architecture**  
   This layer specifies the full computer: CPU + memory + interconnect + I/O. It decides how many cores exist, how caches and memories are arranged, what buses connect peripherals, and how the platform appears to an operating system.

---

> You design from the bottom up, but you program from the top down.

When designing, you start from physics and devices, then build up to gates, datapaths, and ISAs. When programming, you start from the ISA and system view, and only occasionally think about the microarchitecture below.

A useful mental model is a game engine:

- **Logic** is the physics engine (the rules of reality).  
- **Microarchitecture** is the mechanics (how motion and interactions are actually implemented).  
- **ISA** is the gameplay API that scripts use.  
- **System** is the whole game world and its resources.

## Why FPGAs are ideal for architectural exploration

Field‑Programmable Gate Arrays (FPGAs) are reconfigurable chips that allow us to implement and test real hardware without fabricating silicon. We describe hardware in HDL, synthesize it, and the FPGA fabric behaves like the designed circuit.

For this repository, that means:

- we can prototype and iterate on custom CPU cores and accelerators,
- we can experiment with different microarchitectures under the same ISA, and  
- we can observe real timing and resource usage rather than just simulate on paper.

Because the fabric is reprogrammable, we can modify and re‑synthesise our logic quickly, which is perfect for learning and for architectural “what‑if” experiments.

## CPU vs Accelerator vs Peripheral

At the system level, it is useful to distinguish three roles you will see throughout the modules:

| Component                         | Role                                                                                 | Typical Function                      |
| --------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------- |
| **CPU (Central Processing Unit)** | General-purpose executor. Executes instructions sequentially or in limited parallel. | Arithmetic, control, memory access    |
| **Accelerator**                   | Specialised compute unit for one type of workload.                                   | AI, DSP, image processing, encryption |
| **Peripheral**                    | External interface for I/O or timing.                                                | UART, SPI, GPIO, Timer, AXI bridges   |

Later modules will show how these are interconnected on a bus, how the CPU controls accelerators via memory‑mapped registers, and how peripherals expose clean interfaces to software.

## Project roadmap for this repository

The rest of the repository is organised to take you from this conceptual view down into implementation details and back up into full systems:

- **00 – Overview** (this module) introduces the architectural layers and the overall journey.  
- **01 – Digital Logic Fundamentals** builds the combinational and sequential blocks you need to implement datapaths and control.  
- **02 – Computer Architecture Basics** shows how datapath and control come together into a simple processor.  
- **03 – RISC‑V Implementation** guides you through a custom RV32 core, from ISA decoding to pipeline organisation.  
- **04 – Peripheral Design** adds UARTs, timers, GPIO, and bus logic so the core can talk to the outside world.  
- **05 – Memory Systems** introduces caches and memory hierarchy concepts that affect performance.  
- **06 – FPGA Architecture** explains what is actually inside the FPGA that realises your designs.  
- **07 – System Integration** turns individual blocks into a small SoC on an FPGA board.  
- **08 – Hardware Accelerators** explores specialised compute units coupled to your core.  
- **09 – Advanced Topics** touches on out‑of‑order ideas, coherence, and security.  
- **10 – References and Resources** is a curated reading and viewing list.

As you move through these modules, you will repeatedly connect back to the abstractions from this session: what parts belong to the ISA, what belongs to the microarchitecture, and how the system around the CPU completes the picture.
