# OpenCL.org Initiative

TODO: What is OpenCL.org initiative?

## Acknowledgements

## About [StreamComputing](https://streamcomputing.eu)

# Table Of Contents

##### NOTE FOR CONTRIBUTORS:
> [Edit this when you add/remove sections/chapters.]

1. [OpenCL Introduction](#S-introduction)
2. [Platforms and tools](#S-platforms)
   1. [SDKs](#S-platforms-sdks)
   2. [Debuggers and profilers](#S-platforms-tools)
3. [Libraries](#S-libraries)
4. [Specification](#S-specification)
5. [Getting Started](#S-get-started)
6. Profiling
7. Bibliography

# <a name="S-introduction"></a>OpenCL Introduction

## What is OpenCL?

OpenCL is an open, royalty-free industry standard that makes much faster computations possible through parallel computing. The standard is controlled by non-profit standards organisation Khronos. For example, by using this technology with graphics cards and modern multi-core processors it is possible to convert a video in 20 minutes instead of 2 hours, or analyze spectra of hundreds of thousands of stars in minutes instead of several hours.

## How does it work?

OpenCL is an extension to existing languages. It makes it possible to specify a piece of code that is executed multiple times independently from each other. This code can run on various processors – not only the main CPU. Also, OpenCL has built-in support for vector types (float2, short4, int8, long16, etc), and operations on those types can be mapped to vector extensions of modern CPUs (SSE, AVX).

>For example, you need to calculate `sin(x)` of a large array `A` of one million numbers. Based on the information provided by the OpenCL framework you can pick the best device, or even several devices, and send the data to the device(s). Normally you would loop over the million numbers, one number after another, but with OpenCL you can just say: "For each `x` in array `A` give me `sin(x)`", and each `x` is processed in parallel. When finished, you can take the data back from the device(s) or proceed with other computations.

OpenCL framework is great at exposing parallel nature of various compute-devices: x86 CPUs, GPUs, FPGAs, DSPs. This can significantly lower the total execution time compared to conventional sequential methods.

## 5 questions on OpenCL

**Q: Why is it so fast?**

>OpenCL framework gives programmers tools and features which enable them to implement efficient parallel algorithms. It does it by providing direct access to the cores of multi-core CPUs, and to the hundreds of little processors on graphics cards. 

**Q: Does it work on any type of hardware?**

>As it is an open standard, it can work on any type of hardware that targets parallel execution. This can be a CPU, GPU, DSP or FPGA. You can read more in [Platforms and tools](#S-platforms) chapter.

**Q: How does it compare to OpenMP/MPI?**

>Where OpenMP and MPI try to split loops over threads/servers and is CPU-oriented, OpenCL focuses on getting threads being data-position aware and making use of processor-capabilities.

**Q: Does it replace C or C++?**

>No, it is a framework which integrates well with C, C++, Python, Java and more.

**Q: How stable/mature is OpenCL?**

>First version of OpenCL was released in 2009. The latest version is OpenCL 2.1 from November 2015. The standard is being actively developed. Next version (2.2), which currently has provisional status, will include OpenCL C++ kernel language based C++14 standard.

# <a name="S-platforms"></a>Platforms and tools

## <a name="S-platforms-sdks"></a>SDKs And OpenCL Implementations

### [AMD OpenCL™ Accelerated Parallel Processing - AMD APP SDK](http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing-app-sdk/)

> AMD OpenCL™ Accelerated Parallel Processing (APP) technology is a set of advanced hardware and software technologies that enable AMD graphics processing cores (GPU), working in concert with the system’s x86 cores (CPU), to execute heterogeneously to accelerate many applications beyond just graphics.

The SDK provides samples, documentation, and other materials. GPU drivers must be installed in order to run OpenCL programs on AMD GPUs.

##### Note
> AMD Linux Catalyst driver are not being updated since 18.12.2015 (Crimson Edition 15.12). We recommend you to install and test AMDGPU-PRO driver (only OpenCL 1.2), or try AMD ROCm.

Supported hardware:
 * AMD GPU
 * AMD APU
 * x86 CPU

Supported OS:
 * Windows
 * Linux 

Standards:
 * OpenCL 2.0 (AMD Catalyst/Crimson)
 * OpenCL 1.2 (AMDGPU-PRO)

#### See also
 * [AMD OpenCL Programming Guide](http://amd-dev.wpengine.netdna-cdn.com/wordpress/media/2013/12/AMD_OpenCL_Programming_User_Guide2.pdf)
 * [List of various OpenCL OS projects/libraries supported by AMD](http://developer.amd.com/tools-and-sdks/open-source/)

### [AMD ROCm](https://radeonopencompute.github.io/) (Partially Open Source)

ROCm is Open Source Platform for GPU Computing. ROCm 1.4 includes developer preview of OpenCL support (not yet open source in this release).

Supported hardware:
 * AMD GPU (limited list, GFX7 and GFX8 only - Hawaii, Fiji, Polaris)

Supported OS:
 * Linux (with a special ROCm kernel)

Standards:
 * OpenCL 2.0 compatible kernel language and OpenCL 1.2 compatible runtime

#### See also
* https://github.com/RadeonOpenCompute/ROCm

### [Intel® SDK for OpenCL™ Applications](https://software.intel.com/en-us/intel-opencl)

> The Intel® SDK for OpenCL™ Applications is a comprehensive development environment for developing and optimizing OpenCL™ applications on Intel® platforms.

>The SDK supports offloading compute-intensive parallel workloads to Intel® Graphics Technology using an advanced OpenCL™ kernel compiler, runtime debugger and code performance analyzer.

Intel® SDK for OpenCL™ Application includes Intel OpenCL™ Code Builder which allows you to build (offline), debug, and analyze OpenCL programs.

##### Note
> Since Intel SDK supports OpenCL 2.1 (and therefore supports SPIR-V 1.0), it is currently the only SDK that lets you run OpenCL C++ kernels (not all features works).

Supported hardware:
 * Intel® Graphics (GPU)
 * Intel® Processors (CPU)
 * Intel® Xeon Phi™ Coprocessors

Supported OS:
 * Windows
 * Linux
 * Android (as a target only)

Standards:
 * OpenCL 2.0 & 1.2
 * OpenCL 2.1 (CPU only) with SPIR and SPIR-V support


#### See also

* [Free OpenCL training materials provided by Intel](https://software.intel.com/en-us/intel-opencl-support/training)
* [OpenCL code samples provided by Intel](https://software.intel.com/en-us/intel-opencl-support/code-samples)

### [NVIDIA® CUDA® Toolkit](https://developer.nvidia.com/cuda-toolkit)

NVIDIA does not have a separate OpenCL SDK, but CUDA Toolkit contains OpenCL headers and shared libraries.
OpenCL support is included in NVIDIA GPU drivers.

OpenCL samples with source code for Windows, Linux and macOS are available at https://developer.nvidia.com/opencl

Supported hardware:
 * NVIDIA GPUs

Supported OS:
 * Windows
 * Linux
 * macOS

Standards:
 * OpenCL 1.2
 * beta-support of OpenCL 2.0

#### See also
* https://developer.nvidia.com/opencl
* [NVIDIA beta-support for OpenCL 2.0 on Windows](https://streamcomputing.eu/blog/2017-02-22/nvidia-enables-opencl-2-0-beta-support/), and [on Linux](https://streamcomputing.eu/blog/2017-03-06/nvidia-beta-support-opencl-2-0-linux/)

### [Portable Computing Language (pocl)](http://portablecl.org/) (Open Source)

>Portable Computing Language (pocl) aims to become a MIT-licensed open source implementation of the OpenCL standard which can be easily adapted for new targets and devices, both for homogeneous CPU and heterogeneous GPUs/accelerators.

>pocl uses Clang as an OpenCL C frontend and LLVM for the kernel compiler implementation, and as a portability layer. Thus, if your desired target has an LLVM backend, it should be able to get OpenCL support easily by using pocl.

Supported hardware:
 * x86 CPU
 * HSA targets

Supported OS:
 * Windows
 * Linux

Standards:
 * OpenCL 1.2 with [some features missing](https://github.com/pocl/pocl/wiki/OpenCL-1.2-missing-features)

#### See also
* [pocl on GitHub](https://github.com/pocl/pocl)
* [pocl's documentation](http://portablecl.org/docs/html/)

### [Intel® FPGA SDK for OpenCL™](https://www.altera.com/products/design-software/embedded-software-developers/opencl/overview.html)

>The Intel FPGA SDK for OpenCL allows the easy implementation of applications onto FPGAs by abstracting away the complexities of FPGA design, allowing software programmers to write hardware-accelerated kernel functions in OpenCL C, an ANSI C-based language with additional OpenCL constructs.
As part of our SDK we provide a suite of tools to further resemble the fast development flow of software programmers.

Supported hardware:
 * Altera FPGA

Supported software:
 * Windows
 * Linux

Standards:
 * OpenCL 1.0 with support of SVM (OpenCL 2.0 feature) and image arrays (OpenCL 1.2 feature)


### [Xilinx SDAccel™ development environment](http://www.xilinx.com/products/design-tools/software-zone/sdaccel.html)

>The SDAccel™ development environment for OpenCL™, C, and C++, enables up to 25X better performance/watt for data center application acceleration leveraging FPGAs.
SDAccel, member of the SDx™ family, combines the industry's first architecturally optimizing compiler supporting any combination of OpenCL, C, and C++ kernels, along with libraries, development boards and the first complete CPU/GPU like development and run-time experience for FPGAs.

### [Qualcomm® Adreno™ SDK](https://developer.qualcomm.com/software/adreno-gpu-sdk)

Supported hardware:
 * Adreno GPU (Snapdragon processor)

Supported software:
 * Windows
 * Linux
 * macOS
 * Android (as a target only)

Standards:
 * OpenCL 2.0

### [ARM Mali OpenCL SDK](https://developer.arm.com/products/software/mali-sdks/opencl)

>The Mali OpenCL SDK provides developers with a framework and series of samples for developing OpenCL 1.1 applications on ARM Mali based platforms such as the Mali-T600 and above family of GPUs.
The samples cover a wide range of use cases that utilize the Mali GPU to achieve a significant improvement in performance when compared to running on the CPU alone.

Supported hardware:
 * ARM Mali based platforms such as the Mali-T600 family of GPUs

Supported software:
 * Windows
 * Linux

Standards:
 * OpenCL 1.1

### [The Texas Instruments OpenCL Implementation](http://downloads.ti.com/mctools/esd/docs/opencl/intro.html)

Texas Instruments supports OpenCL 1.1 on a few selected processors. Full list of supported systems is available [here](http://downloads.ti.com/mctools/esd/docs/opencl/intro.html).

Supported hardware:
 * Selected processors ([list](http://downloads.ti.com/mctools/esd/docs/opencl/intro.html))

Standards:
 * OpenCL 1.1

#### See also
* [Introduction to OpenCL on TI Embedded Processors](https://training.ti.com/sites/default/files/docs/Introduction_to_OpenCL_slides.pdf)

## <a name="S-platforms-tools"></a>Debuggers and profilers

### [CodeXL](http://gpuopen.com/compute-product/codexl/) (Debugger, Profiler)

> CodeXL is a comprehensive tool suite that enables developers to harness the benefits of CPUs, GPUs and APUs. CodeXL is available both as a Visual Studio extension and a standalone user interface application for Windows and Linux.

CodeXL works on ROCm platform.

Features:
* Combined Host and GPU Debugging
    * Real-time OpenCL kernel debugging
* AMD GPU Profiling
    * Application Timeline Trace (multiple contexts and queues, tips about redundant synchronizations, OpenCL objects leaks...)
    * GPU Performance Counters (memory transfers, cache hits, occupancy, registers usage...)
* CPU Profiling 
* Static Kernel Analysis
* APU/CPU/GPU power profiling
* Remote machine profiling and debugging 

#### See also

* [GPUOpen technical blogs about CodeXL by AMD](http://gpuopen.com/tag/codexl/)
    * [Getting Up to Speed on the CodeXL GPU Profiler](http://gpuopen.com/getting-up-to-speed-with-the-codexl-gpu-profiler-and-radeon-open-compute/)
* [Detailed list of all features](http://gpuopen.com/compute-product/codexl/)
* https://github.com/GPUOpen-Tools/CodeXL

### [Intel® VTune™ Amplifier XE](https://software.intel.com/en-us/intel-vtune-amplifier-xe) (Profiler)

>With Intel® VTune™ Amplifier, you get all these advanced profiling capabilities with a single, friendly analysis interface. And for media applications, you also get powerful tools to tune OpenCL* and the GPU.

##### Note
> Intel® SDK for OpenCL™ Application includes Intel OpenCL™ Code Builder which allows you to build (offline), debug, and analyze OpenCL programs.

#### See alo

* [Detailed list of features](https://software.intel.com/en-us/intel-vtune-amplifier-xe/details)
* [Intel VTune Amplifier XE: Getting started with OpenCL performance analysis on Intel HD Graphics](https://software.intel.com/en-us/articles/intel-vtune-amplifier-xe-getting-started-with-opencl-performance-analysis-on-intel-hd-graphics)

### [Intel® Graphics Performance Analyzers (GPA)](https://software.intel.com/en-us/gpa) (Debugger, Profiler)

Intel® SDK for OpenCL™ Applications provides integration with the Intel® Graphics Performance Analyzers (Intel® GPA), which enables you to optimize and analyze your OpenCL code in visual computing applications.

Intel GPA support various metrics for Intel CPU and HD Graphics devices. Some
metrics are specific to the rendering (Microsoft DirectX* API) pipeline only, while some
are more general and can be associated with OpenCL execution.

With Intel GPA you can also inspect various important hardware counters for Intel CPU
and HD Graphics devices in real time, for example:

 * Utilization of CPU cores and the execution units in Intel HD Graphics devices
 * Memory traffic for Intel HD Graphics devices
 * Power consumption, and so on

#### See also

* [Profiling OpenCL Applications with System Analyzer and Platform Analyzer](https://software.intel.com/en-us/articles/profiling-opencl-applications-with-system-analyzer-and-platform-analyzer)
* [Collecting OpenCL-related Metrics with Intel Graphics Performance Analyzers](https://software.intel.com/sites/default/files/m/d/4/1/d/8/Collecting_Important_OpenCL-related_metrics_with_GPA.pdf)

### [Qualcomm Snapdragon Profiler](https://developer.qualcomm.com/software/snapdragon-profiler)

> Snapdragon Profiler is profiling software that runs on the Windows, Mac, and Linux platforms. It connects with Android devices powered by Snapdragon processors over USB. Snapdragon Profiler allows developers to analyze CPU, GPU, DSP, memory, power, thermal, and network data, so they can find and fix performance bottlenecks.

#### See also

* [List of docs and tutorials for Snapdragon Profiler](https://developer.qualcomm.com/software/snapdragon-profiler/tools)

### [Oclgrind](https://github.com/jrprice/Oclgrind)

>Oclgrind is an open source SPIR interpreter and OpenCL device simulator. The core of this project is an interpreter for SPIR, which can simulate the execution of an OpenCL kernel running on a virtual OpenCL device. Oclgrind is designed to be extensible and aims to provide a platform with which a variety of useful tools can be created to aid OpenCL application development.

At present, Oclgrind includes support for the following features:

* Detecting memory access errors (such as reading/writing outside the bounds of an array)
* Data-race detection
* Interactive kernel debugging
* Detecting work-group divergence (barriers and async copies)
* Collecting histograms of instructions executed
* Logging OpenCL runtime API errors

#### See also

* [Oclgrind Wiki](https://github.com/jrprice/Oclgrind/wiki)
* [Oclgrind: An Extensible OpenCL Device Simulator @ IWOCL'15](http://www.iwocl.org/wp-content/uploads/iwocl-2015-talk-James-Price-UoBristol.pdf), [Video](https://www.youtube.com/watch?v=J2LSVtybX8g)

# <a name="S-libraries"></a>Libraries

The crucial part of every technology and programming language are libraries. They extend the language, simplify it, add new features, and help you build you applications faster. The same goes for OpenCL. This chapter presents a few selected libraries which at the time seemed popular, useful and reliable.

## Wrappers 

### [OpenCL-CLHPP](https://github.com/KhronosGroup/OpenCL-CLHPP)

OpenCL Host API C++ bindings. 

> For many large applications C++ is the language of choice and so it seems reasonable to define C++ bindings for OpenCL.

> The interface is contained with a single C++ header file cl2.hpp and all definitions are contained within the namespace cl. There is no additional requirement to include cl.h and to use either the C++ or original C bindings; it is enough to simply include cl2.hpp.

> The bindings themselves are lightweight and correspond closely to the underlying C API. Using the C++ bindings introduces no additional execution overhead.

* Homepage: https://github.com/KhronosGroup/OpenCL-CLHPP
* Documentation: http://github.khronos.org/OpenCL-CLHPP

### [Boost.Compute](https://github.com/boostorg/compute/)

Boost.Compute is an official Boost library, added to the Boost C++ Libraries package since 1.61 version. It is a comprehensive wrapper for OpenCL 1.0 - 2.0. It provides STL-like algorithms and various helper features (functions, classes) which make developing in C++/OpenCL environment much faster.

> Boost.Compute is a GPU/parallel-computing library for C++ based on OpenCL.

> The core library is a thin C++ wrapper over the OpenCL API and provides access to compute devices, contexts, command queues and memory buffers.

> On top of the core library is a generic, STL-like interface providing common algorithms (e.g. `transform()`, `accumulate()`, `sort()`) along with common containers (e.g. `vector<T>`, `flat_set<T>`). It also features a number of extensions including parallel-computing algorithms (e.g. `exclusive_scan()`, `scatter()`, `reduce()`) and a number of fancy iterators (e.g. `transform_iterator<>`, `permutation_iterator<>`, `zip_iterator<>`).

* Homepage: https://github.com/boostorg/compute/
* Documentation: http://www.boost.org/doc/libs/1_63_0/libs/compute/doc/html/index.html
* Talks:
    * [Talk about Boost.Compute @ IWOCL 2016](http://www.iwocl.org/wp-content/uploads/iwocl-2016-boost-compute.pdf) by Jakub Szuppe
    * [Boost.Compute @ C++Now 2015](https://github.com/boostcon/cppnow_presentations_2015/blob/master/files/Boost.ComputeCxxNow2015.pdf) by Kyle Lutz ([Video](https://www.youtube.com/watch?v=q7oCblCtTT8))
* Other resources:
    * [Differences between VexCL, Thrust, and Boost.Compute](http://stackoverflow.com/questions/20154179/differences-between-vexcl-thrust-and-boost-compute)  

### [AMD Aparapi](https://github.com/aparapi/aparapi)

Aparapi enables Java developers to define OpenCL kernels and executed them on GPU and APU devices.

>Aparapi allows Java developers to take advantage of the compute power of GPU and APU devices by executing data parallel code fragments on the GPU rather than being confined to the local CPU. It does this by converting Java bytecode to OpenCL at runtime and executing on the GPU, if for any reason Aparapi can't execute on the GPU it will execute in a Java thread pool.

> We like to think that for the appropriate workload this extends Java's 'Write Once Run Anywhere' to include GPU devices.

* Homepage: https://github.com/aparapi/aparapi
* Documentation:
    * https://github.com/aparapi/aparapi/blob/master/doc/README.md
    * https://code.google.com/archive/p/aparapi/


##### Note
> You find more OpenCL wrappers/bindings in blog post [StreamComputing: OpenCL Wrappers](https://streamcomputing.eu/knowledge/for-developers/opencl-wrappers/).

## Specialized libraries

### [VexCL](https://github.com/ddemidov/vexcl)

VexCL was developed with scientific computing in mind. It provides custom DSL to simplify using various accelerators for vector arithmetic, reduction, sparse matrix-vector product etc. It has some features of a wrapper library, but not all of them. However, it is possible to define generic C++ algorithms with its DSL and write custom kernels using VexCL. It supports both OpenCL and CUDA.

>VexCL is a vector expression template library for OpenCL/CUDA. It has been created for ease of GPGPU development with C++. VexCL strives to reduce amount of boilerplate code needed to develop GPGPU applications. The library provides convenient and intuitive notation for vector arithmetic, reduction, sparse matrix-vectork products, etc. Multi-device and even multi-platform computations are supported. The source code of the library is distributed under very permissive MIT license.

* Homepage: https://github.com/ddemidov/vexcl
* Documentation: http://vexcl.readthedocs.io/en/latest/
* Talks: http://vexcl.readthedocs.io/en/latest/talks.html
* Other resources:
    * [Differences between VexCL, Thrust, and Boost.Compute](http://stackoverflow.com/questions/20154179/differences-between-vexcl-thrust-and-boost-compute)

### [ArrayFire](https://arrayfire.com/)

>ArrayFire is a general-purpose library that simplifies the process of developing software that targets parallel and massively-parallel architectures including CPUs, GPUs, and other hardware acceleration devices.

* Homepages:
    * https://arrayfire.com/
    * https://github.com/arrayfire/arrayfire 
* Documentation: http://arrayfire.org/docs/

### [ViennaCL](http://viennacl.sourceforge.net/)

>ViennaCL is a free open-source linear algebra library for computations on many-core architectures (GPUs, MIC) and multi-core CPUs. The library is written in C++ and supports CUDA, OpenCL, and OpenMP (including switches at runtime).

* Homepage: http://viennacl.sourceforge.net/
* Documentation: http://viennacl.sourceforge.net/viennacl-documentation.html

### [clMathLibraries](https://github.com/clMathLibraries)

clMathLibraries is a name for a group of math libraries with OpenCL backend. Also see [StreamComputing: OpenCL alternatives for CUDA Linear Algebra Libraries](https://streamcomputing.eu/blog/2014-01-16/opencl-alternatives-for-cuda-linear-algebra-libraries/).

* [clFFT](https://github.com/clMathLibraries/clFFT) - a software library containing FFT functions written in OpenCL
* [clBLAS](https://github.com/clMathLibraries/clBLAS) - a software library containing BLAS functions written in OpenCL
* [clSPARSE](https://github.com/clMathLibraries/clSPARSE) - a software library containing Sparse functions written in OpenCL.
* [clRNG](https://github.com/clMathLibraries/clRNG) - an OpenCL based software library containing random number generation functions

##### Note
> More complete lists of OpenCL libraries, bindings and toolkits can be found at:
> * [khronos.org/opencl/resources](https://www.khronos.org/opencl/resources), and at
> * [iwocl.org/resources/opencl-libraries-and-toolkits](http://www.iwocl.org/resources/opencl-libraries-and-toolkits/).

# <a name="S-specification"></a>Specification

The OpenCL specification documents can be found on [the Khronos OpenCL Registry](https://www.khronos.org/registry/OpenCL/). They are a must-read for everyone who wants to dive into OpenCL programming and a great reference for the experienced OpenCL developer. 

* OpenCL 2.2 (Provisional)
    * [The OpenCL 2.2 Specification (API)](https://www.khronos.org/registry/OpenCL/specs/opencl-2.2.pdf)
    * [The OpenCL C++ Language Specification](https://www.khronos.org/registry/OpenCL/specs/opencl-2.2-cplusplus.pdf)
* OpenCL 2.1
    * [The OpenCL 2.1 Specification (API)](https://www.khronos.org/registry/OpenCL/specs/opencl-2.1.pdf)
    * [The OpenCL 2.0 C Language Specification](https://www.khronos.org/registry/OpenCL/specs/opencl-2.0-openclc.pdf)
        * Note: OpenCL 2.1 uses OpenCL 2.0 C Language for OpenCL programs
    * [Online Reference Pages](http://www.khronos.org/registry/cl/sdk/2.1/docs/man/xhtml/)
* OpenCL 2.0 
    * [The OpenCL 2.0 Specification (API)](https://www.khronos.org/registry/OpenCL/specs/opencl-2.0.pdf)
    * [The OpenCL 2.0 C Language Specification](https://www.khronos.org/registry/OpenCL/specs/opencl-2.0-openclc.pdf)
    * [Online Reference Pages](http://www.khronos.org/registry/cl/sdk/2.0/docs/man/xhtml/)
* OpenCL 1.2
    * [The OpenCL 1.2 Specification (API)](https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf)
    * [The OpenCL 1.2 C Language Specification](https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf#page=197)
    * [Online Reference Pages](http://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/)
* All specifications and other documents regarding OpenCL standard, including including environment specifications, extensions specifications, and specifications for OpenCL 1.0 and 1.1, are available at https://www.khronos.org/registry/OpenCL.

## The OpenCL Architecture

The crucial part of The OpenCL Specification is the chapter about the OpenCL architecture. It is very important to understand at least its basic principles. Without this knowledge it is not possible neither to efficiently program using OpenCL nor to use it to its full capabilities. 

The architecture is described by four models:

* Platform Model
* Memory Model
* Programming Model
* Execution Model

The platform model describes in general how the OpenCL framework looks like and gives definitions of a host and an OpenCL device. The memory model explains different memory regions and memory objects introduced in OpenCL. More advanced sections about memory model present details of how shared virtual memory mechanism works, and presents memory consistency model and ordering rules. Since OpenCL 2.0 the programming model does not have its own section. It specifies that the OpenCL execution model supports both data parallel and task parallel programming models.

The execution model is the model that explains how the OpenCL framework really works. It defines responsibilities of a host program, it says what is a command queue, what is a kernel, and it describes how an execution of an OpenCL kernel on a device works. In order to understand OpenCL you must carefully read and understand particularly this part of the specification, and know: how workloads are mapped onto devices, when synchronization happens, what is an index space.
 
# <a name="S-get-started"></a>Getting Started

## [CL-basic]()

CL-basic is a C prototype to help you get started with creating your first simple OpenCL application. It offers simplified host-code OpenCL API functions and a sample OpenCL kernel that you can reference to get started quick! In addition, this prototype can be compiled under both Windows and Linux-based systems thanks to the use of CMake.

Host-code functions can be found in files:
> `cl_util.c`

> `cl_util.h`

For getting platform and/or device information, use the following functions are:
```c
void PrintPlatformName(cl_platform_id platform);
void PrintDeviceName(cl_device_id device);
int PrintOpenCLInfo();
void SelectOpenCLPlatformAndDevice(cl_platform_id* pPlatform, cl_device_id* pDevice);
```

For creating and releasing an OpenCL context, the following functions are:
```c
cl_context CreateOpenCLContext(cl_platform_id platform, cl_device_id device);
void ReleaseOpenCLContext(cl_context *pContext);
```

For creating and releasing an OpenCL command queue, the following functions are:
```c
cl_command_queue CreateOpenCLQueue(cl_device_id device, cl_context context);
void ReleaseOpenCLQueue(cl_command_queue *pQueue);
```

For creating and releasing an OpenCL buffer allocated on an OpenCL device, the following functions are:
```c
cl_mem CreateDeviceBuffer(cl_context context, size_t sizeInBytes);
void ReleaseDeviceBuffer(cl_mem *pDeviceBuffer);
```

For copying data rom the host to device, or from the device to the host, the following functions are:
```c
void CopyHostToDevice(void* hostBuffer, cl_mem deviceBuffer, size_t sizeInBytes, cl_command_queue queue, cl_bool blocking);
void CopyDeviceToHost(cl_mem deviceBuffer, void* hostBuffer, size_t sizeInBytes, cl_command_queue queue, cl_bool blocking);
```

For loading an OpenCL source from a file, creating and releasing the OpenCL program, the following functions are:
```c
char* LoadOpenCLSourceFromFile(char* filePath, size_t *pSourceLength);
cl_program CreateAndBuildProgram(cl_context context, char* sourceCode, size_t sourceCodeLength);
void ReleaseProgram(cl_program *pProgram);
```

For creating and releasing OpenCL Kernels, the following functions are:
```c
cl_kernel CreateKernel(cl_program program, char* kernelName);
void ReleaseKernel(cl_kernel *pKernel);
```

An error function has also been prepared to make it easier to translate an error code returned by an OpenCL host function, and it can be used as below:
```c
clError = clEnqueueNDRangeKernel(queue, simpleFunctionKernel, workDim, NULL, globalWorkSize, localWorkSize, 0, NULL, NULL);
CHECK_OCL_ERR("clEnqueueNDRangeKernel", clError);
```

Refer to `main.cpp` for a reference of how these host-code functions are used, and `OpenCLKernels.cl` for how the OpenCL kernel is written. 

# Profiling

# Bibliography

* [The Khronos OpenCL Working Group, The OpenCL Specification, Version: 2.1](https://www.khronos.org/registry/OpenCL/specs/opencl-2.1.pdf)
* [The Khronos OpenCL Working Group, The OpenCL C Specification, Version: 2.0](https://www.khronos.org/registry/OpenCL/specs/opencl-2.0-openclc.pdf)
* [IWOCL.org: Resources](http://www.iwocl.org/resources/)
