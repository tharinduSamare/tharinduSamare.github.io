---
title: HeiChips26 Hackathon (RISCV-CPU)
publishDate: 2026-10-02 00:00:00
img: /assets/images/HeiChips26/RISCV_custom_core_architecture.png
img_alt: HeiChips26_SOC_architecture
description: |
  Task specific RISC-V CPU to drive MMIO Accelerator
start_date: "2026/08"
end_date: "2026/09"
tags:
  - FPGA
  - Area Optimization
  - Open-source
  - SystemVerilog
---

## How Tiny Can a Task-Specific RISC-V Core Be?

What about **246 LUT4s out of just 288 available?** 🚀

My team [Shangeeth](https://www.linkedin.com/in/shangeeth-g-r-005414181/), [Udaya Subedi](https://www.linkedin.com/in/udaya-init/), [Udara Mendis](https://www.linkedin.com/in/udara-mendis/) and I participated in the **HeiChips26 Summer School & Hackathon**, where we designed an ASIC-based **DNA alignment MMIO accelerator** using the Smith-Waterman algorithm. To control the accelerator, we also developed our own task-specific RISC-V CPU.

The challenge: the chip has a tiny eFPGA with only **288 LUT4s**, so our CPU had to fit within this extremely tight resource budget.

We first evaluated two well-known open-source RISC-V cores:

* 🔹 **PicoRV32:** 2,247 LUT4s → 780% utilization
* 🔹 **SERV:** 2,770 LUT4s → 961% utilization

Both were far beyond our available resources.

So, we designed our own minimal RISC-V core specifically for our application.

After a lot of optimization and brainstorming, we achieved:

* ✅ **246 LUT4s → 85% utilization**
* ✅ ~**9× smaller than PicoRV32**
* ✅ ~**11× smaller than SERV**

### Key Design Decisions


1. **Register file with only 3 registers**

   * **X0** – Hardwired to zero.
   * **X1 [8:0]** – Holds data to/from the accelerator and memory.
   * **X2 [9:0]** – Holds the address of the accelerator or memory location.

2. **Simplified ALU with only 3 operations**

   * **AND** – Used to isolate status register bits.
   * **OR**
   * **ADD** – Used mainly for address calculations.

3. **BEQZ** – A simplified branch operation that branches when the ALU output is zero.

4. **10-bit memory address width**

   * The available on-chip SRAM is only **4 KiB (1024 × 32-bit)**, so there was no need for a larger address space.

5. **Multi-cycle implementation with 6 states**

   * Fetch
   * Fetch_wait
   * Execute
   * Mem_wr
   * Mem_rd
   * WB

6. **Von Neumann architecture**

   * Instructions and data share the same memory.

We then benchmarked our core against PicoRV32. Interestingly, our much smaller CPU could drive the accelerator **1.99× faster** for our workload.

This optimization comes with trade-offs. Because our processor has only two writable registers besides X0, it is not capable of handling conventional `for` loops or `while` loops efficiently. Therefore, the loops in our application had to be flattened into individual instructions. As a result, the assembly program takes considerably more memory space. For example, processing **20 DNA sequence pairs requires around 400 instruction and data memory words**.

We also cannot use the standard **RISC-V toolchain** to directly compile C code into machine code for our processor because of its limited instruction set and register file. We therefore had to manually write the assembly program.

However, for our goal—**driving the accelerator at its highest performance while fitting into 288 LUT4s**—it was sufficient.

We thoroughly simulated the design and emulated the complete system on an **Artix-7 FPGA** to validate it before tapeout.

This project taught us an important lesson:

> When resources are extremely constrained, sometimes the best solution is not to optimize an existing design, but to rethink the architecture from the ground up.

A huge thank you to **[Prof. Dirk Koch](https://www.linkedin.com/in/dirk-koch-773b258/), [Leo Moser](https://www.linkedin.com/in/leo-moser/), and the entire HeiChips26 team** for organizing the Summer School and Hackathon and giving us the opportunity to work toward our own real ASIC!

Special thanks to **[MSc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/) and [MSc. Lucas Deutschmann](https://www.linkedin.com/in/lucas-deutschmann-671530181/)** from **[RPTU Kaiserslautern-Landau](https://rptu.de/)** for their valuable ideas, feedback, and guidance throughout the hackathon.

🔗 **Complete design and implementation:**
https://github.com/tharinduSamare/heichips26-DNA_sequencer


##### Custom RISCV CPU Architecture
![HeiChips26 RISCV Core Architecture](/assets/images/HeiChips26/RISCV_custom_core_architecture.png)

##### SOC Architecture
![HeiChips26 SOC Architecture](/assets/images/HeiChips26/DNA_Sequence_Align_SOC_block_diagram.png)

##### eFPGA Resource Utilization
![eFPGA resource utilization](/assets/images/HeiChips26/eFPGA_resource_utilization.png)
