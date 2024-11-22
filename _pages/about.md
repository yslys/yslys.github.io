---
permalink: /
title: "Hi there, I'm Yusen!"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---
I am a second year CS graduate student at the University of Wisconsin-Madison, advised by Prof. Michael Swift.

My research interests include hardware-software co-design, Operating Systems and Computer Architecture.



# Research Project:
I designed an interface of near-memory accelerator that communicates with the host via PCIe. This is a software-hardware co-design, which allows me to touch all three layers of computer system: hardware level, kernel level and software level. 

<!-- I regard myself as a combination of three engineering teams. If you do not believe it, you can keep reading. Let's start with kernel level: -->
<!-- 
## Kernel Level
* The interface is a portable Linux kernel module that can be easily adjusted to support any PCIe device
* To verify the correctness of the interface, I simulated an accelerator in QEMU. After a series of testing and debugging, the interface functions well.
* With the interface being done, the next question is - how to evaluate its performance? QEMU is a functional simulator, not providing us with timing information. So, gem5 is the best candidate.

## Hardware level
* I started simulating the accelerator in gem5.
  * Before doing that, I kept in mind what workloads I would like to optimize - memory-intensive workloads, of which Star-Schema Benchmark is a good candidate.
  * I borrowed some implementation in gem5-GPU to setup the skeleton of the accelerator and IOMMU.
* Database accelerator simulation
  * I borrowed the design of Intel's Data Streaming Accelerator.
  * For the operations the accelerator support, I picked selection, projection and aggregation.
* Core design question: How to simulate the pipelined behavior of the hardware?
  * In QEMU, since the aim was to verify the correctness of the interface, I broke the execution of work into 3 stages: copy data to accelerator, process the data, then write the result back. But this is not what real hardware would do.
  * In gem5, I started thinking about how to simulate the hardware execution pipeline.
  * At what granularity can I abstract the execution pipeline?
    * Instruction level? No, since that would mean implementing a PCIe device along with some CPU properties, time-consuming. 
    * What I pick: functional level simulation. I broke each task into small units. For instance, if the task is: do a selection on data of length 100 where value of the data is less than x. I broke that up into smaller pieces named "query unit"s. Each query unit represents the smallest execution unit which a hardware functional unit can handle. Suppose we set the granularity to be 20, then there will be 5 query units which will be executing in a pipelined manner.
    * Each query unit has the same execution steps/stages, including 1. obtain virtual address of data 2. do address translation, 3. read data, 4. process data, 5. write data back, 6. ring the doorbell. In this way, different query units will be in its own stage, then transit to the next stage if there is available hardware resource. This is exactly the same as the hardware pipeline (fetch, decode, execute, memory access, writeback)!
    * Why my design is good? Because it allows us to tweak the different parameters to test different hardware configurations easily! I can manually set the process data stage latency to be 100 cycles, or 200 cycles, depending on what operation we are simulating. 
  * Correctness?
    * This is a good question regarding my design - how to prove that the latency we obtained make sense?
    * gem5 has a cool function called `schedule(event, latency)` i.e. scheduling an `event` `latency` ticks later. On completion of any event, there will be a callback function being invoked. I implemented a centralized scheduler which is being invoked on completion of any event, and that scheduler will always scan through all query units and see if they can transit to the next stage, with available hardware resource. So my design is guaranteed to be correct, and accurate, as long as the latencies are reasonable.
* The reason I need to simulate the accelerator is because I need to evaluate my interface, but as you can see, the simulation piece can already be a huge amount of work.

## User level
* What's next? Workloads implementation
  * Now that we have the simulated accelerator and the interface, we need to run some workloads!
  * One thing I appreciate the modular design is that we can simply modify one layer without needing to modify other layers to get it work. However, in this work, it is not as simple as it is.
  * To verify the correctness of both the accelerator and interface, I already wrote different microbenchmarks, but that is not enough. Evaluation requires real-world workloads!
  * I need to go through the source code of the workload, understand it and figure out where to add the interface.
  * Of course, with endless debugging due to cross-layer design.

## What this experience brings me?
* Systems design skills:
  * From the naive design to the pipelined design.
  * Cooperating all three layers to make sure they work properly.
  * Unit tests all the time - make sure the test covers as many edge cases as possible.
* Strong systems programming skills:
  * I wrote a lot of code.
  * I deleted a lot of code (due to false design).
  * There is still a lot of code left.
  * I don't remember how many lines of C and C++ code I have written :).
* Strong debugging skills.
  * High cost of making a mistake - each run takes around 10 minutes on microbenchmark, 1 hour for real-world workload. I need to find a fast way to locate the bug with a minimal reproducible example.
  * Debugging tools? I cannot run GDB in gem5's emulated environment since it is too slow.
  * How did I debug? I can only use `printf()` and analyze the trace, due to the concurrent execution of the query units.
* The ability to work with large code bases including Linux, and gem5.
 -->
