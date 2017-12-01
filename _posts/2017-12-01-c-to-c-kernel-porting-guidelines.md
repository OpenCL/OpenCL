---
ID: 271
post_title: C to C++ kernel porting guidelines
author: Vincent Hindriksen
post_excerpt: ""
layout: post
permalink: >
  http://opencl.org/2017/12/c-to-c-kernel-porting-guidelines/
published: true
post_date: 2017-12-01 12:14:35
---
This document is a set of guidelines for developers who know OpenCL C and plan to port their kernels to OpenCL C++, and therefore they need to know the main differences between those two kernel languages. The focus is not on highlighting all the differences, but rather on exposing and explaining those that are the most important, and those that may cause hard-to-detect bugs when porting from OpenCL C to OpenCL C++.

This text was initially published at <a href="https://github.com/OpenCL/OpenCLCXXPortingGuidelines/blob/master/OpenCLCToOpenCLCppPortingGuidelines.md">https://github.com/OpenCL/OpenCLCXXPortingGuidelines/blob/master/OpenCLCToOpenCLCppPortingGuidelines.md</a>

Comments, suggestions for improvements, and contributions are most welcome.<!--more-->

<strong><a href="#S-Differences">Differences</a></strong>:
<ul>
 	<li><a href="#S-OpenCLCXX">OpenCL C++ Programming Language</a>:
<ul>
 	<li><a href="#S-OpenCLCXX-VectorLiterals">OpenCL C Vector Literals</a></li>
 	<li><a href="#S-OpenCLCXX-BoolNType"><code>bool<i>N</i></code> Type</a></li>
 	<li><a href="#S-OpenCLCXX-EndOfExplicitNamedAddressSpaces">End Of Explicit Named Address Spaces</a></li>
 	<li><a href="#S-OpenCLCXX-KernelRestrictions">Kernel Function Restrictions</a></li>
 	<li><a href="#S-OpenCLCXX-KernelParamsRestrictions">Kernel Parameter Restrictions</a></li>
 	<li><a href="#S-OpenCLCXX-GeneralRestrictions">General Restrictions</a></li>
</ul>
</li>
 	<li><a href="#S-OpenCLCXXSTL">OpenCL C++ Standard Library</a>:
<ul>
 	<li><a href="#S-OpenCLCXXSTL-NamespaceCL">Namespace cl::</a></li>
 	<li><a href="#S-OpenCLCXXSTL-ConversionsLibrary">Conversions Library (<code>convert_*()</code>)</a></li>
 	<li><a href="#S-OpenCLCXXSTL-ReinterpretingDataLibrary">Reinterpreting Data Library (<code>as_<i>type</i>()</code>)</a></li>
 	<li><a href="#S-OpenCLCXXSTL-AddressSpacesLibrary">Address Spaces Library</a></li>
 	<li><a href="#S-OpenCLCXXSTL-MarkerTypes">Marker Types</a></li>
 	<li><a href="#S-OpenCLCXXSTL-ImagesAndSamplersLibrary">Images and Samplers Library</a></li>
 	<li><a href="#S-OpenCLCXXSTL-PipesLibrary">Pipes Library</a></li>
 	<li><a href="#S-OpenCLCXXSTL-DeviceEnqueueLibrary">Device Enqueue Library</a></li>
 	<li><a href="#S-OpenCLCXXSTL-RelationalFunctions">Relational Functions</a></li>
 	<li><a href="#S-OpenCLCXXSTL-VectorDataLoadandStoreFunctions">Vector Data Load and Store Functions</a></li>
 	<li><a href="#S-OpenCLCXXSTL-AtomicOperationsLibrary">Atomic Operations Library</a></li>
</ul>
</li>
 	<li><a href="#S-OpenCLCXXCompilation">OpenCL C++ Compilation Process</a>:
<ul>
 	<li><a href="#S-OpenCLCXXCompilationToSPIRV">OpenCL C++ Compilation to SPIR-V</a></li>
 	<li><a href="#S-OpenCLCXXCompilationBuildSPIRV">Building program created from SPIR-V</a></li>
</ul>
</li>
</ul>
<strong><a href="#S-Bibliography">Bibliography</a></strong>
<h1><a id="user-content-differences" class="anchor" href="#differences" aria-hidden="true"></a><a name="user-content-S-Differences"></a>Differences</h1>
<h2><a id="user-content-opencl-c-programming-language" class="anchor" href="#opencl-c-programming-language" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX"></a>OpenCL C++ Programming Language</h2>
<h3><a id="user-content-opencl-c-vector-literals" class="anchor" href="#opencl-c-vector-literals" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-VectorLiterals"></a>OpenCL C Vector Literals</h3>
Vector literals, expression used for creating vectors from a list of scalars, vectors or a mixture thereof, known from OpenCL C are not part of the OpenCL C++ kernel language.

In OpenCL C++ vector types can be initialized like any other class - using constructors. For example, the following are available for <code>float4</code>:
<div class="highlight highlight-source-c++">
<pre><span class="pl-en">float4</span>(<span class="pl-k">float</span>, <span class="pl-k">float</span>, <span class="pl-k">float</span>, <span class="pl-k">float</span>)
float4(float2, <span class="pl-k">float</span>, <span class="pl-k">float</span>)
float4(<span class="pl-k">float</span>, float2, <span class="pl-k">float</span>)
float4(<span class="pl-k">float</span>, <span class="pl-k">float</span>, float2)
float4(float2, float2)
float4(float3, <span class="pl-k">float</span>)
float4(<span class="pl-k">float</span>, float3)
float4(<span class="pl-k">float</span>)</pre>
</div>
<h5><a id="user-content-note" class="anchor" href="#note" aria-hidden="true"></a>Note</h5>
<blockquote>In OpenCL C++ vector literals are NOT evaluated as user might expect, unfortunately, they never cause compilation errors.</blockquote>
Vector literals in OpenCL C++ are not evaluated as user might expect. In OpenCL C++ expression <code>(int4)(1, 2, 3, 4)</code> is evaluated to <code>(int4)4</code>. This happens because of how comma operator works: every value enclosed in parentheses except for the last is discarded, and then scalar-to-vector conversion is used for <code>4</code>.

In certain situations vector literals in OpenCL C++ code can cause warnings during compilation, but they do not cause compilation errors.
<h4><a id="user-content-solution" class="anchor" href="#solution" aria-hidden="true"></a>Solution</h4>
Do not use vector literals. Replace them with vector constructors.
<h4><a id="user-content-examples-bad" class="anchor" href="#examples-bad" aria-hidden="true"></a>Examples, bad</h4>
<div class="highlight highlight-source-c++">
<pre>int4 i = (int4)(<span class="pl-c1">1</span>, <span class="pl-c1">2</span>, <span class="pl-c1">3</span>, <span class="pl-c1">4</span>);
<span class="pl-c">// This expression will be evaluated to (int4)4,</span>
<span class="pl-c">// and i will be (4, 4, 4, 4).</span>
<span class="pl-c">// In OpenCL C++ compiler (clang) provided by Khronos</span>
<span class="pl-c">// it causes 'expression result unused' warnings.</span>

int4 i = (int4)(cl::max(<span class="pl-c1">0</span>, <span class="pl-c1">1</span>), cl::max(<span class="pl-c1">0</span>, <span class="pl-c1">2</span>), cl::max(<span class="pl-c1">0</span>, <span class="pl-c1">3</span>), cl::max(<span class="pl-c1">0</span>, <span class="pl-c1">4</span>))
<span class="pl-c">// This expression will be evaluated to (int4)4,</span>
<span class="pl-c">// and i will be (4, 4, 4, 4).</span>
<span class="pl-c">// In OpenCL C++ compiler (clang) provided by Khronos</span>
<span class="pl-c">// it DOES NOT cause any warnings.</span></pre>
</div>
<h4><a id="user-content-examples-correct" class="anchor" href="#examples-correct" aria-hidden="true"></a>Examples, correct</h4>
<div class="highlight highlight-source-c++">
<pre>uint4 u = uint4(<span class="pl-c1">1</span>); <span class="pl-c">//  u will be (1, 1, 1, 1)</span>
int4  i = int4{-<span class="pl-c1">1</span>, -<span class="pl-c1">2</span>, <span class="pl-c1">3</span>, <span class="pl-c1">4</span>} <span class="pl-c">// i will be (-1, -2, 3, 4)</span>

