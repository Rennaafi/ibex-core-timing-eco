# Third-Party Notices

This repository is a personal portfolio project built on top of, and reporting
results from, several third-party open-source tools and an open-source CPU
design. Only the analysis, documentation, curated report excerpts, and
write-up in this repository are original — everything below is credited to
its respective upstream project.

## Toolchain

- **[OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD)** and
  **[OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)**
  — BSD 3-Clause License, Copyright (c) 2018-2023, The Regents of the
  University of California. Used here as the physical design flow
  (floorplan, placement, CTS, routing, STA, and the interactive ECO tools
  `replace_cell`, `repair_timing`, `detailed_placement`, `global_route`,
  `detailed_route`), and as the source of the `ibex` example design's
  default configuration and shipped SDC.
- **[Yosys](https://github.com/YosysHQ/yosys)** — ISC License. Used for RTL
  synthesis (via its built-in `slang` SystemVerilog frontend).

## RTL

The `ibex_core` design synthesized in this project is
**[lowRISC Ibex](https://github.com/lowRISC/ibex)**, a production RV32IMC
RISC-V core — Apache License 2.0, © lowRISC contributors and Ibex
contributors. The RTL source is **not vendored into this repository**; it is
consumed via OpenROAD-flow-scripts' bundled example-design copy
(`flow/designs/src/ibex_sv/`) at build time. Only this project's own ORFS
configuration (`config/config.mk`), the shipped SDC constraint, and curated
timing/power reports and layout renders are included here.

## Process Design Kit (PDK)

- **Nangate45 / FreePDK45** — an open academic standard-cell library
  originally from Nangate / NCSU, bundled with OpenROAD-flow-scripts for
  teaching and research use. Not intended for commercial tapeout.

## What This Means For You

If you reuse this repository:
- The README, documentation, and curated `.rpt`/`.json`/`.md` report
  excerpts are yours to reuse under the [MIT License](LICENSE) in this repo.
- If you reuse `config/config.mk` or the constraint file, retain attribution
  to OpenROAD-flow-scripts and comply with its BSD 3-Clause terms.
- If you want the actual `ibex_core` RTL, get it from
  [lowRISC/ibex](https://github.com/lowRISC/ibex) directly and comply with
  its Apache License 2.0 (including the NOTICE requirements of that
  license).
