---
title: "Parallel Computing 1: Modern Processor Architecture"
cover: /img/work.png
mathjax: true
categories: HPC
tags: 
    - Parallel Computing
    - SIMD
    - Multi-threading
    - Multi-core
    - Superscalar
abbrlink: 3428437
date: 2025-02-23 00:00:00
updated: 2025-02-27 00:00:00
---

This article aims to discuss some fundamental methods to increase the program speed, from the perspective of modern processor.

This is also my study memo for [Stanford CS149](https://gfxcourses.stanford.edu/cs149/fall24/lecture/)

## Review

1. What is a **Program**
> * A stream of hardware instructions to be exectuted

2. What is an **Execution Context** (or State)
> * A snapshot of register files, containing ordinary registers and **PC**.

3. What basic units does a processor have, and what is the basic flow of executing an instruction?
> * A simple CPU contains a fetch/decode unit, execution unit (ALU), and a register file (an execution context)
> * The picture here omits Load/Store unit for simplicity

![A single cycle CPU view](/img/pc/single-cycle-cpu-flow.png)



## Key Technology

### Superscalar

* The hardware detects **independent instructions** from **a single instruction stream** and executes them in the same cycle
* Out of Order Execution
* The CPU in the picture executes 2 instruction per cycle (**two-way superscalar**)
* Provides Instruction Level Parallelism (**ILP**)

![Superscalar arch](/img/pc/superscalar.png)

### Multi-core
* The out-of-order-execution hardware is complicated and takes more transistors and space to make
* Allow software multi-threading (e.g. Linux pthread)
* Provides Thread Level Parallelism (**TLP**)

![Two cores CPU](/img/pc/multicore.png)

### SIMD (Single Instruction Multiple Data)

* If different cores (threads) are executing the same sequence of instructions but on different data, then no need to duplicate all processor units.
* Replicating ALUs is cheap
* Vector Instruction are supported on Intel CPUs (e.g. `vmulps` ,`vloadps`)

![SIMD pic](/img/pc/simd.png)

### Hardware Threading

* Some instructions takes long time to complete, like memory load store which produces long **memory stalls**.
* The idea here is to change the instruction stream to execute, or **change execution context**
* Interleave processing of multiple threads on the same core to hide stalls

![Hardware threading to hide memory stalls](/img/pc/hardware-threading.png)


## Evaluation Metrics

### Latency

How fast (or how long) does an instruction to complete

* Increase clock rate

### Throughtput/Bandwidth

How many instructions are completed per unit of time

* Pipelined CPU
* Superscalar

We will focus on this in the rest of the note

## Compute Bound v.s. Memory Bound

* Modern program must access memory to fully utilize modern processor
* Overcoming bandwidth limits is often the most important
challenge facing software developers targeting modern
throughput-optimized systems