<span class="pl-c">// in each case f will be (1.0f, 2.0f, 3.0f, 4.0f)</span>
float4 f = float4(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">3</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">4</span>.<span class="pl-c1">0f</span>);
float4 f = float4(float2(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>), float2(<span class="pl-c1">3</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">4</span>.<span class="pl-c1">0f</span>));
float4 f = float4(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, float2(<span class="pl-c1">2</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">3</span>.<span class="pl-c1">0f</span>), <span class="pl-c1">4</span>.<span class="pl-c1">0f</span>);</pre>
</div>
<h3><a id="user-content-booln-type" class="anchor" href="#booln-type" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-BoolNType"></a><code>bool<i>N</i></code> Type</h3>
OpenCL C++ introduces new built-in vector type: <code>boolN</code> (where <code>N</code> is 2, 3, 4, 8, or 16). This addition change resolves problem with using the relational (<code>&lt;</code>, <code>&gt;</code>, <code>&lt;=</code>, <code>&gt;=</code>, <code>==</code>, <code>!=</code>) and the logical operators (<code>!</code>, <code>&amp;&amp;</code>, <code>||</code>) with built-in vector types.

In OpenCL C for built-in vector types the relational and the logical operators return a vector signed integer type of the same size as the source operands. In OpenCL C++ it was simpliefied and those operators return <code>boolN</code> for vector types and <code>bool</code> for scalars.

The OpenCL C 2.0 Specification on the results of the relational operators:
<blockquote>The result is a scalar signed integer of type <code>int</code> if the source operands are scalar and a vector signed integer type of the same size as the source operands if the source operands are vector types. Vector source operands of type <code>charn</code> and <code>ucharn</code> return a <code>charn</code> result; vector source operands of type <code>shortn</code> and <code>ushortn</code> return a <code>shortn</code> result; vector source operands of type <code>intn</code>, <code>uintn</code> and <code>floatn</code> return an <code>intn</code> result; vector source operands of type <code>longn</code>, <code>ulongn</code> and <code>doublen</code> return a <code>longn</code> result.</blockquote>
<blockquote>For scalar types, the relational operators shall return <code>0</code> if the specified relation is <code>false</code> and <code>1</code> if the specified relation is <code>true</code>. For vector types, the relational operators shall return <code>0</code> if the specified relation is <code>false</code> and <code>–1</code> (i.e. all bits set) if the specified relation is <code>true</code>. The relational operators always return <code>0</code> if either argument is not a number (<code>NaN</code>).</blockquote>
Including <code>boolN</code> vector types in OpenCL C++ also caused changes in signatures and/or behavior of built-in relational functions like: <code>all()</code>, <code>any()</code> and <code>select()</code>. See <a href="#S-OpenCLCXXSTL-RelationalFunctions">Relational Functions</a> section for more details.
<h4><a id="user-content-examples" class="anchor" href="#examples" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>bool2 b = bool2(<span class="pl-c1">1</span> == <span class="pl-c1">0</span>); <span class="pl-c">// { false, false }</span>

<span class="pl-c">// In OpenCL C: int b = 2 &gt; 1, and b is 1</span>
<span class="pl-k">bool</span> b = <span class="pl-c1">2</span> &gt; <span class="pl-c1">1</span> <span class="pl-c">// true</span>

<span class="pl-c">// In OpenCL C: int b = 2 &gt; 1, and b is 0</span>
<span class="pl-k">bool</span> b = <span class="pl-c1">2</span> == <span class="pl-c1">1</span> <span class="pl-c">// false</span>

<span class="pl-c">// OpenCL C-related note:</span>
<span class="pl-c">// -1 for signed integer type means that all bits are set</span>

<span class="pl-c">// In OpenCL C: int2 b = (uint2)(0, 1) &gt; (uint2)(0, 0),</span>
<span class="pl-c">// and b is { 0, -1 }</span>
bool2 b = uint2(<span class="pl-c1">0</span>, <span class="pl-c1">1</span>) &gt; <span class="pl-en">uint2</span>(<span class="pl-c1">0</span>, <span class="pl-c1">0</span>); <span class="pl-c">// { false, true }</span>

<span class="pl-c">// In OpenCL C: long2 b = (ulong2)(0, 0) &gt; (ulong2)(0, 0),</span>
<span class="pl-c">// and b is { 0, 0 }</span>
bool2 b = ulong2(<span class="pl-c1">0</span>, <span class="pl-c1">0</span>) &gt; <span class="pl-en">ulong2</span>(<span class="pl-c1">0</span>, <span class="pl-c1">0</span>); <span class="pl-c">// { false, false }</span>

<span class="pl-c">// In OpenCL C: long2 b = (long2)(1, 1) &gt; (long2)(0, 0),</span>
<span class="pl-c">// and b is { -1, -1 }</span>
bool2 b = long2(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>) &gt; <span class="pl-en">long2</span>(<span class="pl-c1">0</span>, <span class="pl-c1">0</span>); <span class="pl-c">// { true, true }</span></pre>
</div>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span>  <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_relational<span class="pl-pds">&gt;</span></span>

<span class="pl-c">// In OpenCL C: int2 b = isnan((float2)(0.0f)),</span>
<span class="pl-c">// and b is { 0, 0 }</span>
bool2 b = isnan(float2(<span class="pl-c1">0</span>.<span class="pl-c1">0f</span>)) <span class="pl-c">// { false, false }</span>

<span class="pl-c">// In OpenCL C: long2 b = isfinite((double2)(0.0))</span>
<span class="pl-c">// and b is { -1, -1 }</span>
bool2 b = isfinite(double2(<span class="pl-c1">0.0</span>)) <span class="pl-c">// { true, true }</span></pre>
</div>
<h4><a id="user-content-opencl-c-specification-references" class="anchor" href="#opencl-c-specification-references" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#expressions" rel="nofollow">OpenCL C++ Programming Language: Expressions</a></li>
</ul>
<h3><a id="user-content-end-of-explicit-named-address-spaces" class="anchor" href="#end-of-explicit-named-address-spaces" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-EndOfExplicitNamedAddressSpaces"></a>End Of Explicit Named Address Spaces</h3>
<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces" rel="nofollow">OpenCL C++ 1.0 Specification in Address Spaces section</a> says:
<blockquote>The OpenCL C++ kernel language doesn’t introduce any explicit named address spaces, but they are implemented as part of the standard library described in Address Spaces Library section. There are 4 types of memory supported by all OpenCL devices: global, local, private and constant. The developers should be aware of them and know their limitations.</blockquote>
That means that instead of using keywords <code>global</code>, <code>constant</code>, <code>local</code>, and <code>private</code>, in order to explicitly specify address space for variable or pointer you have to use address space pointers and address space storage classes.
<h5><a id="user-content-note-1" class="anchor" href="#note-1" aria-hidden="true"></a>Note</h5>
<blockquote>Go to <a href="#S-OpenCLCXXSTL-AddressSpacesLibrary">Address Spaces Library</a> section of The Porting Guidelines to read more about address space pointers and address space storage classes.</blockquote>
It is still possible for OpenCL C++ compiler to deduce an address space based on the scope where an object is declared:
<ul>
 	<li>If a variable is declared in program scope, with <code>static</code> or <code>extern</code> specifier and the standard library storage class (see <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#explicit-address-space-storage-classes" rel="nofollow">Explicit address space storage classes</a> section) is not used, the variable is allocated in the global memory of a device.</li>
 	<li>If a variable is declared in function scope, without static specifier and the standard library storage class (see <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#explicit-address-space-storage-classes" rel="nofollow">Explicit address space storage classes</a> section) is not used, the variable is allocated in the private memory of a device.</li>
</ul>
<h4><a id="user-content-opencl-c-specification-references-1" class="anchor" href="#opencl-c-specification-references-1" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces" rel="nofollow">OpenCL C++ Programming Language: Address Spaces</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces-library" rel="nofollow">OpenCL C++ Standard Library: Address Spaces Library</a></li>
</ul>
<h4><a id="user-content-examples-bad-opencl-c-style" class="anchor" href="#examples-bad-opencl-c-style" aria-hidden="true"></a>Examples, bad (OpenCL C-style)</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// Compilation error, "global" address space is not defined</span>
<span class="pl-c">// in OpenCL C++ kernel language</span>
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(global <span class="pl-k">int</span> * input)
{
  <span class="pl-c">// Compilation error, "local" address space is not defined</span>
  <span class="pl-c">// in OpenCL C++ kernel language</span>
  local <span class="pl-k">int</span> array[<span class="pl-c1">256</span>];
  <span class="pl-c">// ...</span>
}

<span class="pl-c">// Compilation error, "constant" address space is not defined</span>
<span class="pl-c">// in OpenCL C++ kernel language</span>
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(constant <span class="pl-k">int</span> * input)
{
  <span class="pl-c">// Compilation error, "private" address space is not defined</span>
  <span class="pl-c">// in OpenCL C++ kernel language</span>
  private <span class="pl-k">int</span> x;
  <span class="pl-c">// ...</span>
}</pre>
</div>
<h4><a id="user-content-examples-correct-opencl-c" class="anchor" href="#examples-correct-opencl-c" aria-hidden="true"></a>Examples, correct (OpenCL C++)</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_work_item<span class="pl-pds">&gt;</span></span>

kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::global_ptr&lt;<span class="pl-k">int</span>[]&gt; input)
{
  cl::local&lt;<span class="pl-k">int</span>[<span class="pl-c1">256</span>]&gt; array;

  <span class="pl-c1">uint</span> gid = <span class="pl-c1">cl::get_global_id</span>(<span class="pl-c1">0</span>);
  array[gid] = input[gid];
  <span class="pl-c">// ...</span>
}

kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::constant_ptr&lt;<span class="pl-k">int</span>[]&gt; input)
{
  <span class="pl-k">int</span> x = <span class="pl-c1">0</span>;
  <span class="pl-c">// ...</span>
}

<span class="pl-k">int</span> y; <span class="pl-c">// Allocated in global memory</span>
<span class="pl-k">static</span> <span class="pl-k">int</span> z; <span class="pl-c">// Allocated in global memory</span>

kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::constant_ptr&lt;<span class="pl-k">int</span>[]&gt; input)
{
  <span class="pl-k">int</span> x = <span class="pl-c1">0</span>; <span class="pl-c">// Allocated in private memory</span>
  <span class="pl-k">static</span> cl::global&lt;<span class="pl-k">int</span>&gt; w; <span class="pl-c">// Allocated in global memory</span>
  <span class="pl-c">// ...</span>
}</pre>
</div>
<h5><a id="user-content-note-2" class="anchor" href="#note-2" aria-hidden="true"></a>Note</h5>
<blockquote>More examples on address spaces can be found in subsections <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-2" rel="nofollow">3.4.5. Restrictions</a> and <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#examples-3" rel="nofollow">3.4.6. Examples</a> of section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces-library" rel="nofollow">Address Spaces Library</a> in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html" rel="nofollow">OpenCL C++ specification</a>.</blockquote>
<h3><a id="user-content-kernel-function-restrictions" class="anchor" href="#kernel-function-restrictions" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-KernelRestrictions"></a>Kernel Function Restrictions</h3>
Since OpenCL C++ kernel language is based on C++14 several restrictions were defined for kernel function to make it resemble kernel function known from OpenCL C:
<ul>
 	<li>A kernel functions are by implicitly declared as extern "C".</li>
 	<li>A kernel function cannot be overloaded.</li>
 	<li>A kernel function cannot be template function.</li>
 	<li>A kernel function cannot be called by another kernel function.</li>
 	<li>A kernel function cannot have parameters specified with default values.</li>
 	<li>A kernel function must have the return type void.</li>
 	<li>A kernel function cannot be called main.</li>
</ul>
<h5><a id="user-content-note-3" class="anchor" href="#note-3" aria-hidden="true"></a>Note</h5>
<blockquote>Compared to OpenCL C in OpenCL C++ you cannot call a kernel function from another kernel function.</blockquote>
<h4><a id="user-content-opencl-c-specification-references-2" class="anchor" href="#opencl-c-specification-references-2" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#kernel-functions" rel="nofollow">OpenCL C++ Programming Language: Kernel Functions</a></li>
</ul>
<h4><a id="user-content-examples-bad-1" class="anchor" href="#examples-bad-1" aria-hidden="true"></a>Examples, bad</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// A kernel function cannot be template function.</span>
<span class="pl-k">template</span>&lt;<span class="pl-k">class</span> <span class="pl-en">T</span>&gt;
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::global_ptr&lt;T[]&gt; input, <span class="pl-c1">uint</span> size)
{ <span class="pl-c">/* ... */</span> }

<span class="pl-c">// A kernel function cannot have parameters specified with default values.</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>(cl::global_ptr&lt;<span class="pl-c1">uint</span>[]&gt; input, <span class="pl-c1">uint</span> size = <span class="pl-c1">10</span>)
{ <span class="pl-c">/* ... */</span> }

kernel <span class="pl-k">void</span> <span class="pl-en">bar</span>(cl::global_ptr&lt;<span class="pl-c1">uint</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{
  <span class="pl-c">// A kernel function cannot be called by another kernel function.</span>
  <span class="pl-c1">foo</span>(input, size);
}

<span class="pl-c">// A kernel function cannot be overloaded.</span>
kernel <span class="pl-k">void</span> <span class="pl-en">bar</span>(cl::global_ptr&lt;<span class="pl-k">float</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{ <span class="pl-c">/* ... */</span> }</pre>
</div>
<h4><a id="user-content-examples-correct-1" class="anchor" href="#examples-correct-1" aria-hidden="true"></a>Examples, correct</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">template</span>&lt;<span class="pl-k">class</span> <span class="pl-en">T</span>&gt;
<span class="pl-k">void</span> <span class="pl-en">function_template</span>(cl::global_ptr&lt;T[]&gt; input, <span class="pl-c1">uint</span> size)
{ <span class="pl-c">/* ... */</span> }

<span class="pl-c">// Specialization for T = float</span>
<span class="pl-k">template</span>&lt;&gt;
<span class="pl-k">void</span> <span class="pl-en">function_template</span>(cl::global_ptr&lt;<span class="pl-k">float</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{ <span class="pl-c">/* ... */</span> }

kernel <span class="pl-k">void</span> <span class="pl-en">kernel_uint</span>(cl::global_ptr&lt;<span class="pl-c1">uint</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{
  function_template&lt;<span class="pl-c1">uint</span>&gt;(input, size);
}

kernel <span class="pl-k">void</span> <span class="pl-en">kernel_float</span>(cl::global_ptr&lt;<span class="pl-k">float</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{
  function_template&lt;<span class="pl-k">float</span>&gt;(input, size);
}</pre>
</div>
<h3><a id="user-content-kernel-parameter-restrictions" class="anchor" href="#kernel-parameter-restrictions" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-KernelParamsRestrictions"></a>Kernel Parameter Restrictions</h3>
The OpenCL host compiler and the OpenCL C++ kernel language device compiler can have different requirements for i.e. type sizes, data packing and alignment, etc., therefore the kernel parameters must meet the following requirements:
<ul>
 	<li>Types passed by pointer or reference must be standard layout types.</li>
 	<li>Types passed by value must be POD types.</li>
 	<li>Types cannot be declared with the built-in bool scalar type, vector type or a class that contain bool scalar or vector type fields.</li>
 	<li>Types cannot be structures and classes with bit field members.</li>
 	<li>Marker types must be passed by value (<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">Marker Types section</a>).</li>
 	<li><code>global</code>, <code>constant</code>, <code>local</code> storage classes can be passed only by reference or pointer. More details in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#explicit-address-space-storage-classes" rel="nofollow">Explicit address space storage classes</a> section.</li>
 	<li>Pointers and references must point to one of the following address spaces: global, local or constant.</li>
</ul>
<h4><a id="user-content-opencl-c-specification-references-3" class="anchor" href="#opencl-c-specification-references-3" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#kernel-functions" rel="nofollow">OpenCL C++ Programming Language: Kernel Functions</a></li>
</ul>
<h3><a id="user-content-general-restrictions" class="anchor" href="#general-restrictions" aria-hidden="true"></a><a name="user-content-S-OpenCLCXX-GeneralRestrictions"></a>General Restrictions</h3>
The following C++14 features are not supported by OpenCL C++:
<ul>
 	<li>the <code>dynamic_cast</code> operator (ISO C++ Section 5.2.7),</li>
 	<li>type identification (ISO C++ Section 5.2.8),</li>
 	<li>recursive function calls (ISO C++ Section 5.2.2, item 9) unless they are a compile-time constant expression,</li>
 	<li>non-placement <code>new</code> and <code>delete</code> operators (ISO C++ Sections 5.3.4 and 5.3.5),</li>
 	<li><code>goto</code> statement (ISO C++ Section 6.6),</li>
 	<li><code>register</code> and <code>thread_local</code> storage qualifiers (ISO C++ Section 7.1.1),</li>
 	<li><code>virtual</code> function qualifier (ISO C++ Section 7.1.2),</li>
 	<li><strong>function pointers</strong> (ISO C++ Sections 8.3.5 and 8.5.3) <strong>unless they are a compile-time constant expression</strong>,</li>
 	<li>virtual functions and abstract classes (ISO C++ Sections 10.3 and 10.4),</li>
 	<li>exception handling (ISO C++ Section 15),</li>
 	<li>the C++ standard library (ISO C++ Sections 17 . . . 30),</li>
 	<li><code>asm</code> declaration (ISO C++ Section 7.4),</li>
 	<li>no implicit lambda to function pointer conversion (ISO C++ Section 5.1.2, item 6),</li>
 	<li>variadic functions (ISO C99 Section 7.15, Variable arguments &lt;stdarg.h&gt;),</li>
 	<li>and, like C++, OpenCL C++ does not support variable length arrays (ISO C99, Section 6.7.5).</li>
</ul>
To avoid potential confusion with the above, please note the following features are supported in OpenCL C++:
<ul>
 	<li><strong>All variadic templates</strong> (ISO C++ Section 14.5.3) <strong>including variadic function templates are supported</strong>.</li>
</ul>
<h4><a id="user-content-opencl-c-specification-references-4" class="anchor" href="#opencl-c-specification-references-4" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#opencl_cxx_restrictions" rel="nofollow">OpenCL C++ Programming Language: Restrictions</a></li>
</ul>

<hr />

<h2><a id="user-content-opencl-c-standard-library" class="anchor" href="#opencl-c-standard-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL"></a>OpenCL C++ Standard Library</h2>
OpenCL C++ does not support the C++14 standard library, but instead implements its own standard library. It is a replacement for built-in functions provided in OpenCL C.
<h5><a id="user-content-note-4" class="anchor" href="#note-4" aria-hidden="true"></a>Note</h5>
<blockquote>OpenCL C++ classes and functions are NOT auto-included.</blockquote>
<h3><a id="user-content-namespace-cl" class="anchor" href="#namespace-cl" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-NamespaceCL"></a>Namespace cl::</h3>
All class and functions provided in OpenCL C++ Standard Library are located in namespace <code>cl::</code>.
<h4><a id="user-content-opencl-c-specification-references-5" class="anchor" href="#opencl-c-specification-references-5" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#opencl-c-standard-library" rel="nofollow">OpenCL C++ Standard Library</a></li>
</ul>
<h4><a id="user-content-solution-1" class="anchor" href="#solution-1" aria-hidden="true"></a>Solution</h4>
Adding a using-directive <code>using namespace cl;</code> right after including all required headers can reduce work needed to port OpenCL C programs to OpenCL C++.
<h4><a id="user-content-examples-1" class="anchor" href="#examples-1" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_integer<span class="pl-pds">&gt;</span></span> <span class="pl-c">// cl::abs(gentype x)</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>(cl::global_ptr&lt;<span class="pl-k">int</span>[]&gt; input <span class="pl-c">/* note cl:: prefix */</span>, <span class="pl-c1">uint</span> size)
{
  <span class="pl-c1">uint</span> global_id = <span class="pl-c1">cl::get_global_id</span>(<span class="pl-c1">0</span>); <span class="pl-c">// note cl:: prefix</span>
  <span class="pl-k">if</span>(global_id &lt; size)
  {
    <span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span> <span class="pl-c">// no need for cl:: prefix in this scope</span>
    input[global_id] = <span class="pl-c1">abs</span>(input[global_id]);
  }
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_integer<span class="pl-pds">&gt;</span></span> <span class="pl-c">// cl::abs(gentype x)</span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span> <span class="pl-c">// No need for cl:: prefix after this using-directive</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>(global_ptr&lt;<span class="pl-k">int</span>[]&gt; input, <span class="pl-c1">uint</span> size)
{
  <span class="pl-c1">uint</span> global_id = <span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>);
  <span class="pl-k">if</span>(global_id &lt; size)
  {
    input[global_id] = <span class="pl-c1">abs</span>(input[global_id]);
  }
}</pre>
</div>
<h3><a id="user-content-conversions-library" class="anchor" href="#conversions-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-ConversionsLibrary"></a>Conversions Library</h3>
OpenCL C <code>convert_<i>type</i>&lt;<i>_sat</i>&gt;&lt;<i>_roundingMode</i>&gt;()</code> and <code>convert_<i>typeN</i>&lt;<i>_sat</i>&gt;&lt;<i>_roundingMode</i>&gt;()</code> built-in functions were replaced in OpenCL C++ with <code>convert_cast&lt;&gt;</code> function template. The behavior of the conversion may be modified by one or two optional modifiers that specify saturation for out-of-range inputs and rounding behavior.

<strong>Rounding Modes</strong>
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">namespace</span> <span class="pl-en">cl</span>
{
  <span class="pl-k">enum</span> <span class="pl-k">class</span> <span class="pl-en">rounding_mode</span>
  {
    rte, <span class="pl-c">// Round to nearest even</span>
    rtz, <span class="pl-c">// Round toward zero</span>
    rtp, <span class="pl-c">// Round toward positive infinity</span>
    rtn  <span class="pl-c">// Round toward negative infinity</span>
  };
}</pre>
</div>
<h5><a id="user-content-note-5" class="anchor" href="#note-5" aria-hidden="true"></a>Note</h5>
<blockquote>If a rounding mode is not specified, conversions to integer type use the <code>rtz</code> (round toward zero) rounding mode and conversions to floating-point type uses the <code>rte</code> rounding mode.</blockquote>
<h4><a id="user-content-opencl-c-specification-references-6" class="anchor" href="#opencl-c-specification-references-6" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#conversions-library" rel="nofollow">OpenCL C++ Standard Library: Conversions Library</a></li>
</ul>
<h4><a id="user-content-examples-2" class="anchor" href="#examples-2" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_convert<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span> <span class="pl-c">// No need for cl:: prefix after this using-directive</span>

kernel <span class="pl-k">void</span> <span class="pl-en">covert_foo_bar</span>()
{
  int4 i { -<span class="pl-c1">1</span>, <span class="pl-c1">0</span>, <span class="pl-c1">1</span>, <span class="pl-c1">2</span> };
  float4 f { -<span class="pl-c1">1</span>.<span class="pl-c1">5f</span>, -<span class="pl-c1">0</span>.<span class="pl-c1">5f</span>, <span class="pl-c1">0</span>.<span class="pl-c1">5f</span>, <span class="pl-c1">1</span>.<span class="pl-c1">5f</span>};

  <span class="pl-c">// Convert ints to floats using the default rounding mode (rte).</span>
  <span class="pl-c">// In OpenCL C: convert_float4_rtp(i)</span>
  float4 f1 = convert_cast&lt;float4&gt;(i);

  <span class="pl-c">// In OpenCL C: convert_float4_rtp(i)</span>
  float4 f2 = convert_cast&lt;float4, rounding_mode::rtp&gt;(i);

  <span class="pl-c">// In OpenCL C: convert_int4_sat(f)</span>
  int4 i1 = convert_cast&lt;int4, saturate::on&gt;(f);

  <span class="pl-c">// In OpenCL C: convert_int4_sat_rte(f)</span>
  int4 i1 = convert_cast&lt;int4, rounding_mode::rte, saturate::on&gt;(f);
}</pre>
</div>
<h3><a id="user-content-reinterpreting-data-library" class="anchor" href="#reinterpreting-data-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-ReinterpretingDataLibrary"></a>Reinterpreting Data Library</h3>
OpenCL C <code>as_<i>type</i>()</code> and <code>as_<i>typeN</i>()</code> operators used for reinterpreting bits in a data type as another data type in OpenCL were replaced in OpenCL C++ with <code>TargetType as_type(InputType const&amp;)</code> function template.
<h5><a id="user-content-note-6" class="anchor" href="#note-6" aria-hidden="true"></a>Note</h5>
<blockquote>All data types described in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#device_builtin_scalar_data_types" rel="nofollow">Device built-in scalar data types</a> and <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#device_builtin_vector_data_types" rel="nofollow">Device built-in vector data types</a> tables (except <code>bool</code> and <code>void</code>) may be also reinterpreted as another data type of the same size using the <code>as_type()</code> function template for scalar and vector data types.</blockquote>
<h4><a id="user-content-opencl-c-specification-references-7" class="anchor" href="#opencl-c-specification-references-7" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#reinterpreting-data-library" rel="nofollow">OpenCL C++ Standard Library: Reinterpreting Data Library</a></li>
</ul>
<h4><a id="user-content-examples-3" class="anchor" href="#examples-3" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_reinterpret<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span> <span class="pl-c">// No need for cl:: prefix after this using-directive</span>

kernel <span class="pl-k">void</span> <span class="pl-en">reinterpret_bar_foo</span>()
{
  <span class="pl-k">float</span> f = <span class="pl-c1">1</span>.<span class="pl-c1">0f</span>;
  <span class="pl-c1">uint</span> u = as_type&lt;<span class="pl-c1">uint</span>&gt;(f); <span class="pl-c">// Legal. Contains:  0x3f800000</span>

  float4 f = <span class="pl-c1">float4</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">3</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">4</span>.<span class="pl-c1">0f</span>);
  <span class="pl-c">// Legal. Contains:</span>
  <span class="pl-c">// int4(0x3f800000, 0x40000000, 0x40400000, 0x40800000)</span>
  int4 i = as_type&lt;int4&gt;(f);

  <span class="pl-k">int</span> i;
  <span class="pl-c">// Legal. Result is implementation-defined.</span>
  short2 j = as_type&lt;short2&gt;(i);

  float4 f;
  <span class="pl-c">// Error: result and operand have different sizes</span>
  double4 g = as_type&lt;double4&gt;(f);

  float4 f;
  <span class="pl-c">// Legal.</span>
  <span class="pl-c">// g.xyz will have same values as f.xyz.</span>
  <span class="pl-c">// g.w is undefined</span>
  float3 g = as_type&lt;float3&gt;(f);
}</pre>
</div>
<h3><a id="user-content-address-spaces-library" class="anchor" href="#address-spaces-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-AddressSpacesLibrary"></a>Address Spaces Library</h3>
As mentioned in <a href="#S-OpenCLCXX-EndOfExplicitNamedAddressSpaces">End of explicit named address spaces</a>, in OpenCL C++ explicit named address spaces known from OpenCL C were replaced by explicit address space storage and pointer classes.

<strong>Explicit address space storage classes:</strong>
<ul>
 	<li><code>cl::global&lt;T&gt; x</code> - allocated in global memory.
<ul>
 	<li>The global storage class can only be used to declare variables at program, function and class scope.</li>
 	<li>The variables at function and class scope must be declared with <code>static</code> specifier.</li>
</ul>
</li>
 	<li><code>cl::local&lt;T&gt; x</code> - allocated in local memory.
<ul>
 	<li>The local storage class can only be used to declare variables at program, kernel and class scope.</li>
 	<li>The variables at class scope must be declared with <code>static</code> specifier.</li>
</ul>
</li>
 	<li><code>cl::priv&lt;T&gt; x</code> - allocated in private memory.
<ul>
 	<li>The priv storage class cannot be used to declare variables in the program scope, with static specifier or extern specifier.</li>
</ul>
</li>
 	<li><code>cl::constant&lt;T&gt; x</code> - allocated in global memory, read-only.
<ul>
 	<li>The constant storage class can only be used to declare variables at program, kernel and class scope.</li>
 	<li>The variables at class scope must be declared with static specifier.</li>
</ul>
</li>
</ul>
<strong>Explicit address space storage pointers classes:</strong>
<ul>
 	<li><code>cl::global_ptr&lt;T&gt;</code></li>
 	<li><code>cl::local_ptr&lt;T&gt;</code></li>
 	<li><code>cl::private_ptr&lt;T&gt;</code></li>
 	<li><code>cl::constant_ptr&lt;T&gt;</code></li>
</ul>
The explicit address space pointer classes are just like pointers: they can be converted to and from pointers with compatible address spaces, qualifiers and types. Assignment or casting between explicit pointer types of incompatible address spaces is illegal.

All named address spaces are incompatible with all other address spaces, but local, global and private pointers can be converted to standard C++ pointers.
<h4><a id="user-content-restrictions" class="anchor" href="#restrictions" aria-hidden="true"></a>Restrictions</h4>
<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html" rel="nofollow">The OpenCL C++ specification</a> specification in subsections <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-2" rel="nofollow">3.4.5. Restrictions</a> of section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces-library" rel="nofollow">Address Spaces Library</a> contains detailed list of restrictions with examples regarding explicit address space storage and pointer classes. It is very important to read and understand those restrictions.
<h4><a id="user-content-opencl-c-specification-references-8" class="anchor" href="#opencl-c-specification-references-8" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces" rel="nofollow">OpenCL C++ Programming Language: Address Spaces</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces-library" rel="nofollow">OpenCL C++ Standard Library: Address Spaces Library</a></li>
</ul>
<h4><a id="user-content-examples-4" class="anchor" href="#examples-4" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_array<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_work_item<span class="pl-pds">&gt;</span></span>

<span class="pl-k">int</span> x; <span class="pl-c">// Allocated in global address space</span>
cl::global&lt;<span class="pl-k">int</span>&gt; y; <span class="pl-c">// Allocated in global address space</span>

cl::constant&lt;<span class="pl-k">int</span>&gt; z {<span class="pl-c1">0</span>}; <span class="pl-c">// Allocated in global address space, read-only,</span>
                         <span class="pl-c">// must be initialized</span>

<span class="pl-c">// Program scope array of 5 ints allocated in local address space</span>
cl::local&lt;cl::array&lt;<span class="pl-k">int</span>, <span class="pl-c1">5</span>&gt;&gt; w = { <span class="pl-c1">10</span> };

<span class="pl-c">// Explicit address space class object passed by value</span>
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::global_ptr&lt;<span class="pl-k">int</span>[]&gt; input)
{
  cl::local&lt;<span class="pl-k">int</span>[<span class="pl-c1">256</span>]&gt; array;

  <span class="pl-k">static</span> cl::global&lt;<span class="pl-k">int</span>&gt; a;
  <span class="pl-k">static</span> cl::constant&lt;<span class="pl-k">int</span>&gt; b {<span class="pl-c1">0</span>};
}

<span class="pl-c">// Explicit address space storage object passed by reference</span>
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::global&lt;cl::array&lt;<span class="pl-k">int</span>, <span class="pl-c1">5</span>&gt;&gt;&amp; input)
{ <span class="pl-c">/* ... */</span> }

<span class="pl-c">// Explicit address space storage object passed by pointer</span>
kernel <span class="pl-k">void</span> <span class="pl-en">example_kernel</span>(cl::global&lt;<span class="pl-k">int</span>&gt; * input)
{ <span class="pl-c">/* ... */</span> }</pre>
</div>
<h5><a id="user-content-note-7" class="anchor" href="#note-7" aria-hidden="true"></a>Note</h5>
<blockquote>More examples on address spaces can be found in subsections <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-2" rel="nofollow">3.4.5. Restrictions</a> and <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#examples-3" rel="nofollow">3.4.6. Examples</a> of section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#address-spaces-library" rel="nofollow">Address Spaces Library</a> in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html" rel="nofollow">OpenCL C++ specification</a>.</blockquote>
<h3><a id="user-content-marker-types" class="anchor" href="#marker-types" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-MarkerTypes"></a>Marker Types</h3>
Like OpenCL C, OpenCL C++ includes special types - images, pipes. All those types are considered marker types. Being a marker type comes with the following set of restrictions:
<ul>
 	<li>Marker types have the default constructor deleted.</li>
 	<li>Marker types have all default copy and move assignment operators deleted.</li>
 	<li>Marker types have address-of operator deleted.</li>
 	<li>Marker types cannot be used in divergent control flow. It can result in undefined behavior.</li>
 	<li>Size of marker types is undefined.</li>
</ul>
All marker types can be passed to functions only by a reference.
<h4><a id="user-content-opencl-c-specification-references-9" class="anchor" href="#opencl-c-specification-references-9" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">OpenCL C++ Standard Library: Marker Types</a></li>
</ul>
<h4><a id="user-content-examples-5" class="anchor" href="#examples-5" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_image<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_work_item<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

float4 <span class="pl-en">bar_val</span>(image2d&lt;float4&gt; img) {
    <span class="pl-k">return</span> img.<span class="pl-c1">read</span>({<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>), <span class="pl-c1">get_global_id</span>(<span class="pl-c1">1</span>)});
}

float4 <span class="pl-en">bar_ref</span>(image2d&lt;float4&gt;&amp; img) {
    <span class="pl-k">return</span> img.<span class="pl-c1">read</span>({<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>), <span class="pl-c1">get_global_id</span>(<span class="pl-c1">1</span>)});
}

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>(image2d&lt;float4&gt; img)
{
    <span class="pl-c">// Error: marker type cannot be passed by value</span>
    float4 val = <span class="pl-c1">bar_val</span>(img);

    <span class="pl-c">// Correct, passing marker type by reference</span>
    float4 val = <span class="pl-c1">bar_ref</span>(img);
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre>#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_image<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_work_item<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

float4 <span class="pl-en">bar</span>(image2d&lt;float4&gt; img) {
    <span class="pl-k">return</span> img.<span class="pl-c1">read</span>({<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>), <span class="pl-c1">get_global_id</span>(<span class="pl-c1">1</span>)});
}

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>(image2d&lt;float4&gt; img1, image2d&lt;float4&gt; img2)
{
    <span class="pl-c">// Error: marker type cannot be declared in the kernel</span>
    image2d&lt;float4&gt; img3;

    <span class="pl-c">// Error: marker type cannot be assigned</span>
    img1 = img2;

    <span class="pl-c">// Error: taking address of marker type</span>
    image2d&lt;float4&gt; *imgPtr = &amp;img1;

    <span class="pl-c">// Undefined behavior: size of marker type is not defined</span>
    <span class="pl-c1">size_t</span> s = <span class="pl-k">sizeof</span>(img1);

    <span class="pl-c">// Undefined behavior: divergent control flow</span>
    float4 val = <span class="pl-c1">bar</span>(<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>) ? img1: img2);
}</pre>
</div>
<h3><a id="user-content-images-and-samplers-library" class="anchor" href="#images-and-samplers-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-ImagesAndSamplersLibrary"></a>Images and Samplers Library</h3>
Images are another part of the OpenCL that changed a lot compared to OpenCL C. Instead of image types and built-in image read/write functions in OpenCL C++ there are image class templates with corresponding methods. Image and sampler class templates are <a href="#S-OpenCLCXXSTL-MarkerTypes">marker types</a>.
<h4><a id="user-content-image-types" class="anchor" href="#image-types" aria-hidden="true"></a>Image types</h4>
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
</tr>
</thead>
<tbody>
<tr>
<td>image1d_t</td>
<td>cl::image1d</td>
</tr>
<tr>
<td>image1d_buffer_t</td>
<td>cl::image1d_buffer</td>
</tr>
<tr>
<td>image1d_array_t</td>
<td>cl::image1d_array</td>
</tr>
<tr>
<td>image2d_t</td>
<td>cl::image2d</td>
</tr>
<tr>
<td>image2d_array_t</td>
<td>cl::image2d_array</td>
</tr>
<tr>
<td>image2d_depth_t</td>
<td>cl::image2d_depth</td>
</tr>
<tr>
<td>image2d_array_depth_t</td>
<td>cl::image2d_array_depth</td>
</tr>
<tr>
<td>image3d_t</td>
<td>cl::image3d</td>
</tr>
<tr>
<td>sampler_t</td>
<td>cl::sampler</td>
</tr>
</tbody>
</table>
To instantiate image template class user has to specify image element type (which is type returned when reading from an image, and required when writing pixel to an image), and access mode (<code>cl::image_access::read</code> is the default access mode).
<h4><a id="user-content-image-dimension" class="anchor" href="#image-dimension" aria-hidden="true"></a>Image dimension</h4>
Based on the dimension of an image different methods are available. All image types have <code>int width()</code> method, images of dimension 2 or 3 have <code>int height()</code>, 3D images have <code>int depth()</code>, and arrayed images have one additional method - <code>int array_size()</code>. See subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#image-dimension" rel="nofollow">Image dimension</a> of OpenCL C++ Specification for more details.
<h4><a id="user-content-image-element-type" class="anchor" href="#image-element-type" aria-hidden="true"></a>Image element type</h4>
Depending on the type of an image different types are allowed to be specified as image element type template parameter. Image type with invalid pixel type is ill formed. See subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#image-element-types" rel="nofollow">Image element types</a> of OpenCL C++ Specification for more details.

Image processing kernels written in OpenCL C++ can be made more readable using <code>.rgba</code> vector component access (compared to <code>.xyzw</code> in OpenCL C). Like <code>xyzw</code> selector, <code>rgba</code> selector works only for vector types with 4 or less elements. See also Vector Component Access part of subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#builtin-vector-data-types" rel="nofollow">Built-in Vector Data Types</a> and section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#vector-utilities-library" rel="nofollow">Vector Utilities Library</a> of OpenCL C++ Specification.
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
kernel <span class="pl-k">void</span> <span class="pl-en">openclcxx</span>(image2d&lt;uint4, <span class="pl-c">// image element type</span>
                              image_access::read <span class="pl-c">// access mode</span>
                             &gt; img)
{
  uint4 color;
  <span class="pl-c">// rgba selector</span>
  color.<span class="pl-smi">r</span> = <span class="pl-c1">255</span>;
  color.<span class="pl-smi">gb</span> = <span class="pl-c1">uint2</span>(<span class="pl-c1">0</span>);
  color.<span class="pl-smi">a</span> = <span class="pl-c1">255</span>;
  <span class="pl-c">//...</span>
}

<span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">openclc</span>(read_only <span class="pl-c1">image2d_t</span> img) <span class="pl-c">// read_only keyword sets access mode</span>
                                             <span class="pl-c">// image element type not defined</span>
{
  uint4 color;
  <span class="pl-c">// xyzw selector</span>
  color.<span class="pl-smi">x</span> = <span class="pl-c1">255</span>;
  color.<span class="pl-smi">yz</span> = (uint2)(<span class="pl-c1">0</span>);
  color.<span class="pl-smi">w</span> = <span class="pl-c1">255</span>;
  <span class="pl-c">//...</span>
}</pre>
</div>
<h4><a id="user-content-image-access-mode" class="anchor" href="#image-access-mode" aria-hidden="true"></a>Image access mode</h4>
Based on the image access mode different read and write methods are present in the instantiated image class. See subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#image-access" rel="nofollow">Image access</a> of OpenCL C++ Specification for more details.
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">namespace</span> <span class="pl-en">cl</span>
{
  <span class="pl-k">enum</span> <span class="pl-k">class</span> <span class="pl-en">image_access</span>
  {
      sample,
      read,
      write,
      read_write
  };
}</pre>
</div>
<h4><a id="user-content-sampler" class="anchor" href="#sampler" aria-hidden="true"></a>Sampler</h4>
Like in OpenCL C, in OpenCL C++ there only two ways of acquiring a sampler inside of a kernel. One is to pass it as a kernel parameter from host using <code>clSetKernelArg</code> function, the other is to create <code>cl::sampler</code> using <code>make_sampler</code> function in the kernel code. The sampler objects at non-program scope must be declared with static specifier.
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">template </span>&lt;addressing_mode A, normalized_coordinates C, filtering_mode F&gt;
<span class="pl-k">constexpr</span> sampler <span class="pl-en">make_sampler</span>();</pre>
</div>
Sampler parameters and their behavior are described in subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#sampler-modes" rel="nofollow">Sampler Modes</a> of OpenCL C++ Specification.
<h4><a id="user-content-opencl-c-specification-references-10" class="anchor" href="#opencl-c-specification-references-10" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#images-and-samplers-library" rel="nofollow">OpenCL C++ Standard Library: Images and Samplers Library</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">OpenCL C++ Standard Library: Marker Types</a></li>
</ul>
<h4><a id="user-content-examples-6" class="anchor" href="#examples-6" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_image<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_work_item<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

<span class="pl-k">using</span> my_image1d_type = image1d&lt;float4, <span class="pl-c">// image element type</span>
                                image_access::write&gt;; <span class="pl-c">// access mode</span>

<span class="pl-k">using</span> my_image2d_type = image2d&lt;float4&gt;; <span class="pl-c">// access mode is image_access::read</span>

kernel <span class="pl-k">void</span> <span class="pl-en">openclcxx</span>(my_image1d_type img1d, my_image2d_type img2d)
{
    <span class="pl-k">const</span> <span class="pl-k">int</span>  <span class="pl-smi">coords1d</span>(<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>));
    <span class="pl-k">const</span> int2 <span class="pl-smi">coords2d</span>(<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>), <span class="pl-c1">get_global_id</span>(<span class="pl-c1">1</span>));

    float4 <span class="pl-smi">val1d</span>(<span class="pl-c1">0</span>.<span class="pl-c1">0f</span>);
    <span class="pl-c">// 1) write() is enabled because the access mode of my_image1d_type</span>
    <span class="pl-c">//    is image_access::write</span>
    <span class="pl-c">// 2) write() takes int value as pixel coordinates because my_image1d_type</span>
    <span class="pl-c">//    is a 1d image type</span>
    <span class="pl-c">// 3) write() takes float4 value as pixel value because float4 is the image</span>
    <span class="pl-c">//    element type of my_image1d_type</span>
    img1d.<span class="pl-c1">write</span>(coords1d, val1d);

    <span class="pl-c">// 1) read() is enabled because the access mode of my_image2d_type</span>
    <span class="pl-c">//    is image_access::read</span>
    <span class="pl-c">// 2) read() takes int2 as an input argument because my_image2d_type</span>
    <span class="pl-c">//    is a 2d image type</span>
    <span class="pl-c">// 3) read() returns float4 because float4 is the image element type</span>
    <span class="pl-c">//    of my_image2d_type</span>
    float4 val2d = img2d.<span class="pl-c1">read</span>(coords2d);
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">openclc</span>(write_only <span class="pl-c1">image1d_t</span> img1d, <span class="pl-c">// write_only keyword sets access mode</span>
                    read_only  <span class="pl-c1">image2d_t</span> img2d) <span class="pl-c">// read_only keyword sets access mode</span>
{
    <span class="pl-k">const</span> <span class="pl-k">int</span>  coords1d = <span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>);
    <span class="pl-k">const</span> int2 coords2d = (int2)(<span class="pl-c1">get_global_id</span>(<span class="pl-c1">0</span>), <span class="pl-c1">get_global_id</span>(<span class="pl-c1">1</span>));

    float4 val1d = (float4)(<span class="pl-c1">0</span>.<span class="pl-c1">0f</span>);
    <span class="pl-c1">write_imagef</span>(img1d, coords1d, val1d);

    <span class="pl-c">// float4 read_imagef(image2d_t, int2) function is used to</span>
    <span class="pl-c">// read from img 2d image.</span>
    float4 val2d = <span class="pl-c1">read_imagef</span>(img2d, coords2d);
}</pre>
</div>
<h3><a id="user-content-pipes-library" class="anchor" href="#pipes-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-PipesLibrary"></a>Pipes Library</h3>
In OpenCL C++ <code>pipe</code> keyword was replaced with <code>cl::pipe</code> class template. Reserve operations return <code>cl::pipe::reservation</code> object, instead of returning reservation id of type <code>reserve_id_t</code>.

