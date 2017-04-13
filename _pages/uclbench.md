---
ID: 159
post_title: uCLbench
author: Vincent Hindriksen
post_date: 2017-04-13 22:33:13
post_excerpt: ""
layout: page
permalink: http://opencl.org/tools/uclbench/
published: true
hefo_before:
  - "0"
hefo_after:
  - "0"
---
To characterize and compare the OpenCL performance of existing and future devices uCLbench provides the following types of micro-benchmarks:
<ul>
 	<li>Arithmetic Throughput: Parallel and sequential throughput for all basic mathematical operations, and many built-in functions defined by the OpenCL standard. When available, native implementations (with reduced accuracy) are also measured.</li>
 	<li>Memory Subsystem: Host to device, device to device and device to host copying bandwidth. Streaming bandwidth for on-device address spaces. Latency for memory accesses to global, local and constant address spaces. Also determines existence and size of caches.</li>
 	<li>Branching Penalty: Impact of divergent dynamic branching on device performance, particularly pronounced on GPUs.</li>
 	<li>Runtime Overheads: Kernel compilation time and queuing delays incurred when invoking kernels of various code volume.</li>
</ul>
&nbsp;
<ul>
<ul>
 	<li>Webpage: <a href="http://www.dps.uibk.ac.at/insieme/uclbench.html">http://www.dps.uibk.ac.at/insieme/uclbench.html</a></li>
Project page:</ul>
</ul>
<a href="https://github.com/PeterTh/uCLbench">https://github.com/PeterTh/uCLbench</a>