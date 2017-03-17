# OpenCL.org Initiative

TODO: What is OpenCL.org initiative?

## Acknowledgements

## About [StreamComputing](https://streamcomputing.eu)

# Table Of Contents

##### NOTE FOR CONTRIBUTORS:
> [Edit this when you add/remove sections/chapters.]

1. [OpenCL Introduction](#S-introduction)
2. Platforms and tools
   1. SDKs
   2. Debuggers
   3. Profilers
3. [Libraries](#S-libraries)
4. Specification
5. Getting Started
6. Profiling
7. Bibliography

# <a name="S-introduction"></a>OpenCL Introduction 

# Platforms and tools

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

* Homepage: https://github.com/boostorg/compute/
* Documentation: http://www.boost.org/doc/libs/1_63_0/libs/compute/doc/html/index.html
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
> Full list of libraries using OpenCL can be found at [khronos.org/opencl/resources](https://www.khronos.org/opencl/resources).

# <a name="S-specification"></a>Specification

# Getting Started

# Profiling

# Bibliotgraphy

* [The Khronos OpenCL Working Group, The OpenCL Specification, Version: 2.1](https://www.khronos.org/registry/OpenCL/specs/opencl-2.1.pdf)
* [The Khronos OpenCL Working Group, The OpenCL C Specification, Version: 2.0](https://www.khronos.org/registry/OpenCL/specs/opencl-2.0-openclc.pdf)