All <code>pipe</code>s-related function were moved to <code>cl::pipe</code> or <code>reservation</code> as their methods.
<h4><a id="user-content-pipe-storage" class="anchor" href="#pipe-storage" aria-hidden="true"></a>Pipe storage</h4>
OpenCL C++ introduces new pipe-related type - <code>cl::pipe_storage</code> class template. It enables programmers to create <code>cl::pipe</code> objects in an OpenCL program without need to create <code>cl_pipe</code> on host using API. <code>cl::pipe_storage</code> class template has two template parameters: <code>T</code> - element type, and <code>N</code> - the maximum number of packets which can be held by an object.
<h5><a id="user-content-note-8" class="anchor" href="#note-8" aria-hidden="true"></a>Note</h5>
One kernel can have only one pipe accessor (<code>cl::pipe</code> object) associated with one <code>cl::pipe_storage</code> object.
<h4><a id="user-content-requirements-and-restictions" class="anchor" href="#requirements-and-restictions" aria-hidden="true"></a>Requirements and Restictions</h4>
<code>cl::pipe::reservation</code>, <code>cl::pipe_storage</code> and <code>cl::pipe</code> are marker types. However, they also have additional sets of requirements and restictions beyond those specified in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">Market Types</a> section. The most important are:
<ul>
 	<li>The element type <code>T</code> of <code>pipe</code> and <code>pipe_storage</code> class templates must be a POD type i.e. satisfy <code>is_pod&lt;T&gt;::value == true</code>.</li>
 	<li>A kernel cannot read from and write to the same pipe object.</li>
 	<li>Variables of type <code>pipe_storage</code> can only be declared at program scope or with the <code>static</code> specifier.</li>
 	<li>Variables of type <code>pipe</code> created from <code>pipe_storage</code> can only be declared inside a kernel function at kernel scope.</li>
 	<li>The <code>reservation</code>, <code>pipe_storage</code>, and <code>pipe</code> types cannot be used as a class or union field, a pointer type, an array or the return type of a function.</li>
 	<li>The <code>reservation</code>, <code>pipe_storage</code>, and <code>pipe</code> types cannot be used with the <code>global</code>, <code>local</code>, <code>priv</code> and <code>constant</code> address space storage classes.</li>
