---
author: NicholasWilson
comments: false
date: 2017-10-30 13:12:23+00:00
excerpt: DCompute is a framework and compiler extension to support writing native
  kernels for OpenCL and CUDA in D to utilize GPUs and other accelerators for computationally
  intensive code. Its compute API drivers automate the interactions between user code
  and the tedious and error prone APIs with the goal of enabling the rapid development
  of high performance D libraries and applications.
layout: post
link: https://dlang.org/blog/2017/10/30/d-compute-running-d-on-the-gpu/
slug: d-compute-running-d-on-the-gpu
title: 'DCompute: Running D on the GPU'
wordpress_id: 1206
categories:
- Code
- Compilers &amp; Tools
- Guest Posts
permalink: /d-compute-running-d-on-the-gpu/
redirect_from: /2017/10/30/d-compute-running-d-on-the-gpu/
---

_Nicholas Wilson is a student at Murdoch University, studying for his BEng (Hons)/BSc in Industrial Computer Systems (Hons) and Instrumentation & Control/ Molecular Biology & Genetics and Biomedical Science. He just finished his thesis on low-cost defect detection of solar cells by electroluminescence imaging, which gives him time to work on DCompute and write about it for the D Blog.He plays the piano, ice skates, and has spent 7 years putting D to use on number bashing, automation, and anything else that he could make a computer do for him._



* * *



![](http://dlang.org/blog/wp-content/uploads/2017/07/ldc.png)

DCompute is a framework and compiler extension to support writing native kernels for OpenCL and CUDA in D to utilize GPUs and other accelerators for computationally intensive code. Its compute API drivers automate the interactions between user code and the tedious and error prone APIs with the goal of enabling the rapid development of high performance D libraries and applications.


### Introduction


This is the second article on [DCompute](https://github.com/libmir/dcompute). In the [previous article](https://dlang.org/blog/2017/07/17/dcompute-gpgpu-with-native-d-for-opencl-and-cuda/), we looked at the development of DCompute and some trivial examples. While we were able to successfully build kernels, there was no way to run them short of using them with an existing framework or doing everything yourself. This is no longer the case. As of [v0.1.0](https://github.com/libmir/dcompute/releases/tag/v0.1.0), DCompute now comes with native wrappers for both OpenCL and CUDA, enabling kernel dispatch as easily as CUDA.

In order to run a kernel we need to pass it off to the appropriate compute API, either CUDA or OpenCL. While these APIs both try to achieve similar things they are different enough that to squeeze that last bit of performance out of them you need to treat each API separately. But there is sufficient overlap that we can make the interface reasonably consistent between the two. The C bindings to these APIs, however, are very low level and trying to use them is very tedious and extremely prone to error (yay `void*`).
In addition to the tedium and error proneness, you have to redundantly specify a lot of information, which further compounds the problem. Fortunately this is D and we can remove a lot of the redundancy through introspection and code generation.

The drivers wrap the C API, providing a clean and consistent interface that’s easy to use. While the documentation is a little sparse at the moment, the source code is for the most part straightforward (if you’re familiar with the C APIs, looking where a function is used is a good place to start). There is the occasional piece of magic to achieve a sane API.


### Taming the beasts


OpenCL’s `clGet*Info` functions are the way to access properties of the class hidden behind the `void*`. A typical call looks like

    enum CL_FOO_REFERENCE_COUNT = 0x1234;
    cl_foo* foo = ...; 
    cl_int refCount;
    clGetFooInfo(foo, CL_FOO_REFERENCE_COUNT, refCount.sizeof, &refCount,null);
And that’s not even one for which you have to call, to figure out how much memory you need to allocate, then call again with the allocated buffer (and $DEITY help you if you want to get a `cl_program`’s binaries).

Using D, I have been able to turn that into this:

```d
struct Foo
{
    void* raw;
    static struct Info
    {
        @(0x1234) int referenceCount;
        ...
    }
    mixin(generateGetInfo!(Info, clGetFooInfo));
}

Foo foo  = ...;
int refCount = foo.referenceCount;
```
All the magic is in [`generateGetInfo`](https://github.com/libmir/dcompute/blob/master/source/dcompute/driver/ocl/util.d) to generate a property for each member in `Foo.Info`, enabling much better scalability and bonus documentation.

CUDA also has properties exposed in a similar manner, however they are not essential (unlike OpenCL) for getting things done so their development has been deferred.

Launching a kernel is a large point of pain when dealing with the C API of both OpenCL and (only marginally less horrible) CUDA, due to the complete lack of type safety and having to use the `&` operator into a `void*` far too much. In DCompute this incantation simply becomes

    Event e = q.enqueue!(saxpy)([N])(b_res, alpha, b_x, b_y, N);
for OpenCL (1D with N work items), and

    q.enqueue!(saxpy)([N, 1, 1], [1 ,1 ,1])(b_res, alpha, b_x, b_y, N);
for CUDA (equivalent to `saxpy<<<N,1,0,q>>>(b_res,alpha,b_x,b_y, N);`)

Where `q` is a queue, `N` is the length of buffers (`b_res`, `b_x` & `b_y`) and `saxpy` (single-precision _a x plus y_) is the kernel in this example. A full example may be found [here](https://github.com/libmir/dcompute/blob/master/source/dcompute/tests/main.d), along with the magic that drives the [OpenCL](https://github.com/libmir/dcompute/blob/4182fb8e1b2532adee2c6af3859856cc45cad85e/source/dcompute/driver/ocl/queue.d#L79) and [CUDA](https://github.com/libmir/dcompute/blob/4182fb8e1b2532adee2c6af3859856cc45cad85e/source/dcompute/driver/cuda/queue.d#L60) enqueue functions.


### The future of DCompute


While DCompute is functional, there is still much to do. The drivers still need some polish and user testing, and I need to set up continuous integration. A driver that unifies the different compute APIs is also in the works so that we can be even more cross-platform than the industry cross-platform standard.

Being able to convert [SPIR-V into SPIR](https://www.khronos.org/spir/) would enable targeting `cl_khr_spir`-capable 1.x and 2.0 CL implementations, dramatically increasing the number of devices that can run D kernel code (there’s nothing stopping you using the OpenCL driver for other kernels though).

On the compiler side of things, supporting OpenCL image and CUDA texture & surface operations in LDC would increase the applicability of the kernels that could be written.
I currently maintain a forward-ported fork of [Khronos’s SPIR-V LLVM](https://github.com/KhronosGroup/SPIRV-LLVM) to generate SPIR-V from LLVM IR. I plan to use [IWOCL](http://www.iwocl.org/) to coordinate efforts to merge it into the LLVM trunk, and in doing so, remove the need for some of the hacks in place to deal with the oddities of the SPIR-V backend.


### Using DCompute in your projects


If you want to use [DCompute](https://github.com/libmir/dcompute), you’ll need a recent [LDC](https://github.com/ldc-developers/ldc) built against LLVM with the [NVPTX](https://llvm.org/docs/NVPTXUsage.html) (for CUDA) and/or SPIRV (for OpenCL 2.1+) targets enabled and should add `"dcompute": "~>0.1.0"` to your `dub.json`. LDC 1.4+ releases have NVPTX enabled. If you want to target OpenCL, you’ll need to build LDC yourself against [my fork of LLVM](https://github.com/thewilsonator/llvm/tree/compute).
