---
ID: 194
post_title: Texas Instruments
author: Vincent Hindriksen
post_date: 2017-05-02 07:22:43
post_excerpt: ""
layout: page
permalink: >
  http://opencl.org/hardware/texas-instruments/
published: true
hefo_before:
  - "1"
hefo_after:
  - "1"
---
<img class="alignright size-medium wp-image-9817" src="https://streamhpc.com/wp-content/uploads/2015/09/Texas-Instruments-logo-design-300x111.png" alt="Texas-Instruments-logo-design" width="300" height="111" />TI has a fully conformant OpenCL 1.1 implementation.

The below table is taken from <a href="http://downloads.ti.com/mctools/esd/docs/opencl/intro.html">http://downloads.ti.com/mctools/esd/docs/opencl/intro.html</a> and shows which DSPs have OpenCL-support.
<table class="docutils" border="1">
<thead valign="bottom">
<tr class="row-odd">
<th class="head">SoC</th>
<th class="head">System</th>
<th class="head">Khronos Conformance</th>
<th class="head">Installation Instructions</th>
</tr>
</thead>
<tbody valign="top">
<tr class="row-even">
<td><a class="reference external" href="http://www.ti.com/product/AM5728">AM572</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/tmdsevm572x">AM572 EVM</a></td>
<td>OpenCL v1.1 Conformant</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-am57x">Processor SDK for AM57x</a></td>
</tr>
<tr class="row-odd">
<td><a class="reference external" href="http://www.ti.com/product/dra756">DRA75x</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/j6evm5777">DRA75x EVM</a></td>
<td>OpenCL v1.1 Conformant</td>
<td><a class="reference external" href="http://software-dl.ti.com/infotainment/esd/jacinto6/processor-sdk-linux-automotive/latest/index_FDS.html">Processor SDK for DRA7x</a> (<a class="reference external" href="http://processors.wiki.ti.com/index.php/Processor_SDK_Linux_Automotive_Software_Developers_Guide#Testing_OpenCL">Enabling OpenCL on DRA75x</a>)</td>
</tr>
<tr class="row-even">
<td><a class="reference external" href="http://www.ti.com/product/AM5718">AM571</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/tmdsevm572x">AM572 EVM</a></td>
<td>OpenCL v1.1 Conformant</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-am57x">Processor SDK for AM57x</a></td>
</tr>
<tr class="row-odd">
<td><a class="reference external" href="http://www.ti.com/product/66ak2h14">66AK2H</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/EVMK2H">66AK2H EVM</a></td>
<td>OpenCL v1.1 Conformant</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-k2h">Processor SDK for K2H</a></td>
</tr>
<tr class="row-even">
<td><a class="reference external" href="http://www.ti.com/product/66ak2l06">66AK2L</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/XEVMK2LX">66AK2L EVM</a></td>
<td>Not submitted for conformance</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-k2l">Processor SDK for K2L</a></td>
</tr>
<tr class="row-odd">
<td><a class="reference external" href="http://www.ti.com/product/66ak2e05">66AK2E</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/XEVMK2EX">66AK2E EVM</a></td>
<td>Not submitted for conformance</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-k2e">Processor SDK for K2E</a></td>
</tr>
<tr class="row-even">
<td><a class="reference external" href="http://www.ti.com/product/66ak2g02">66AK2G</a></td>
<td><a class="reference external" href="http://www.ti.com/tool/EVMK2G">66AK2G EVM</a></td>
<td>Not submitted for conformance</td>
<td><a class="reference external" href="http://www.ti.com/tool/processor-sdk-k2g">Processor SDK for K2G</a></td>
</tr>
</tbody>
</table>
<h1>Theoretical Performance of the C66x</h1>
<ul>
 	<li>Fixed point 16x16 MACs per cycle: 32</li>
 	<li>Fixed point 32x32 MACs per cycle: 8</li>
 	<li>Floating point single precision MACs per cycle: 8</li>
 	<li>Arithmetic floating point operations per cycle: 16 2-way SIMD on .L and .S units (e.g. 8 SP operations for A and B) and 4 SP multiply on one .M unit (e.g 8 SP operations for A and B)</li>
 	<li>Arithmetic floating point operations per cycle: 164 2-way SIMD on .L and .S units (e.g. 8 SP operations for A and B) and 4 SP multiply on one .M unit (e.g 8 SP operations for A and B)</li>
 	<li>Load/store width 2 x 64-bit 2 x 64-bit Vector size (SIMD capability): 128-bit (4 x 32-bit, 4 x 16-bit, 4x-8bits)</li>
</ul>
<h2>GFLOPs</h2>
2 FLOPs - 2-way SIMD on .L1 (A side) such as DADDSP or DSUBSP
2 FLOPs - 2-way SIMD on .L2 (B side) such as DADDSP or DSUBSP
2 FLOPs - 2-way SIMD on .S1 (A side) such as DADDSP or DSUBSP
2 FLOPs - 2-way SIMD on .S2 (B side) such as DADDSP or DSUBSP
4 FLOPs - 4-way SIMD on .M1 (A side) such as QMPYSP (or CMPYSP, maybe not 4-way SIMD)
4 FLOPs - 4-way SIMD on .M2 (B side) such as QMPYSP (or CMPYSP, maybe not 4-way SIMD)
========================
16 FLOPs total per cycle per C66x CorePac (<a href="https://e2e.ti.com/support/dsp/c6000_multi-core_dsps/f/639/t/145286">source</a>)
<h1>Boards</h1>
A good starter board is the <a href="http://beagleboard.org/x15">BeagleBoard X-15</a>, and has <a href="http://www.elinux.org/Beagleboard:BeagleBoard-X15#Will_I_be_able_to_program_the_C66x_DSPs.3F">OpenCL drivers</a>. It has 2x C66X DSPs and 2x 1.5-GHz ARM Cortex-A15.

<img class="size-large wp-image-9816 alignnone" src="https://streamhpc.com/wp-content/uploads/2015/09/X15_TOP_SIDE-550x568.jpg" alt="X15_TOP_SIDE" width="550" height="568" />