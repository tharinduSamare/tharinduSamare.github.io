---
title: InvisiBOOM RISCV CPU
publishDate: 2026-08-22 00:00:00
img: /assets/images/invisiBOOM/InvisiBOOM_implementation.png
img_alt: Invisi_BOOM LSU architecture
description: |
  Making Speculative Execution Invisible in the Cache Hierarchy - Implemented on the BOOM RISC-V core
start_date: "2025/06"
end_date: "2026/06"
tags:
  - RISCV
  - BOOM
  - Spectre Attacks
  - Hardware Security
  - Chisel
  - SystemVerilog
---

Spectre attacks are hardware side-channel attacks that exploit speculative execution and are particularly challenging to mitigate without introducing significant performance overhead.

[InvisiSpec](https://ieeexplore.ieee.org/document/8574559) is one proposed solution, introduced by Hennessy et al. at MICRO 2018. Instead of allowing speculative loads to directly affect the cache hierarchy, InvisiSpec keeps the data fetched by speculative loads in a buffer that remains invisible to the cache hierarchy until the corresponding instructions become non-speculative. The CPU can use the data directly from the buffer. This approach can provide protection against speculative-execution side channels with relatively low performance overhead. Under the supervision of [M.Sc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/), I implemented a simplified version of InvisiSpec on the [Berkeley Out-of-Order RISC-V Processor](https://boom-core.org/) (BOOM).

The result is **InvisiBOOM**.

**To the best of my knowledge, InvisiBOOM is the first real implementation of InvisiSpec on a real-world, widely used open-source out-of-order processor.** InvisiBOOM successfully runs the complete RISC-V test suite as well as randomized RISC-V torture tests, providing functional verification of the modified processor.

The main modifications were made to the Load/Store Unit (LSU). The original InvisiSpec design introduces speculative buffers at multiple levels of the cache hierarchy. However, BOOM uses the TileLink cache-coherence protocol to communicate between cache levels, making modifications across the entire cache hierarchy considerably more complex. Therefore, in InvisiBOOM, I implemented the speculative buffer at the L1 cache level. If a speculative load miss L1 cache, it waits until the instruction becomes non-speculative and retry again.

Despite this simplification, the design can still achieve performance improvements, particularly in workloads with loops.

This project gave me valuable hands-on experience with out-of-order execution, speculative execution, cache hierarchies, hardware security, and RTL-level processor design.

I would like to sincerely thank [M.Sc. Tobias Jauch](https://www.linkedin.com/in/tobias-jauch-a4b063185/) for supervising and supporting me throughout this project, and [Prof. Dr.-Ing. Wolfgang Kunz](https://www.linkedin.com/in/wolfgang-kunz-29141918/) for teaching us and for his continuous support and inspiration throughout my master's studies.

If you are interested in the design and implementation, you can check out the project here: [link](https://github.com/RPTU-EIS/InvisiBOOM)

![Original BOOM implementation](/assets/images/invisiBOOM/BOOM_implementation.png)
![Invisi BOOM implementation](/assets/images/invisiBOOM/InvisiBOOM_implementation.png)