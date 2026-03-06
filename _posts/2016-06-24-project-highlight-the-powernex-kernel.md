---
author: DBlogAdmin
comments: false
date: 2016-06-24 14:04:56+00:00
layout: post
link: https://dlang.org/blog/2016/06/24/project-highlight-the-powernex-kernel/
slug: project-highlight-the-powernex-kernel
title: 'Project Highlight: The PowerNex Kernel'
wordpress_id: 52
categories:
- Project Highlights
permalink: /project-highlight-the-powernex-kernel/
redirect_from: /2016/06/24/project-highlight-the-powernex-kernel/
---

Hang around the D community long enough and you'll discover that people are using the language in a variety of fields for a variety of projects, both professionally and personally. Computer games, scientific software, web apps, economic modelling, even scripted utilities. There have even been a few open source kernel projects using D, the most recent of which is [PowerNex by Dan Printzell](https://github.com/Vild/PowerNex).

As hobby projects go, an OS kernel is one of the more complex projects a programmer could tackle. It requires a certain level of motivation and dedication that isn't needed for other types of projects. Dan insists that it's "fun, rewarding and hard, like big projects should be."


I have always been interested in OS development and have been trying to write my own OS for years now. Each attempt was written in C, but none of them worked well because I mostly just copy-pasted code with no real knowledge of how it worked. Back in November 2015, I decided to start writing yet another kernel, but this time in D. I also challenged myself to make it 64-bit. The reason I chose D is simple. I love the language. I love that you can write nice looking code with the help of string mixins and templates and that the code can interface easily with C.


The D programming language ships with a runtime, appropriately named [DRuntime](https://github.com/dlang/druntime), which manages the garbage collector, ensures static constructors and destructors are called, and more. Some language features depend on the runtime being present. When developing an OS kernel, making use of the full runtime is not an option. Dan took the minimal D runtime for bare metal D development that Adam Ruppe described in his book, the [D Cookbook](http://amzn.to/1ZTE47m), and used that as the basis for his kernel.


It is still pretty much the same thing Adam provided, with some patches to fix deprecated stuff and to connect it to the rest of the kernel.


It hasn't all been a walk through the roses, though. By default, variables in D are in Thread Local Storage (TLS). In order to force variables to become globally shared across threads, they must be marked as either `shared` or `__gshared`. The former is intended to tell the compiler to restrict certain operations on the variable (you can read about it in [the freely available concurrency chapter](http://www.informit.com/articles/article.aspx?p=1609144) from Andrei Alexandrescu's book, [The D Programming Language](http://amzn.to/1ZTDmqH)). The latter essentially causes the compiler to treat it as a global C variable, with no guarantees and no protection. Normally, TLS variables are a good thing for D programs, but not when starting out in the early stages of kernel development.


The biggest problem I've encountered is that the compiler expects that TLS is enabled, which I haven't done yet, so I need to append __gshared to all the global variables. If I don't write __gshared, the kernel will try and access random memory addresses and do undefined stuff. Sometimes it crashes, sometimes it doesn't. This is the thing that is most often behind PowerNex bugs.


Did I mention that Dan loves D's [string mixins](https://dlang.org/spec/function.html#string-mixins) and [templates](https://dlang.org/spec/template.html)?


String mixins and templates are the best thing in the language. Without these I would probably write the kernel in C instead. One place where they are used is in the Interrupt Service Routines (ISR) handler. The problem with the ISRs is that they don't provide their ID to the handler. So I need to make 256 different functions just to know which ISR was triggered. This could be really error prone, but with some help from templates and string mixins, [I can generate those](https://github.com/Vild/PowerNex/blob/master/Kernel/src/CPU/IDT.d) and be sure that the content for each function is correct.


To compile PowerNex, Dan uses a cross-compiled GNU Binutils, a patched version of DMD, and his own build system, called [Wild](https://github.com/Vild/Wild).


The GNU Binutils is for compiling the assembly files and for linking the final executable. [The patch for DMD](https://github.com/PowerNex/dmd/commits/PowerNexCompiler) that I currently use basically just adds PowerNex as a target and as a [predefined version](https://dlang.org/spec/version.html#predefined-versions) (which is active when compiling). It is really hackily implemented because I'm not too familiar with the DMD source code. I want to implement these better and get it upstream in the future when I will be able to compile userspace programs.

The build system is not that much to look at currently. It is written in D and uses a JSON file as a frontend to define a set of file processors, rules and targets. With the help of these, Wild can compile PowerNex. I'm currently working on conversion from JSON to a custom format to be able to provide the features needed for the compilation of the kernel and all its userspace programs.


He has a few specific goals in mind before he's ready to brand a PowerNex 1.0 release.


One of my first short term goals is to be able to run a simple ELF executable. Next, I want to port druntime and phobos; once I have that done I will be able to run almost any D program natively. Finally, I will port either DMD or SDC (the [Stupid D Compiler](https://github.com/SDC-Developers/SDC)), depending on what state SDC is in when I get there.


You can see a couple of screenshots of PowerNex in action via [a post from one D community member](http://forum.dlang.org/post/cgtvuamxqgcwxeucjbjb@forum.dlang.org) in Dan's forum announcement thread. If the idea of kernel development with D gives you goosebumps, [go have some fun](https://github.com/Vild/PowerNex)!
