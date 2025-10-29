# Comp-Arch-Study-Group

A comprehensive, self-guided hub that walks from digital logic intuition to advanced system integration on FPGA platforms. Each numbered directory is a self-contained module with its own `README.md`, diagrams, and HDL snippets. Work through them sequentially to follow the layered progression of computer architecture design.

## How to Navigate

- Follow the folders in numeric order; each level builds on the last.
- Start every module by reading its `README.md`; most modules also link to HDL examples, diagrams, and external references.
- HDL code samples default to Verilog unless otherwise noted. Simulation tooling suggestions emphasize open-source flows (Verilator, Yosys, GHDL) with vendor-specific notes where helpful.
- Suggested lab ideas, experiments, and deep dives appear at the end of many sections for hands-on reinforcement.

## Module Overview

### `00_Overview` — Foundations and Roadmap
- **What is Computer Architecture?** Layers of abstraction (logic → microarchitecture → ISA → system), why FPGAs are ideal for architectural exploration, comparing CPUs, accelerators, and peripherals.
- **Project Roadmap:** How the custom RISC-V core threads through the repository, HDL-to-bitstream flow (simulation → synthesis → implementation → programming), summarized toolchain expectations (Vivado, Verilator, GHDL, Yosys, etc.).

### `01_Digital_Logic_Fundamentals` — Logic Building Blocks
- **Combinational Logic:** Boolean algebra refresh, simplification with Karnaugh maps, TTL-to-Verilog intuition, canonical modules (MUX/DEMUX, encoders/decoders, comparators, priority encoders) with concise HDL snippets.
- **Sequential Logic:** Latches, flip-flops, registers, counters, FSM styles (Moore vs Mealy), setup/hold implications, timing diagrams, metastability mitigation practices.
- **Memory Basics:** SRAM vs DRAM characteristics, how FPGA LUTs can implement distributed memories, Verilog RAM templates, intro to BRAM primitives.
- **FPGA Building Blocks:** LUT architecture, flip-flops and DSP slices, BRAM organization, clock resources (MMCM, PLL), and vendor-specific primitives to know.

### `02_Computer_Architecture_Basics` — From Datapath to Pipeline
- **Datapath & Control Path:** ALU composition, register file interface, control signal orchestration, top-level block diagrams linking instruction fetch to write-back.
- **Pipeline Design:** 5-stage pipeline walkthrough (IF, ID, EX, MEM, WB), hazard taxonomy, forwarding paths, stall insertion logic, and interlock examples.
- **Memory Hierarchy:** Cache fundamentals (direct-mapped, fully associative, set-associative), replacement policies, coherence concepts for multi-core designers.
- **Branch Prediction:** Static vs dynamic predictions, bimodal/2-bit predictors, structures such as BTB and BHT, sample predictor state machines.

### `03_RISC-V_Implementation` — Custom Core Deep Dive
- **ISA Overview:** RV32I/M subset, instruction formats (R, I, S, B, U, J), opcode/func3/func7 tables, ALU control decoding.
- **Your Core:** Single-cycle baseline architecture, datapath schematic, control logic, modular RTL organization, accompanying testbenches and simulation tips.
- **Formal Verification:** Assertion-based verification, property templates, introduction to SymbiYosys, equivalence checking with Verilator traces.

### `04_Peripheral_Design` — Talking to the World
- **Bus Systems:** Memory-mapped I/O, address decoding strategies, simple Wishbone-style bus primer.
- **Custom Peripherals:** UART Tx/Rx, SPI and I²C controllers, GPIO module, timer peripheral, each with design notes and HDL references.
- **Interrupt Controller:** Vectoring concepts, masking/priority encoding, integration with the core.
- **AXI Interface:** AXI4-Lite peripheral skeletons, handshaking mechanics (AR/AW/W channels), Vivado IP integration steps, block-design walkthrough connecting RISC-V core to AXI peripherals, interconnect overview.

### `05_Memory_Systems` — Managing Data at Scale
- **Memory Types:** SRAM, DRAM, Flash fundamentals, FPGA-specific usage tips.
- **DDR Protocols:** DDR3/DDR4 signaling overview, key timing parameters (tRCD, tCL, tRP), controller architecture, vendor IP usage.
- **Page Tables:** Virtual memory introduction, page table entry anatomy, TLB concepts, lightweight C-based simulation of page walks.
- **Cache Systems:** Write-through vs write-back, allocation policies, miss penalties, pipeline integration and verification hooks.

