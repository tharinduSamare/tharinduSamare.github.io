---
title: RISCV32I Pipelined Processor + UVM testbench 
publishDate: 2026-08-21 00:00:00
img: /assets/images/rv32i_core_uvm_tb/processor_architecture.png
img_alt: RV32I pipeline processor architecture
description: |
  RV32I processor with UVM testbench to support running (simplified) riscv-tests
tags:
  - SystemVerilog
  - Chisel
  - RISCV-32I
  - UVM
---

As an undergraduate, I loved to design RISC-V processors and deploy them on FPGAs. Now, as a master's student, I built another **RISCV-32I 5-stage pipelined core**, but this time using Chisel HDL. I also went beyond the RTL design and built a **complete UVM-based verification environment** as well.

I was able to verify the functionality of the core using the [riscv-tests](https://github.com/riscv-software-src/riscv-tests). 

I started this work as the class project of "Architecture of Digital Systems I (ADS1)" module at [RPTU](https://rptu.de/). Initially it was a simplified processor supporting only arithmetic and logic operations. Since I was passionate about the project, I continued working on it after completing the module, adding the remaining control-flow instructions and pipeline hazard handling to turn it into a complete RV32I 5-stage pipelined processor. 

Following suggestions from [M.Sc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/), I also developed a complete UVM-based testbench to verify the processor.

Now this processor design and its UVM testbench are being used as the class project for ADS1 module at [RPTU](https://rptu.de/). It is great to see my work being used to give students hands on experience with RISC-V architecture and UVM verification.

I would like to thank [M.Sc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/) for supervising and supporting me throughout the project, and [Prof. Dr.-Ing. Wolfgang Kunz](https://www.linkedin.com/in/wolfgang-kunz-29141918/) for teaching us and for his continuous support and inspiration throughout my master's studies.

If you are interested in the design, you can check out the project here: [link](https://github.com/tharinduSamare/RV32_processor)

![UVM testbench](/assets/images/rv32i_core_uvm_tb/rv32core_uvm_tb.png)