# Comp-Arch-Study-Group

A comprehensive, self-guided hub that walks from digital logic intuition to advanced system integration on FPGA platforms. Each numbered directory is a self-contained module with its own notes, diagrams, and HDL snippets. Work through them sequentially to follow the layered progression of computer architecture design.

The content of this repository is also published as a documentation site at:

- `https://comparch.5iri.me/`

## Roadmap (at a glance)

```text
comp-arch-study-group/
├── 00_Overview/                 # Foundations and roadmap
├── 01_Digital_Logic_Fundamentals/
├── 02_Computer_Architecture_Basics/
├── 03_RISC-V_Implementation/
├── 04_Peripheral_Design/
├── 05_Memory_Systems/
├── 06_FPGA_Architecture/
├── 07_System_Integration/
├── 08_Hardware_Accelerators/
├── 09_Advanced_Topics/
└── 10_References_and_Resources/
```

## How to Navigate

- Follow the folders in numeric order; each level builds on the last.
- Start every module by reading its top-level markdown files (for example, `00_Overview/intro_to_comparch.md` and `01_Digital_Logic_Fundamentals/session_2.md`).
- HDL code samples default to Verilog unless otherwise noted. Simulation tooling suggestions emphasise open-source flows (Verilator, Yosys, GHDL) with vendor-specific notes where helpful.
- Suggested lab ideas, experiments, and deep dives appear at the end of many sections for hands-on reinforcement.

## Local docs preview

To build and preview the documentation site locally:

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://127.0.0.1:4000/` in your browser.