### `06_FPGA_Architecture` — Under the Hood of the Fabric
- **Configurable Logic Blocks:** LUT structures, flip-flop packing, carry chains, DSP slices.
- **Routing Fabric:** Switch matrices, global/local interconnect, routing constraints.
- **I/O Blocks:** DDR IO, SERDES overviews, configuring IOB primitives.
- **Clocking and Reset:** Clock domain crossing tactics, metastability handling, reset tree design.
- **Synthesis → Place & Route:** Netlist generation, timing constraints, floorplanning hints, timing closure strategies, bitstream creation.

### `07_System_Integration` — Building the SoC
- **Bus Architecture:** Comparing AXI, AHB, Wishbone; interconnect topologies and arbitration.
- **SoC Design:** Top-level integration checklist for CPU, memory, peripherals; crafting a memory map and address space documentation.
- **Simulation & Debugging:** Full-system simulation with Verilator, wave capture in GTKWave, firmware bring-up, on-chip debug hooks.
- **FPGA Deployment:** Targeting boards like Arty A7, bitstream generation, JTAG programming, UART-based bring-up flow.

### `08_Hardware_Accelerators` — Beyond the General-Purpose Core
- **Why Accelerators?** Performance bottlenecks in von Neumann pipelines, specialization cases.
- **Basic Accelerator Design:** Fixed-function accelerator walkthrough (e.g., matrix multiply), CPU integration via AXI memory-mapped interfaces.
- **Pipelining & Parallelization:** Loop unrolling, dataflow optimizations, initiation interval considerations, intro to HLS toolchains.
- **DMA & Data Movement:** DMA controller concepts, AXI-Stream overview, buffering strategies for high-throughput accelerators.

### `09_Advanced_Topics` — Frontier Architectures
- **RISC-V Extensions:** Adding custom instructions, overview of standard F/D/V extensions, ISA modification workflow.
- **Out-of-Order & Superscalar:** Reservation stations, reorder buffer, Tomasulo’s algorithm synopsis, high-level scheduling diagrams.
- **Coherency Protocols:** MESI/MOESI states, bus-based vs directory-based coherence.
- **Security Features:** PMP/MPU design, MMU protections, speculative execution vulnerabilities and mitigations.
- **Partial Reconfiguration:** Dynamic regions on FPGA, configuration management, adaptive system case studies.

### `10_References_and_Resources` — Curated Study List
- **Books:** _Computer Organization and Design_ (Patterson & Hennessy), _Digital Design and Computer Architecture_ (Harris & Harris).
- **Courses:** Nand2Tetris, MIT 6.004, Berkeley CS152.
- **Specifications & Papers:** RISC-V ISA spec, AMBA AXI, DDR whitepapers, select FPGA architecture papers.
- **Open-Source Cores:** PicoRV32, Rocket, BOOM, VexRiscv, with links and quick comparison notes.
- **FPGA Frameworks:** LiteX, MiSTer, TinyFPGA, and other community projects to explore.

## Suggested Usage Patterns

- **Learning Path:** Progress sequentially, pairing each module with lab work. Branch into advanced sections (08–09) once the SoC baseline is solid.
- **Reference Mode:** Jump directly to topic folders when designing components; each module keeps concise explanations up front with links to deeper references.
- **Collaboration:** Treat the repository like a wiki—open issues for missing explanations, add diagrams or HDL snippets via pull requests, and tag updates by module number.

## Tooling & Environment Notes

- Recommended open-source toolchain: `iverilog`/`gtkwave` for quick tests, `Verilator` for cycle accuracy and C++ test harnesses, `Yosys` + `nextpnr` for vendor-neutral synthesis, `SymbiYosys` for formal checks.
- Vendor tooling: Vivado (Xilinx), Quartus (Intel), Diamond (Lattice) for bitstream generation, IP integration, and timing closure on target boards.
- Code style guidance: consistent module naming, synchronous resets by default, parameterize widths where practical, document assumptions in module headers.

## Roadmap Ideas

- Add HDL labs for each major component (ALU, pipeline stages, cache controller, AXI peripheral).
- Publish simulation walkthroughs highlighting waveform captures and debugging strategies.
- Extend the accelerator section with end-to-end examples (software driver + HDL + testbench).
- Incorporate partial reconfiguration demos and security-oriented case studies.

## Contributing

1. Fork or branch from `main`.
2. Add or expand module `README.md` files with explanations, diagrams, and HDL samples.
3. Run any provided simulations/tests for the module touched.
4. Open a pull request summarizing the educational angle and any verification results.

Let this repository be a living notebook. Iterate on the content as you learn, wire up experiments, and capture insights for the next engineer exploring computer architecture through FPGA systems.
