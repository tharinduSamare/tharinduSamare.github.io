---
title: HeiChips26 Hackathon (ASIC)
publishDate: 2026-10-01 00:00:00
img: /assets/images/HeiChips26/DNA_Sequence_Align_SOC_block_diagram.png
img_alt: HeiChips26_SOC_architecture
description: |
  ASIC accelerator for DNA Sequence Alignment
start_date: "2026/08"
end_date: "2026/09"
tags:
  - ASIC
  - Librelane
  - Open-source
  - VLSI
  - SystemVerilog
  - HeiChips26
---

## HeiChips26 DNA Sequence Alignment ASIC Accelerator

Having my own ASIC is a dream of every digital design enthusiast. With the **HeiChips26 Summer School and Hackathon**, we are about to turn that dream into reality! 🚀

Together with my teammates [Shangeeth](https://www.linkedin.com/in/shangeeth-g-r-005414181/), [Udaya Subedi](https://www.linkedin.com/in/udaya-init/), [Udara Mendis](https://www.linkedin.com/in/udara-mendis/) and I, we designed an **ASIC accelerator for DNA sequence alignment based on the Smith-Waterman algorithm**. Our design will be part of the **HeiChips26 tapeout**, and, if all goes as planned, we hope to hold the first fabricated chip in our hands next year (2027)! 🤞

The main part of our design is a dedicated accelerator synthesized for the **IHP SG13CMOS5L** technology. It runs at **100 MHz** and fits into a **500 µm × 200 µm** tile.

### Accelerator Architecture

The accelerator consists of:

* An **8-PE systolic array** for DNA character alignment
* FIFOs for incoming DNA sequences and alignment results
* A **fully pipelined datapath**
* Support for processing up to **16 sequence pairs** at once
* An **MMIO interface**, allowing it to be integrated with a generic CPU

For the complete demonstration system, we also designed a tiny task-specific **RISC-V core** and memory controller that fit into **288 LUT4s** of the eFPGA section of the tapeout chip.

### Design & Verification

The complete design and verification flow was based on the **LibreLane open-source digital ASIC design flow**.

We verified the design through simulation and also emulated it on a **Nexys A7 FPGA board**.

We also benchmarked our accelerator-based system against the open-source **PicoRV32** core and a fully software-based implementation.

In our benchmark, the combination of our **accelerator + custom RISC-V core achieved a 219.4× speedup** compared with the software-based implementation.

A huge thank you to **[Prof. Dirk Koch](https://www.linkedin.com/in/dirk-koch-773b258/), [Leo Moser](https://www.linkedin.com/in/leo-moser/), and the entire HeiChips26 team** for organizing the Summer School and Hackathon and for giving us the opportunity to work towards our own real ASIC!

Special thanks to **[MSc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/) and [MSc. Lucas Deutschmann](https://www.linkedin.com/in/lucas-deutschmann-671530181/)** from the Department of Electrical and Computer Engineering for their valuable ideas, feedback, and guidance throughout the hackathon. 🙏

Feel free to check out our complete design and implementation here:

🔗 https://github.com/tharinduSamare/heichips26-DNA_sequencer


##### SOC Architecture
![HeiChips26 SOC Architecture](/assets/images/HeiChips26/DNA_Sequence_Align_SOC_block_diagram.png)

##### Layout Design
![Layout design](/assets/images/HeiChips26/heichips26_dna_sequencer.png)

##### Benchmarking
![Benchmarking](/assets/images/HeiChips26/Benchmarking.png)

##### ASIC Resource Utilization
![ASIC resource utilization](/assets/images/HeiChips26/Asic_utilization.png)
