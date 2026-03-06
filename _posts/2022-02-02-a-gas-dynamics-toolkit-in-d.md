---
author: QueenslandU
comments: false
date: 2022-02-02 07:57:50+00:00
layout: post
link: https://dlang.org/blog/2022/02/02/a-gas-dynamics-toolkit-in-d/
slug: a-gas-dynamics-toolkit-in-d
title: A Gas Dynamics Toolkit in D
wordpress_id: 3049
categories:
- Guest Posts
- Project Highlights
- User Stories
permalink: /a-gas-dynamics-toolkit-in-d/
redirect_from: /2022/02/02/a-gas-dynamics-toolkit-in-d/
---

The Eilmer flow simulation code is the main simulation program in our [collection of gas dynamics simulation tools](https://gdtk.uqcloud.net/). An example of its application is shown here with the simulation of the hypersonic flow over the BoLT-II research vehicle that is to be flown in 2022.


![BoLT-II simulation with steady-state variant of the Eilmer code. Flow is from bottom-left to top-right of the picture. Only one quarter of the vehicle surface, coloured grey, is shown. Several slices through the flow domain are coloured with the local Mach number, with blue for low values and red for high values. Several streamlines, drawn in black, start at the blunt leading edge of the vehicle and follow the gas flow along the vehicle surface. Image produced by Kyle Damm.]({{ '/assets/images/a-gas-dynamics-toolkit-in-d/bolt_II_example_visual_16092021.png' | relative_url }})

_BoLT-II simulation with steady-state variant of the Eilmer code. Flow is from bottom-left to top-right of the picture. Only one quarter of the vehicle surface, coloured grey, is shown. Several slices through the flow domain are coloured with the local Mach number, with blue for low values and red for high values. Several streamlines, drawn in black, start at the blunt leading edge of the vehicle and follow the gas flow along the vehicle surface. Image produced by Kyle Damm._





### Some history



This simulation program, originally called `cns4u`, started as a relatively small C program that ran on the Cray-Y/MP supercomputer at NASA Langley Research Center in 1991. A PDF of an early report with the title 'Single-Block Navier-Stokes Integrator' [can be found here](https://apps.dtic.mil/sti/citations/ADA240385). It describes the simple finite-volume formulation of the code that allows simulation of a nonreacting gas on a single, structured grid. Thirty years on, many capabilities have been added through the efforts of a number of academic staff and students. These capabilities include high-temperature thermochemical effects with reacting gases and distributed-memory parallel simulations on cluster computers. The language in which the program was written changed from C to C++, with connections to Tcl, Python and Lua.

The motivation for using C++ in combination with the scripting languages was to allow many code variations and user programmability so that we could tackle any number of initially unimagined gas-dynamic processes as new PhD students arrived to do their studies. By 2010, the Eilmer3 code (as it was called by then) was sitting at about 100k lines of code and was growing. We were, and still are, mechanical engineers and students of gas-dynamics first and programmers second. C++ was a lot of trouble for us. Over the next 4 years, C++ became even more trouble for us as the Eilmer3 code grew to about 250k lines of code and many PhD students used it to do all manner of simulations for their thesis studies.

Also in 2010, a couple of us ([PAJ and RJG](https://gdtk.uqcloud.net/docs/introduction/dev-team-and-contributors/)) living in different parts of the world (Queensland, Australia and Virginia, USA) came across the D programming language and took note of Andrei Alexandrescu’s promise of stability into the future. Here was the promise of a C++ replacement that we could use to rebuild our code and remain somewhat sane. We each bought [a copy of Andrei’s book](https://wiki.dlang.org/Books) and experimented with the D language to see if it really was the _C++-done-right_ that we wished for. One of us still has the copy of the initial printing of Andrei’s book without his name on the front cover.



### Rebuilding in D



In 2014 we got serious about using D for the next iteration of Eilmer and started porting the core gas dynamics code from C++ to D. Over the next four years, in between university teaching activities, we reimplemented much of the Eilmer3 C++ code in D and extended it. We think that this was done to good effect. [This conference paper](https://www.scientific.net/AMM.846.54), from late 2015, documents our effort at the initial port of the structured grid solver. ([A preprint is hosted on our site](https://gdtk.uqcloud.net/pdfs/T0316-eilmer-dlang-v2.pdf).) The Eilmer4 program is as fast as the earlier C++ program but is far more versatile while being implemented in fewer lines of code. It now works with unstructured as well as structured grids and has a new flexible boundary condition model, a high-temperature thermochemistry module, and in the past two years we have added the Newton-Krylov-accelerated steady-state solver that was used to do the simulation shown above. And importantly for us, with the code now being in D, we now have have many fewer _WTF_ moments.

If you want more details on our development of the Eilmer4 code in D, we have the [slides from a number of presentations](https://gdtk.uqcloud.net/docs/pub-n-pres/presentations/) given to the [Centre for Hypersonics](https://mechmining.uq.edu.au/research/hypersonics) over the past six years.



### Features of D that have been of benefit to us include:







  * Template programming that other Mechanical Engineers can understand (thanks Walter!). Many of our numerical routines are defined to work with `numbers` that we define as an alias to either `double` or `Complex!double` values. This has been important to us because we can use the same basic update code and get the sensitivity coefficients via finite differences in the complex direction. We think this saved us a large number of lines of code.



  * String mixins have replaced our use of the M4 preprocessor to generate C++ code in Eilmer3. We still have to do a bit of head-scratching while building the code with mixins, but we have retained most of our hair—something that we did not expect to do if we continued to work with C++.



  * Good error messages from the compiler. We often used to be overwhelmed by the C++ template error messages that could run to hundreds of lines. The D compilers have been much nicer to us and we have found the “did you mean” suggestions to be quite useful.



  * A comprehensive standard library in combination with language features such as delegates and closures that allow us to write less code and instead concentrate on our gas dynamics calculations. I think that having to use C++ Functors was about the tipping point in our 25-year adventure with C++.



  * Ranges and the foreach loops make our D code so much tidier than our equivalent C++ code.



  * Low-barrier shared-memory parallelism. We do many of the flow update calculations in parallel over blocks of cells and we like to take advantage of the many cores that are available on a typical workstation.



  * Simple and direct linkage to C libraries. We make extensive use of Lua for our configuration and do large simulations by using many processors in parallel via the OpenMPI library.



  * The garbage collector is wonderful, even if other people complain about it. It makes life simpler for us. For the input and output, we take the comfortable path of letting the compiler manage the memory and then tell the garbage collector to tidy up after us. Of course, we don’t want to overuse it. @nogc is used in the core of the code to force us not to generate garbage. We allocate much of our data storage at the start of a simulation and then pass references to parts of it into the core functions.



  * Fast compilation and good optimizing compilers. Nearly an hour’s build time was fairly common for our old C++ code, and now we would expect a DMD or LDC debug build in about a quarter of a minute. This builds a basic version of the main simulation code on a Lenovo ThinkPad with Core i7 processor. An optimized build can take a little over a minute but the benefit of the faster simulation is paid back by orders of magnitude when a simulation job is run for several hours over hundreds of processors.



  * `version(xxxx) { ... }` has been a good way to have variants of the code. Some use complex numbers and others are just double numbers. Also, some variants of the code can have multiple chemical species and/or just work with a single-species nonreacting gas. This reduction in physical modelling allows us to reduce the memory required by the simulation. For big simulations of 3D flows, the required memory can be on the order of hundreds of gigabytes.



  * `debug { ... }` gets used to hide IO code in `@nogc` functions. If a simulation fails, our first action is often to run the debug-flavor of the code to get more information and then, if needed, run the debug flavor of the code under the control of `gdb` to dig into the details.






We have a very specialized application and so don’t make use of much of the software ecosystem that has built up around the D language. For a build tool, we use `make` and for an IDE, we use emacs. The D major mode is very convenient.

There are lots of other features that just work together to make our programming lives a bit better. We are six years in on our adventure with the D programming language and we are still liking it.
