---
layout: page
title: Process Scheduling and Insights with BPFTRACE
description: A mini-project exploring Linux process scheduling, BCC, and BPFTRACE visualizations.
img: assets/img/Architecture/architecture_bg.jpg
importance: 2
category: work
---

### Project Description

In Operating Systems, the Scheduler refers to the program responsible for the removal of the currently executing process, selection of the next process, and the context switch involved in changing the states of these processes. The main goals of a scheduler are to ensure fast process response time, good throughput for background processes, and the avoidance of process starvation. 

For my mini-project in M.Sc Mathematics, guided by Shaju Abraham, Naveen Miriyalu, Dibyam Pradhan, and Dr. Raghunatha Sarma, I conducted a deep dive into Linux Schedulers. Modern OSes segregate virtual memory into user space and kernel space to provide memory protection. In Linux, processes are represented and stored using the `task_struct` structure, also known as the Process Descriptor.

To analyze process scheduling, I utilized **BCC** and **bpftrace**. BCC is a toolkit for creating efficient kernel tracing, making BPF programs easier to write with kernel instrumentation in C. **bpftrace** is a high-level tracing language for Linux enhanced Berkeley Packet Filter (eBPF). 

During the project, several experiments were conducted:
- Visualizing the effects of a CPU-bound operation on the frequency and utilization of the core by pinning a task to a single core.
- Generating CPU-bound workloads and pinning them to a single core to see the effects on the scheduling of the processes (analyzing `offcputimes` and `offwaketimes`).
- Generating IO workloads with FIO (Flexible I/O tester) and generating stack traces for `offcputime`, polled runqueue latency, and time spent on soft IRQs.

I also generated **Flame graphs**—a visualization of hierarchical data created to visualize stack traces of profiled software. In our case, the flame graph of the stack trace showed that `fallocate64`, which is used to allocate memory on the fat file system, is called far too many times. Such operations, if not optimized, can be a potential bottleneck due to the inherently high latency of memory operations.

### Native Linux Profiling and BPF Tracing

Before the widespread adoption of eBPF, developers and administrators relied on a variety of built-in Linux tools to profile and trace system performance:
- **`perf`**: The "Swiss army knife" of Linux profiling, used to measure CPU cycles, cache misses, and identify function hotspots.
- **`strace` & `ltrace`**: Used to trace system calls and library calls made by a process, highly effective for debugging I/O or network bottlenecks.
- **`top` / `htop` / `pidstat`**: Real-time monitors that reveal CPU, memory, and I/O utilization per process.
- **`valgrind`**: A powerful suite (including tools like `memcheck` and `cachegrind`) for deep memory debugging and cache profiling, though it incurs significant performance overhead.

However, the introduction of eBPF revolutionized tracing by allowing programmatic, low-overhead instrumentation directly within the kernel. BCC (BPF Compiler Collection) provides a vast suite of pre-built tools tailored for almost every subsystem of the Linux OS.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Architecture/bcc_tracing_tools.png" title="Linux bcc/BPF Tracing Tools" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A map of Linux bcc/BPF Tracing Tools demonstrating the sheer breadth of visibility eBPF provides. Notice how tools like <code>runqlat</code> and <code>offcputime</code> sit squarely in the Scheduler block, which were crucial for the analysis in this project.
</div>

Below you can view the complete presentation report for more details.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <!-- PDF Viewer using an iframe -->
        <iframe src="/assets/pdf/Schedulers_and_BPFTRACE_final.pdf" width="100%" height="600px" style="border: 1px solid #ddd; border-radius: 8px;">
            This browser does not support PDFs. Please download the PDF to view it: <a href="/assets/pdf/Schedulers_and_BPFTRACE_final.pdf">Download PDF</a>.
        </iframe>
    </div>
</div>

<div class="caption">
    The complete mini-project presentation: "Process Scheduling and Insights with BPFTRACE".
</div>