</ul>
The full lists of requirements and restictions can be found in subsections <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#requirements" rel="nofollow">Requirements</a> and <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-5" rel="nofollow">Restrictions</a> of Pipe Library section in OpenCL C++ Specification.
<h4><a id="user-content-opencl-c-specification-references-11" class="anchor" href="#opencl-c-specification-references-11" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#pipes-library" rel="nofollow">OpenCL C++ Standard Library: Pipes Library</a>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#requirements" rel="nofollow">Requirements</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-5" rel="nofollow">Restrictions</a></li>
</ul>
</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">OpenCL C++ Standard Library: Marker Types</a></li>
</ul>
<h4><a id="user-content-examples-7" class="anchor" href="#examples-7" aria-hidden="true"></a>Examples</h4>
Reading from and writing to a pipe:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_pipe<span class="pl-pds">&gt;</span></span>

kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(cl::pipe&lt;<span class="pl-k">int</span> <span class="pl-c">/* type */</span>, cl::pipe_access::write <span class="pl-c">/* access mode */</span>&gt; wp,
                   cl::pipe&lt;<span class="pl-k">int</span> <span class="pl-c">/* access mode defaults to read */</span>&gt; rp)
{
  <span class="pl-k">int</span> val;
  <span class="pl-c">// ...</span>
  <span class="pl-c">// write() method is enabled only for pipes with</span>
  <span class="pl-c">// pipe_access::write access mode</span>
  <span class="pl-k">if</span>(wp.<span class="pl-c1">write</span>(val)) { <span class="pl-c">// val passed by const reference</span>
      <span class="pl-c">// ...</span>
  }

  <span class="pl-c">// read() method is enabled only for pipes with</span>
  <span class="pl-c">// pipe_access::read access mode</span>
  <span class="pl-k">if</span>(rp.<span class="pl-c1">read</span>(val)) { <span class="pl-c">// val passed by reference</span>
      <span class="pl-c">// ...</span>
  }
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(write_only <span class="pl-c">/* access mode */</span> pipe <span class="pl-c">/* keyword */</span> <span class="pl-k">int</span> <span class="pl-c">/* type */</span> wp,
                   read_only  <span class="pl-c">/* access mode */</span> pipe <span class="pl-c">/* keyword */</span> <span class="pl-k">int</span> <span class="pl-c">/* type */</span> rp)
{
  <span class="pl-k">int</span> val;
  <span class="pl-c">// ...</span>

  <span class="pl-c">// In OpenCL write_pipe(...) and read_pipe(...) operations</span>
  <span class="pl-c">// returns 0 when write/read is successful, and a negative</span>
  <span class="pl-c">// value otherwise</span>
  <span class="pl-k">if</span>(<span class="pl-c1">write_pipe</span>(p, &amp;val) == <span class="pl-c1">0</span>) {
      <span class="pl-c">// ...</span>
  }

  <span class="pl-k">if</span>(<span class="pl-c1">read_pipe</span>(p, &amp;val) == <span class="pl-c1">0</span>) {
      <span class="pl-c">// ...</span>
  }
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_pipe<span class="pl-pds">&gt;</span></span>

kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(cl::pipe&lt;<span class="pl-k">int</span>&gt; p)
{
  <span class="pl-k">int</span> val;
  <span class="pl-c">// cl::pipe&lt;int, cl::pipe_access::read&gt;::reservation&lt;memory_scope_work_item&gt;</span>
  <span class="pl-k">auto</span> r = p.<span class="pl-c1">reserve</span>(<span class="pl-c1">3</span>);
  <span class="pl-c">// ...</span>
  <span class="pl-c">// read() method is available because pipe p is in</span>
  <span class="pl-c">// pipe_access::read access mode</span>
  <span class="pl-k">if</span>(r.<span class="pl-c1">read</span>(<span class="pl-c1">2</span>, val)) {
    <span class="pl-c">// ...</span>
  }
  r.<span class="pl-c1">commit</span>();
}</pre>
</div>
Making and using a reservation:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(read_only pipe <span class="pl-k">int</span> p)
{
  <span class="pl-k">int</span> val;
  <span class="pl-c1">reserve_id_t</span> rid = <span class="pl-c1">reserve_read_pipe</span>(p, <span class="pl-c1">3</span>);
  <span class="pl-c">// ...</span>
  <span class="pl-k">if</span>(<span class="pl-c1">read_pipe</span>(p, rid, <span class="pl-c1">2</span>, &amp;val)) {
      <span class="pl-c">// ...</span>
  }
  <span class="pl-c1">commit_read_pipe</span>(p, rid);
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_pipe<span class="pl-pds">&gt;</span></span>

kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(cl::pipe&lt;<span class="pl-k">int</span>&gt; p)
{
  <span class="pl-k">int</span> val;
  <span class="pl-c">// cl::pipe&lt;int, cl::pipe_access::read&gt;::reservation&lt;memory_scope_work_item&gt;</span>
  <span class="pl-k">auto</span> r = p.<span class="pl-c1">reserve</span>(<span class="pl-c1">3</span>);
  <span class="pl-c">// ...</span>
  <span class="pl-c">// read() method is available because pipe p is in</span>
  <span class="pl-c">// pipe_access::read access mode</span>
  <span class="pl-k">if</span>(r.<span class="pl-c1">read</span>(<span class="pl-c1">2</span>, val)) {
    <span class="pl-c">// ...</span>
  }
  r.<span class="pl-c1">commit</span>();
}</pre>
</div>
Using <code>pipe_storage</code>:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_pipe<span class="pl-pds">&gt;</span></span>

cl::pipe_storage &lt;<span class="pl-k">int</span>, <span class="pl-c1">1337</span>&gt; my_pipe;

kernel <span class="pl-k">void</span> <span class="pl-en">reader</span>()
{
  <span class="pl-k">auto</span> p = my_pipe.<span class="pl-smi">get</span>&lt;cl::pipe_access::<span class="pl-c1">read</span>&gt;();
  <span class="pl-c">// ...</span>
  p.<span class="pl-c1">read</span>(...);
  <span class="pl-c">// ...</span>
}

kernel <span class="pl-k">void</span> <span class="pl-en">writer</span>()
{
  <span class="pl-k">auto</span> p = my_pipe.<span class="pl-smi">get</span>&lt;cl::pipe_access::<span class="pl-c1">write</span>&gt;();
  <span class="pl-c">// ...</span>
  p.<span class="pl-c1">write</span>(...);
  <span class="pl-c">// ...</span>
}

kernel <span class="pl-k">void</span> <span class="pl-en">error_kernel</span>()
{
  <span class="pl-k">auto</span> p1 = my_pipe.<span class="pl-smi">get</span>&lt;cl::pipe_access::<span class="pl-c1">write</span>&gt;();
  <span class="pl-c">// Error, one kernel can have only one pipe accessor</span>
  <span class="pl-c">// (cl::pipe object) associated with one cl::pipe_storage object.</span>
  <span class="pl-k">auto</span> p2 = my_pipe.<span class="pl-smi">get</span>&lt;cl::pipe_access::<span class="pl-c1">read</span>&gt;();
  <span class="pl-c">// ...</span>
}</pre>
</div>
<h3><a id="user-content-device-enqueue-library" class="anchor" href="#device-enqueue-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-DeviceEnqueueLibrary"></a>Device Enqueue Library</h3>
When it comes to enqueuing a kernel without host interaction, the biggest difference between OpenCL C and OpenCL C++ is that in OpenCL C++ enqueued kernel can be a lambda expression or a function, whereas in OpenCL C it is defined using block syntax.

All functions except function which returns default device queue and kernel query functions were moved to appropriate classes as their methods. See <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#header-opencl_device_queue-synopsis" rel="nofollow">Header &lt;opencl_device_queue&gt; Synopsis</a> subsections of OpenCL C++ specification.
<h4><a id="user-content-device-queue" class="anchor" href="#device-queue" aria-hidden="true"></a>Device Queue</h4>
In OpenCL C++ <code>cl::device_queue</code> class represents device queue (<code>queue_t</code> in OpenCL C). <code>cl::device_queue</code> is a marker type (see <a href="#S-OpenCLCXXSTL-MarkerTypes">Marker Types</a>).
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
</tr>
</thead>
<tbody>
<tr>
<td>queue_t</td>
<td>cl::device_queue</td>
</tr>
</tbody>
</table>
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">namespace</span> <span class="pl-en">cl</span>
{
  <span class="pl-k">struct</span> <span class="pl-en">device_queue</span>: marker_type
  {
    <span class="pl-c">// ...</span>

    <span class="pl-k">template </span>&lt;<span class="pl-k">class</span> <span class="pl-en">Fun</span>, <span class="pl-k">class</span>... Args&gt;
    enqueue_status <span class="pl-en">enqueue_kernel</span>(enqueue_policy flag,
                                  <span class="pl-k">const</span> ndrange &amp;ndrange,
                                  Fun fun,
                                  Args... args) <span class="pl-k">noexcept</span>;

    <span class="pl-c">// In OpenCL C:</span>
    <span class="pl-c">// int enqueue_kernel(queue_t queue,</span>
    <span class="pl-c">//                    kernel_enqueue_flags_t flags,</span>
    <span class="pl-c">//                    const ndrange_t ndrange,</span>
    <span class="pl-c">//                    void (^block)(local void *, ...),</span>
    <span class="pl-c">//                    uint size0, ...);</span>

    <span class="pl-c">// ...</span>
  };
}</pre>
</div>
<h5><a id="user-content-note-9" class="anchor" href="#note-9" aria-hidden="true"></a>Note</h5>
<blockquote><code>args</code> are the arguments that will be passed to <code>fun</code> when kernel will be enqueued with the exception for <code>local_ptr</code> parameters. For local pointers user must supply the size of local memory that will be allocated using <code>local_ptr::size_type{<i>num_elements</i>}</code>. In OpenCL C user has to pass <code>uint</code> value for a corresponding local pointer, which specifies the size of a local memory accessible using that local pointer.</blockquote>
<h4><a id="user-content-event" class="anchor" href="#event" aria-hidden="true"></a>Event</h4>
In OpenCL C++ <code>cl::event</code> class represents device-side event (<code>clk_event_t</code> in OpenCL C).
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
</tr>
</thead>
<tbody>
<tr>
<td>clk_event_t</td>
<td>cl::event</td>
</tr>
</tbody>
</table>
<code>cl::event</code> has the same possible states as <code>clk_event_t</code>, however in OpenCL C++ error is not represented by any negative value, but rather by <code>cl::event_status::error</code> enum.
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>CL_SUBMITTED</td>
<td>cl::event_status::submitted</td>
<td>Initial status of a user event</td>
</tr>
<tr>
<td>CL_COMPLETE</td>
<td>cl::event_status::complete</td>
<td></td>
</tr>
<tr>
<td>Any negative integer value</td>
<td>cl::event_status::error</td>
<td>Status indicating an error</td>
</tr>
</tbody>
</table>
See <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#event-class-methods" rel="nofollow">Event Class Methods</a> and <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#event-status" rel="nofollow">Event Status</a> subsections of OpenCL C++ specification.
<h4><a id="user-content-enqueue-policy" class="anchor" href="#enqueue-policy" aria-hidden="true"></a>Enqueue Policy</h4>
Available enqueue policies did not changed compared to OpenCL C. In OpenCL C enqueue policy type was <code>kernel_enqueue_flags_t</code> enum, in OpenCL C++ it is <code>cl::enqueue_policy</code> enum class.
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
</tr>
</thead>
<tbody>
<tr>
<td>CLK_ENQUEUE_FLAGS_NO_WAIT</td>
<td>cl::enqueue_polic::no_wait</td>
</tr>
<tr>
<td>CLK_ENQUEUE_FLAGS_WAIT_KERNEL</td>
<td>cl::enqueue_polic::wait_kernel</td>
</tr>
<tr>
<td>CLK_ENQUEUE_FLAGS_WAIT_WORK_GROUP</td>
<td>cl::enqueue_polic::wait_work_group</td>
</tr>
</tbody>
</table>
See <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#enqueue-policy" rel="nofollow">Enqueue Policy</a> subsection of OpenCL C++ specification.
<h4><a id="user-content-requirements" class="anchor" href="#requirements" aria-hidden="true"></a>Requirements</h4>
Functor and lambda objects passed to <code>enqueue_kernel()</code> method of device queue has to follow specific restrictions:
<ul>
 	<li>It has to be trivially copyable.</li>
 	<li>It has to be trivially copy constructible.</li>
 	<li>It has to be trivially destructible.</li>
</ul>
Code enqueuing function objects that do not meet this criteria is ill-formed.
<h4><a id="user-content-opencl-c-specification-references-12" class="anchor" href="#opencl-c-specification-references-12" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#device-enqueue-library" rel="nofollow">OpenCL C++ Standard Library: Device Enqueue Library</a>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-6" rel="nofollow">Restrictions</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#examples-7" rel="nofollow">Examples</a></li>
</ul>
</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#marker-types" rel="nofollow">OpenCL C++ Standard Library: Marker Types</a></li>
</ul>
<h4><a id="user-content-examples-8" class="anchor" href="#examples-8" aria-hidden="true"></a>Examples</h4>
Block syntax vs. lambda expression:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_device_queue<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>

kernel <span class="pl-k">void</span> <span class="pl-en">my_func</span>(cl::global_ptr&lt;<span class="pl-k">int</span>&gt; a, cl::global_ptr&lt;<span class="pl-k">int</span>&gt; b, cl::global_ptr&lt;<span class="pl-k">int</span>&gt; c)
{
  <span class="pl-c">// ...</span>
  <span class="pl-k">auto</span> dq = <span class="pl-c1">cl::get_default_device_queue</span>();
  dq.<span class="pl-c1">enqueue_kernel</span>(
    cl::enqueue_polic::no_wait,
    <span class="pl-c1">cl::ndrange</span>({<span class="pl-c1">10</span>, <span class="pl-c1">10</span>}),
    [=](){          <span class="pl-c">//</span>
      *a = *b + *c; <span class="pl-c">// Lambda expression</span>
    }               <span class="pl-c">//</span>
  );
  <span class="pl-c">// ...</span>
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">my_func</span>(global <span class="pl-k">int</span> *a, global <span class="pl-k">int</span> *b, global <span class="pl-k">int</span> *c)
{
  <span class="pl-c">// ...</span>
  <span class="pl-c1">enqueue_kernel</span>(
    <span class="pl-c1">get_default_queue</span>(),
    CLK_ENQUEUE_FLAGS_NO_WAIT,
    <span class="pl-c1">ndrange_2D</span>(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>),
    ^{              <span class="pl-c">//</span>
      *a = *b + *c; <span class="pl-c">// Block syntax</span>
    }               <span class="pl-c">//</span>
  );
  <span class="pl-c">// ...</span>
}</pre>
</div>
Enqueuing a functor:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_device_queue<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>

<span class="pl-k">struct</span> <span class="pl-en">my_functor</span> {
    <span class="pl-k">void</span> <span class="pl-en">operator</span> ()(cl::local_ptr&lt;ushort16[]&gt; p, <span class="pl-k">int</span> x) <span class="pl-k">const</span>
    { <span class="pl-c">/* ... */</span> }
};

kernel <span class="pl-k">void</span> <span class="pl-en">my_func</span>(cl::device_queue q)
{
  <span class="pl-c">// ...</span>
  my_functor f;
  dq.<span class="pl-c1">enqueue_kernel</span>(
    cl::enqueue_polic::no_wait,
    <span class="pl-c1">cl::ndrange</span>(<span class="pl-c1">1</span>),
    f, <span class="pl-c">// functor</span>
    cl::local_ptr&lt;ushort16[]&gt;::size_type{<span class="pl-c1">10</span>}, <span class="pl-c">// define size of p</span>
    <span class="pl-c1">2</span> <span class="pl-c">// x</span>
  );
  <span class="pl-c">// ...</span>
}</pre>
</div>
<h3><a id="user-content-relational-functions" class="anchor" href="#relational-functions" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-RelationalFunctions"></a>Relational Functions</h3>
In OpenCL C++ there were significant changes in signatures and/or behaviour of built-in relational functions. This is because OpenCL C++ introduces <code>bool<i>N</i></code> type which can replace <code>int<i>N</i></code> as a type returned by relational functions.
<h4><a id="user-content-all-and-any" class="anchor" href="#all-and-any" aria-hidden="true"></a><code>all()</code> and <code>any()</code></h4>
In OpenCL C:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// igentype can be char, charN, short, shortN, int, intN, long, and longN</span>
<span class="pl-k">int</span> <span class="pl-en">any</span> (igentype x);
<span class="pl-k">int</span> <span class="pl-en">all</span> (igentype x);</pre>
</div>
<blockquote><code>any()</code> returns 1 if <strong>the most significant bit</strong> in any component of <code>x</code> is set; otherwise returns 0.</blockquote>
<blockquote><code>all()</code> returns 1 if <strong>the most significant bit</strong> in all components of <code>x</code> is set; otherwise returns 0.</blockquote>
In OpenCL C++:
<div class="highlight highlight-source-c++">
<pre><span class="pl-k">bool</span> <span class="pl-en">any</span>(booln t);
<span class="pl-k">bool</span> <span class="pl-en">all</span>(booln t);</pre>
</div>
<blockquote><code>any()</code> returns <code>true</code> if any component of <code>t</code> is <code>true</code>; otherwise returns <code>false</code>.</blockquote>
<blockquote><code>all()</code> returns <code>true</code> if all components of <code>t</code> are <code>true</code>; otherwise returns <code>false</code>.</blockquote>
<h4><a id="user-content-select" class="anchor" href="#select" aria-hidden="true"></a><code>select()</code></h4>
In OpenCL C:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// igentype can be char, charN, short, shortN, int, intN, long, and longN</span>
<span class="pl-c">// ugentype can be uchar, ucharN, ushort, ushortN, uint, uintN, ulong, and ulongN</span>
gentype <span class="pl-en">select</span> (gentype a, gentype b, igentype c);
gentype <span class="pl-en">select</span> (gentype a, gentype b, ugentype c);</pre>
</div>
<blockquote>For each component of a vector type, <code>result[i] = if MSB of c[i] is set ? b[i] : a[i]</code>.</blockquote>
<blockquote>For scalar type, <code>result = c ? b : a</code>.</blockquote>
<blockquote><code>igentype</code> and <code>ugentype</code> must have the same number of elements and bits as <code>gentype</code>.</blockquote>
<blockquote>NOTE: The above definition means that the behavior of select and the ternary operator for vector and scalar types is dependent on different interpretations of the bit pattern of <code>c</code>.</blockquote>
In OpenCL C++ <code>select()</code> is less confusing:
<div class="highlight highlight-source-c++">
<pre>gentype <span class="pl-en">select</span>(gentype a, gentype b, booln c);</pre>
</div>
<blockquote>For each component of a vector type, <code>result[i] = c[i] ? b[i] : a[i]</code>.</blockquote>
<blockquote>For a scalar type, <code>result = c ? b : a</code>.</blockquote>
<blockquote><code>bool<i>N</i></code> must have the same number of elements as gentype.</blockquote>
<h4><a id="user-content-opencl-c-specification-references-13" class="anchor" href="#opencl-c-specification-references-13" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#relational-functions" rel="nofollow">OpenCL C++ Standard Library: Relational Functions</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#expressions" rel="nofollow">OpenCL C++ Programming Language: Expressions</a></li>
</ul>
<h4><a id="user-content-examples-9" class="anchor" href="#examples-9" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_relational<span class="pl-pds">&gt;</span></span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>()
{
  <span class="pl-k">bool</span> b1 = <span class="pl-c1">isequal</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">1</span>.<span class="pl-c1">0f</span>); <span class="pl-c">// true</span>
  <span class="pl-k">bool</span> b2 = <span class="pl-c1">isequal</span>(<span class="pl-c1">1.0</span>, <span class="pl-c1">2.0</span>); <span class="pl-c">// false</span>

  bool2 b3 = <span class="pl-c1">isequal</span>(<span class="pl-c1">float2</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>), <span class="pl-c1">float2</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>)); <span class="pl-c">// { true, true }</span>
  bool2 b4 = <span class="pl-c1">isequal</span>(<span class="pl-c1">double2</span>(<span class="pl-c1">1.0</span>), <span class="pl-c1">double2</span>(<span class="pl-c1">2.0</span>)); <span class="pl-c">// { false, false }</span>

  bool2 b5 = { <span class="pl-c1">true</span>, <span class="pl-c1">false</span> };
  <span class="pl-k">auto</span> b6 = <span class="pl-c1">all</span>(b3); <span class="pl-c">// false</span>
  <span class="pl-k">auto</span> b7 = <span class="pl-c1">any</span>(b3); <span class="pl-c">// true</span>

  bool2 c { <span class="pl-c1">true</span>, <span class="pl-c1">false</span> };
  float2 a {  <span class="pl-c1">1</span>.<span class="pl-c1">0f</span>,  <span class="pl-c1">1</span>.<span class="pl-c1">0f</span> };
  float2 b { -<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, -<span class="pl-c1">1</span>.<span class="pl-c1">0f</span> };
  <span class="pl-k">auto</span> r1 = <span class="pl-c1">select</span>(a, b, c); <span class="pl-c">// { -1.0f, 1.0f }</span>

  <span class="pl-k">auto</span> r2 = <span class="pl-c1">select</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">false</span>); <span class="pl-c">// 1.0f</span>
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>()
{
  <span class="pl-c">// Note: in integer value -1 MSB is set to 1</span>

  <span class="pl-k">int</span> b1 = <span class="pl-c1">isequal</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">1</span>.<span class="pl-c1">0f</span>); <span class="pl-c">// 1 (true)</span>
  <span class="pl-k">long</span> b2 = <span class="pl-c1">isequal</span>(<span class="pl-c1">1.0</span>, <span class="pl-c1">2.0</span>); <span class="pl-c">// 0 (false)</span>

  int2 b3 = <span class="pl-c1">isequal</span>((float2)(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>), (float2)(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>)); <span class="pl-c">// { -1, -1 } ({ true, true })</span>
  long2 b4 = <span class="pl-c1">isequal</span>((double2)(<span class="pl-c1">1.0</span>), (double2)(<span class="pl-c1">2.0</span>)); <span class="pl-c">// { 0, 0 }  ({ false, false })</span>

  <span class="pl-k">int</span> b5 = <span class="pl-c1">all</span>( (int2)(-<span class="pl-c1">1</span>, <span class="pl-c1">10</span>) ); <span class="pl-c">// 0</span>
  <span class="pl-k">int</span> b6 = <span class="pl-c1">all</span>( (int2)(-<span class="pl-c1">1</span>, -<span class="pl-c1">1</span>) ); <span class="pl-c">// 1</span>

  <span class="pl-k">int</span> b7 = <span class="pl-c1">any</span>( (int2)(-<span class="pl-c1">1</span>, <span class="pl-c1">0</span>) ); <span class="pl-c">// 1</span>
  <span class="pl-k">int</span> b8 = <span class="pl-c1">any</span>( (int2)(<span class="pl-c1">1</span>, <span class="pl-c1">1</span>) ); <span class="pl-c">// 0</span>

  int2 c = (int2)(-<span class="pl-c1">1</span>, <span class="pl-c1">1</span>);
  float2 a = (float2)(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">1</span>.<span class="pl-c1">0f</span>);
  float2 b = (float2)(-<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, -<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>);
  float2 r1 = <span class="pl-c1">select</span>(a, b, c); <span class="pl-c">// { -1.0f, 1.0f }</span>

  <span class="pl-k">float</span> r2 = <span class="pl-c1">select</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>, -<span class="pl-c1">1</span>); <span class="pl-c">// 2.0f</span>
  <span class="pl-k">float</span> r3 = <span class="pl-c1">select</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>,  <span class="pl-c1">1</span>); <span class="pl-c">// 1.0f</span>
  <span class="pl-k">float</span> r4 = <span class="pl-c1">select</span>(<span class="pl-c1">1</span>.<span class="pl-c1">0f</span>, <span class="pl-c1">2</span>.<span class="pl-c1">0f</span>,  <span class="pl-c1">0</span>); <span class="pl-c">// 1.0f</span>
}</pre>
</div>
<h3><a id="user-content-vector-data-load-and-store-functions" class="anchor" href="#vector-data-load-and-store-functions" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-VectorDataLoadandStoreFunctions"></a>Vector Data Load and Store Functions</h3>
In OpenCL C++ vector data load and store functions were greatly simplified compared to OpenCL: instead of 39 different functions, now there are just 9 function templates. The requirements and the behaviours of functions have not be changed. Also arguments and their order was not changed.
<table>
<thead>
<tr>
<th>OpenCL C</th>
<th>OpenCL C++</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>gentype<i>N</i> vload<i>N</i></code></td>
<td><code>template &lt;size_t N, class T&gt; make_vector_t&lt;T, N&gt; vload</code></td>
</tr>
<tr>
<td><code>void vstore<i>N</i>(...)</code></td>
<td><code>template &lt;class T&gt; void vstore(…, vector_element_t&lt;T&gt;* p)</code></td>
</tr>
<tr>
<td><code>float<i>N</i> vload_half[<i>N</i>]</code></td>
<td><code>template &lt;size_t N&gt; make_vector_t&lt;float, N&gt; vload_half</code></td>
</tr>
<tr>
<td><code>void vstore_half[<i>N</i>][<i>_rounding_mode</i>]</code></td>
<td><code>template &lt;rounding_mode R, class Type&gt; void vstore_half(…, half* p)</code></td>
</tr>
<tr>
<td><code>float<i>N</i> vloada_half<i>N</i></code></td>
<td><code>template &lt;size_t N&gt; make_vector_t&lt;float, N&gt; vloada_half</code></td>
</tr>
<tr>
<td><code>void vstore_half<i>N</i>[<i>_rounding_mode</i>]</code></td>
<td><code>template &lt;rounding_mode R, class T&gt; void vstorea_half(…, half* p)</code></td>
</tr>
</tbody>
</table>
Read <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#header-opencl_vector_load_store" rel="nofollow">Header &lt;opencl_vector_load_store&gt; Synopsis</a> subsection of <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#vector-data-load-and-store-functions" rel="nofollow">Vector Data Load and Store Functions</a> section to see vector data load and store function templates declarations.
<h4><a id="user-content-opencl-c-specification-references-14" class="anchor" href="#opencl-c-specification-references-14" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#vector-data-load-and-store-functions" rel="nofollow">OpenCL C++ Standard Library: Vector Data Load and Store Functions</a></li>
</ul>
<h4><a id="user-content-examples-10" class="anchor" href="#examples-10" aria-hidden="true"></a>Examples</h4>
<code>vload</code> and <code>vstore</code>:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_vector_load_store<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(<span class="pl-k">float</span> * fptr, <span class="pl-k">const</span> constant_ptr&lt;half&gt; hptr)
{
  <span class="pl-k">auto</span> f4 = vload&lt;<span class="pl-c1">4</span>&gt;(<span class="pl-c1">0</span>, fptr); <span class="pl-c">// reads from (fptr + (0 * 4)), float4 returned</span>
  <span class="pl-k">auto</span> f2 = vload&lt;<span class="pl-c1">2</span>&gt;(<span class="pl-c1">2</span>, fptr); <span class="pl-c">// reads from (fptr + (2 * 2)), float2 returned</span>

#<span class="pl-k">ifdef</span> cl_khr_fp16 <span class="pl-c">// cl_khr_fp16 must be defined and supported</span>
  <span class="pl-k">auto</span> h8 = vload&lt;<span class="pl-c1">8</span>&gt;(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 8)), half8 returned</span>
#<span class="pl-k">endif</span>

  <span class="pl-c1">vstore</span>(float4{ <span class="pl-c1">1</span>, <span class="pl-c1">2</span>, <span class="pl-c1">3</span>, <span class="pl-c1">4</span>}, <span class="pl-c1">0</span>, fptr); <span class="pl-c">// float4 stored at (fptr + (0 * 4))</span>
  <span class="pl-c1">vstore</span>(f2, <span class="pl-c1">2</span>, fptr); <span class="pl-c">// float2 stored at (fptr + (2 * 2))</span>
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar</span>(<span class="pl-k">float</span> * fptr, <span class="pl-k">const</span> constant half * hptr)
{
  float4 f4 = <span class="pl-c1">vload4</span>(<span class="pl-c1">0</span>, fptr); <span class="pl-c">// reads from (fptr + (0 * 4)), float4 returned</span>
  float2 f2 = <span class="pl-c1">vload2</span>(<span class="pl-c1">2</span>, fptr); <span class="pl-c">// reads from (fptr + (2 * 2)), float2 returned</span>

#<span class="pl-k">ifdef</span> cl_khr_fp16 <span class="pl-c">// cl_khr_fp16 must be defined and supported</span>
  half8 h8 = <span class="pl-c1">vload8</span>(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 8)), half8 returned</span>
#<span class="pl-k">endif</span>

  <span class="pl-c1">vstore4</span>(f4, <span class="pl-c1">0</span>, fptr); <span class="pl-c">// float4 stored at (fptr + (0 * 4))</span>
  <span class="pl-c1">vstore2</span>(f2, <span class="pl-c1">2</span>, fptr); <span class="pl-c">// float2 stored at (fptr + (2 * 2))</span>
}</pre>
</div>
<code>vload_half</code>, <code>vstore_half</code>, <code>vloada_half</code>, and <code>vstorea_half</code>:
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_vector_load_store<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foobar_half</span>(half * hptr)
{
  <span class="pl-c">// half vload</span>
  <span class="pl-k">auto</span> f4 = vload_half&lt;<span class="pl-c1">4</span>&gt;(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float4 returned</span>
  <span class="pl-k">auto</span> f3 = vload_half&lt;<span class="pl-c1">3</span>&gt;(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 3)), float3 returned</span>

  <span class="pl-c">// half array vload</span>
  <span class="pl-k">auto</span> f4a = vloada_half&lt;<span class="pl-c1">4</span>&gt;(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float4 returned</span>
  <span class="pl-k">auto</span> f3a = vloada_half&lt;<span class="pl-c1">3</span>&gt;(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float3 returned</span>

  <span class="pl-c">// half vstore</span>
  <span class="pl-c1">vstore_half</span>(f3, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float3 stored at (hptr + (0 * 3)),</span>
                            <span class="pl-c">// rounded to nearest even (rounding_mode::rte)</span>
  vstore_half&lt;rounding_mode::rtz&gt;(f4, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float4 stored at (hptr + (0 * 4)),</span>
                                                <span class="pl-c">// rounded toward zero</span>
  <span class="pl-c">// half array vstore</span>
  <span class="pl-c1">vstorea_half</span>(f3a, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float3 stored at (hptr + (0 * 4))</span>
                              <span class="pl-c">// rounded to nearest even (rounding_mode::rte)</span>
  vstorea_half&lt;rounding_mode::rtz&gt;(f4a, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float4 stored at (hptr + (0 * 4))</span>
                                                  <span class="pl-c">// rounded toward zero</span>
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C</span>
kernel <span class="pl-k">void</span> <span class="pl-en">foobar_half</span>(half * hptr)
{
  <span class="pl-c">// half vload</span>
  float4 f4 = <span class="pl-c1">vload_half4</span>(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float4 returned</span>
  float3 f3 = <span class="pl-c1">vload_half3</span>(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 3)), float3 returned</span>

  <span class="pl-c">// half array vload</span>
  float4 f4a = <span class="pl-c1">vloada_half4</span>(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float4 returned</span>
  float3 f3a = <span class="pl-c1">vloada_half3</span>(<span class="pl-c1">0</span>, hptr); <span class="pl-c">// reads from (hptr + (0 * 4)), float3 returned</span>

  <span class="pl-c">// half vstore</span>
  <span class="pl-c1">vstore_half3</span>(f3, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float3 stored at (hptr + (0 * 3)),</span>
                             <span class="pl-c">// rounded to nearest even</span>
  <span class="pl-c1">vstore_half4_rtz</span>(f4, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float4 stored at (hptr + (0 * 4)),</span>
                                 <span class="pl-c">// rounded toward zero</span>

  <span class="pl-c">// half array vstore</span>
  <span class="pl-c1">vstorea_half3</span>(f3a, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float3 stored at (hptr + (0 * 4))</span>
                               <span class="pl-c">// rounded to nearest even</span>
  <span class="pl-c1">vstorea_half4_rtz</span>(f4a, <span class="pl-c1">0</span>, hptr); <span class="pl-c">// float4 stored at (hptr + (0 * 4))</span>
                                   <span class="pl-c">// rounded toward zero</span>
}
</pre>
</div>
<h3><a id="user-content-atomic-operations-library" class="anchor" href="#atomic-operations-library" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXSTL-AtomicOperationsLibrary"></a>Atomic Operations Library</h3>
OpenCL C atomic operation are based on C11 atomics. In OpenCL C++ atomics are based on C++14 atomics and synchronization operations. Section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#atomic-operations-library" rel="nofollow">Atomic Operations Library</a> of OpenCL C++ presents synopsis of the atomics library and differences from C++14 specification.

Because atomic functions in OpenCL C and OpenCL C++ have virtually the same argument lists adding <code>using namespace cl;</code> can Significantly speed up porting kernels to OpenCL C++.
<h4><a id="user-content-atomic-types" class="anchor" href="#atomic-types" aria-hidden="true"></a>Atomic types</h4>
In OpenCL C++ different OpenCL C atomic types like <code>atomic_int</code>, <code>atomic_float</code> were replaced with one class template <code>atomic&lt;T&gt;</code>, however, for supported types proper type alias are declared (for example: <code>using atomic_int = atomic&lt;int&gt;;</code>).
<ul>
 	<li>There are explicit specializations for integral types. Each of these specializations provides set of extra operators suitable for integral types.</li>
 	<li>There is an explicit specialization of the atomic template for pointer types.</li>
 	<li>All atomic classes have deleted copy constructor and deleted copy assignment operators.</li>
 	<li>64-bit atomic types require <code>cl_khr_int64_base_atomics</code> and <code>cl_khr_int64_extended_atomics</code> extensions and <code>atomic&lt;double&gt;</code> in addition requires <code>cl_khr_fp64</code>.</li>
</ul>
<h4><a id="user-content-restrictions-1" class="anchor" href="#restrictions-1" aria-hidden="true"></a>Restrictions</h4>
<ul>
 	<li>The generic <code>atomic&lt;T&gt;</code> class template is only available if <code>T</code> is <code>int</code>, <code>uint</code>, <code>long</code>, <code>ulong</code>, <code>float</code>, <code>double</code>, <code>intptr_t</code>, <code>uintptr_t</code>, <code>size_t</code>, <code>ptrdiff_t</code>.</li>
 	<li>The atomic data types cannot be declared inside a kernel or non-kernel function unless they are declared as <code>static</code> keyword or in <code>local&lt;T&gt;</code> and <code>global&lt;T&gt;</code> containers. See examples.</li>
 	<li>The atomic operations on the private memory can result in undefined behavior.</li>
 	<li><code>memory_order_consume</code> from C++14 is not supported by OpenCL C++.</li>
</ul>
Full list of restrictions can be found in subsection <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#restrictions-3" rel="nofollow">Restrictions</a> of section <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#atomic-operations-library" rel="nofollow">Atomic Operations Library</a> in OpenCL C++ specification.
<h4><a id="user-content-opencl-c-specification-references-15" class="anchor" href="#opencl-c-specification-references-15" aria-hidden="true"></a>OpenCL C++ Specification References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#atomic-operations-librarys" rel="nofollow">OpenCL C++ Standard Library: Atomic Operations Library</a></li>
</ul>
<h4><a id="user-content-examples-11" class="anchor" href="#examples-11" aria-hidden="true"></a>Examples</h4>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C++</span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_memory<span class="pl-pds">&gt;</span></span>
#<span class="pl-k">include</span> <span class="pl-s"><span class="pl-pds">&lt;</span>opencl_atomic<span class="pl-pds">&gt;</span></span>
<span class="pl-k">using</span> <span class="pl-k">namespace</span> <span class="pl-en">cl</span><span class="pl-k">;</span>

atomic_int a; <span class="pl-c">// OK: program scope atomic in the global memory</span>
              <span class="pl-c">// atomic_int is alias for atomic&lt;int&gt;</span>
local&lt;atomic&lt;<span class="pl-k">int</span>&gt;&gt; <span class="pl-en">b</span>(<span class="pl-c1">1</span>); <span class="pl-c">// OK: program scope atomic in the local memory</span>
                         <span class="pl-c">// Initialized to 1. The initialization is not atomic.</span>
global&lt;atomic&lt;<span class="pl-k">int</span>&gt;&gt; c = ATOMIC_VAR_INIT(<span class="pl-c1">2</span>); <span class="pl-c">// OK: program scope atomic in the global memory</span>
                                            <span class="pl-c">// Initialized to 2. The initialization is not atomic.</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>()
{
    <span class="pl-k">static</span> global&lt;atomic&lt;<span class="pl-k">int</span>&gt;&gt; d; <span class="pl-c">// OK: atomic in the global memory</span>
    <span class="pl-k">static</span> atomic&lt;<span class="pl-k">int</span>&gt; e; <span class="pl-c">// OK: atomic in the global memory</span>
    local&lt;atomic&lt;<span class="pl-k">int</span>&gt;&gt; f; <span class="pl-c">// OK: atomic in the local memory</span>

    atomic&lt;global&lt;<span class="pl-k">int</span>&gt;&gt; g; <span class="pl-c">// Error: class members cannot be</span>
                           <span class="pl-c">//        in address space</span>

    atomic&lt;<span class="pl-k">int</span>&gt; h; <span class="pl-c">// undefined behavior</span>

    <span class="pl-c1">atomic_init</span>(&amp;a, <span class="pl-c1">123</span>); <span class="pl-c">// Initialize a to 123. The initialization is not atomic.</span>
}</pre>
</div>
<div class="highlight highlight-source-c++">
<pre><span class="pl-c">// OpenCL C+</span>
global atomic_int a; <span class="pl-c">// OK: program scope atomic in the global memory</span>
local  atomic_int b; <span class="pl-c">// Error: program scope local variables not suppoerted in OpenCL C</span>
global atomic_int c = ATOMIC_VAR_INIT(<span class="pl-c1">2</span>); <span class="pl-c">// OK: program scope atomic in the global memory</span>
                                          <span class="pl-c">// Initialized to 2. The initialization is not atomic.</span>

kernel <span class="pl-k">void</span> <span class="pl-en">foo</span>()
{
    <span class="pl-k">static</span> global atomic_int d; <span class="pl-c">// OK: atomic in the global memory</span>
    <span class="pl-k">static</span> atomic_int e; <span class="pl-c">// OK: atomic in the global memory</span>
    local atomic_int f; <span class="pl-c">// OK: atomic in the local memory</span>

    atomic_int h; <span class="pl-c">// undefined behavior</span>

    <span class="pl-c1">atomic_init</span>(&amp;a, <span class="pl-c1">123</span>); <span class="pl-c">// Initialize a to 123. The initialization is not atomic.</span>
}</pre>
</div>

<hr />

<h2><a id="user-content-opencl-c-compilation-process" class="anchor" href="#opencl-c-compilation-process" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXCompilation"></a>OpenCL C++ Compilation Process</h2>
OpenCL C++ kernel language can not be consumed by <code>clCreateProgramWithSource()</code> API function, which is used to create a program from OpenCL C source. OpenCL C++ source first have to be compiled to SPIR-V 1.2 binary, which can later be passed to <code>clCreateProgramWithIL()</code> to create an OpenCL program. After that program can be build with <code>clBuildProgram()</code>.
<h3><a id="user-content-opencl-c-compilation-to-spir-v" class="anchor" href="#opencl-c-compilation-to-spir-v" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXCompilationToSPIRV"></a>OpenCL C++ Compilation to SPIR-V</h3>
To compile OpenCL C++ kernel language to SPIR-V user have to use compiler that is not a part of OpenCL framework. The Khronos Group provides reference <a href="https://github.com/KhronosGroup/SPIR/tree/spirv-1.1">offline compiler based on Clang 3.6</a> and an implementation of OpenCL C++ Standard Library called <a href="https://github.com/KhronosGroup/libclcxx">libclcxx</a>.
<h4><a id="user-content-preprocessor-options" class="anchor" href="#preprocessor-options" aria-hidden="true"></a>Preprocessor options</h4>
Every preprocessor option that would normally be specified in <code>clBuildProgram()</code>, for OpenCL C++ must be passed when it is being compiled to SPIR-V.
<pre><code>-D name
</code></pre>
Predefine name as a macro, with definition 1.
<pre><code>-D name=definition
</code></pre>
The contents of definition are tokenized and processed as if they appeared during translation phase three in a <code>#define</code> directive. In particular, the definition will be truncated by embedded newline characters.
<h4><a id="user-content-other-compilation-options" class="anchor" href="#other-compilation-options" aria-hidden="true"></a>Other compilation options</h4>
Some feature-related options must be specified during compilation to SPIR-V:
<ul>
 	<li><code>-cl-fp16-enable</code> - enables full half data type support and defines <code>cl_khr_fp16</code> macro. Disabled by default.</li>
 	<li><code>-cl-fp64-enable</code> - enables full double data type support and defines <code>cl_khr_fp64</code> macro. Disabled by default.</li>
 	<li><code>-cl-zero-init-local-mem-vars</code> - enables software zero-initialization of variables allocated in local memory.</li>
</ul>
<h3><a id="user-content--building-program-created-from-spir-v" class="anchor" href="#-building-program-created-from-spir-v" aria-hidden="true"></a><a name="user-content-S-OpenCLCXXCompilationBuildSPIRV"></a> Building program created from SPIR-V</h3>
When an OpenCL program created using <code>clCreateProgramWithIL()</code> is compiled (<code>clBuildProgram()</code>) not all build options are allowed. They have to be passed when compiling to SPIR-V. Otherwise, there is no difference between building program created from SPIR-V and program created from OpenCL C source. Which options are ignored and which not is described in <a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2.html#_compiler_options" rel="nofollow">OpenCL 2.2 API Specification</a>.
<h4><a id="user-content-opencl-c-specification-and-opencl-22-api-references" class="anchor" href="#opencl-c-specification-and-opencl-22-api-references" aria-hidden="true"></a>OpenCL C++ Specification and OpenCL 2.2 API References</h4>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html#compiler_options" rel="nofollow">OpenCL C++ Specification: Compiler options</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2.html#_compiler_options" rel="nofollow">OpenCL 2.2 Specification: Compiler Options</a></li>
</ul>
<h1><a id="user-content-bibliography" class="anchor" href="#bibliography" aria-hidden="true"></a><a name="user-content-S-Bibliography"></a>Bibliography</h1>
<h3><a id="user-content-opencl-specifications" class="anchor" href="#opencl-specifications" aria-hidden="true"></a>OpenCL Specifications</h3>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.pdf" rel="nofollow">The OpenCL C++ 1.0 Specification</a> (<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.html" rel="nofollow">HTML</a>)</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2.pdf" rel="nofollow">The OpenCL 2.2 API Specification</a> (<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2.html" rel="nofollow">HTML</a>)</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-extension.pdf" rel="nofollow">The OpenCL 2.2 Extension Specification</a> (<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-extension.html" rel="nofollow">HTML</a>)</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-environment.pdf" rel="nofollow">OpenCL 2.2 SPIR-V Environment Specification</a> (<a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-environment.html" rel="nofollow">HTML</a>)</li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.0-openclc.pdf" rel="nofollow">The OpenCL C 2.0 Language Specification</a></li>
 	<li><a href="https://www.khronos.org/registry/OpenCL/specs/opencl-2.1.pdf" rel="nofollow">The OpenCL 2.1 API Specification</a></li>
</ul>
<h3><a id="user-content-opencl-reference-pages" class="anchor" href="#opencl-reference-pages" aria-hidden="true"></a>OpenCL Reference Pages</h3>
<ul>
 	<li>The OpenCL 2.2 Reference Page (not published yet)</li>
 	<li><a href="http://www.khronos.org/registry/cl/sdk/2.1/docs/man/xhtml/" rel="nofollow">The OpenCL 2.1 Reference Page</a></li>
</ul>
<h3><a id="user-content-opencl-headers" class="anchor" href="#opencl-headers" aria-hidden="true"></a>OpenCL Headers</h3>
<ul>
 	<li><a href="https://github.com/KhronosGroup/OpenCL-Headers/tree/master/opencl22">OpenCL 2.2 Headers</a></li>
 	<li><a href="https://github.com/KhronosGroup/OpenCL-Headers/tree/master/opencl21">OpenCL 2.1 Headers</a></li>
</ul>
<h3><a id="user-content-other" class="anchor" href="#other" aria-hidden="true"></a>Other</h3>
<ul>
 	<li><a href="https://www.khronos.org/registry/OpenCL/" rel="nofollow">Khronos OpenCL Registry</a> (<a href="https://github.com/KhronosGroup/OpenCL-Registry">GitHub</a>)</li>
 	<li><a href="https://www.khronos.org/news/press/khronos-releases-opencl-2.2-with-spir-v-1.2" rel="nofollow">OpenCL 2.2 Release Note</a></li>
 	<li>Michael Wong, Adam Stanski, Maria Rovatsou, Ruyman Reyes, Ben Gaster, and Bartok Sochaski. 2016. C++ for OpenCL Workshop, IWOCL 2016. In Proceedings of the 4th International Workshop on OpenCL (IWOCL '16).
<ul>
 	<li><a href="http://www.iwocl.org/wp-content/uploads/iwocl-2016-dive-into-opencl-c.pdf" rel="nofollow">Dive into OpenCL C++</a></li>
 	<li><a href="http://www.iwocl.org/wp-content/uploads/iwcol-2016-opencl-ckernel-language.pdf" rel="nofollow">OpenCL C++ kernel language</a></li>
</ul>
</li>
</ul>