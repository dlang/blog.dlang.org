---
author: DBlogAdmin
comments: false
date: 2018-01-20 11:34:56+00:00
excerpt: Last year, Phil Eaton started working on BSDScheme, a Scheme interpreter
  that he ultimately intends to support Scheme R7RS. In college, he had completed
  two compiler projects in C++ for two different courses. One was a Scheme to Forth
  compiler, the other an implementation of the Tiger language from Andrew Appel’s
  ‘Modern Compiler Implementation’ books.
layout: post
link: https://dlang.org/blog/2018/01/20/project-highlight-bsdscheme/
slug: project-highlight-bsdscheme
title: 'Project Highlight: BSDScheme'
wordpress_id: 1324
categories:
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-bsdscheme/
redirect_from: /2018/01/20/project-highlight-bsdscheme/
---

![](http://dlang.org/blog/wp-content/uploads/2018/01/Lambda_lc-200x200.png)Last year, Phil Eaton started working on [BSDScheme](https://github.com/eatonphil/bsdscheme), a [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) interpreter that he ultimately intends to support [Scheme R7RS](https://bitbucket.org/cowan/r7rs-wg1-infra/src/default/R7RSHomePage.md?fileviewer=file-view-default). In college, he had completed two compiler projects in C++ for two different courses. One was a Scheme to Forth compiler, the other an implementation of the Tiger language from Andrew Appel’s [‘Modern Compiler Implementation’ books](https://www.cs.princeton.edu/~appel/modern/).


I hadn’t really written a complete interpreter or compiler since then, but I’d been trying to get back into language implementation. I planned to write a Scheme interpreter that was at least bootstrapped from a compiled language so I could go through the traditional steps of lexing, parsing, optimizing, compiling, etc., and not just build a language of the meta-circular interpreter variety. I was spurred to action when my co-worker at Linode, [Brian Steffens](https://github.com/briansteffens), wrote [bshift](https://github.com/briansteffens/bshift), a compiler for a C-like language.


For his new project, he wanted to use something other than C++. Though he knows the language and likes some of the features, he overall finds it a “complicated mess”. So he started on BSDScheme using C, building generic ADTs via macros.

As he worked on the project, he referred to Brian’s bshift for inspiration. As it happens, bshift is implemented in D. Over time, he discovered that he “really liked the power and simplicity of D”. That eventually led him to drop C for D.


It was clear it would save me a ton of time implementing all the same data structures and flows one implements in almost every new C project. The combination of compile-time type checking, GC, generic ADT support, and great C interoperability was appealing.


He’s been developing the project on Mac and FreeBSD, using LDC, [the LLVM-based D compiler](https://github.com/ldc-developers/ldc). In that time, he has found a number of D features beneficial, but two stand out for him above the rest. The first is [nested functions](https://dlang.org/spec/function.html#nested).


They’re a good step up from C, where nested functions are not part of the standard and only unofficially supported (in different ways) by different compilers. C++ has lambdas, but that’s not really the same thing. It is a helpful syntactic sugar used in BSDScheme for defining new functions with external context (the names of the parameters to bind).


As for the second, hold on to your seats: it’s the GC.


The existence of a standard GC is a big benefit of D over C and C++. Sure, you could use the Boehm GC, but how that works with threads is up to you to discover. It is not fun to do prototyping in a GC-less language because the amount of boilerplate distracts from the goals. People often say this when they’re referring to Python, Ruby, or Node, but D is not at all comparable for (among) a few reasons: 1) compile-time type-checking, 2) dead-simple C interop, 3) multi-processing support.


Spend some time in [the D forums](http://forum.dlang.org) and you’ll often find newcomers from C and C++ who, unlike Phil, have a strong aversion to garbage collection and are actively seeking to avoid it. You’ll also find replies from long-time D coders who started out the same way, but eventually came to embrace the GC completely and learned how to make it work for them. The D GC can certainly be problematic for certain types of software, but it is a net win for others. This point is reiterated frequently in [the GC series](https://dlang.org/blog/the-gc-series/) on this blog, which shows how to get the most out of the GC, profile it, and mitigate its impact if it does become a performance problem.

As Phil learned the language, he identified areas for improvement in the D documentation.


Certainly it is advantageous compared to the C preprocessor that there is not an entirely separate language for doing compile-time macros, but the behavior difference and transition guides are missing or poorly written. A comparison between D templates and C++ templates (in all their complexity) could also be a great source of explanation.


We’re always looking to improve the documentation and make it more friendly to newcomers of all backgrounds. The docs are often written by people who are already well-versed in the language and its ecosystem, making blind spots somewhat inevitable. Anyone in the process of learning D is welcome and encouraged to help improve both the [Language](https://dlang.org/spec/spec.html) and [Library](https://dlang.org/phobos/index.html) docs. In the top right corner of each page are two links: “Report a bug” and “Improve this page”. The first takes you to [D’s bug tracker](https://issues.dlang.org) to report a documentation bug, the second allows anyone with a logged-in GitHub account to quickly fork [dlang.org](https://dlang.org/), edit the page online, and submit a pull request.

In addition to the ulitmate goal of supporting Scheme R7RS, Phil plans to add [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) support in order to allow BSDScheme to call D functions directly, as well as support for D threads and an LLVM-based backend.

Overall, he seems satisfied with his decision to move to D for the implementation.


I think D has been a good time investment. It is a very practical language with a lot of the necessary high-level aspects and libraries for modern development. In the future, I plan to dig more into the libraries and ecosystem for backend web systems. Furthermore, unlike with C or C++, so far I’d feel comfortable choosing D for projects where I am not the sole developer. This includes issues ranging from prospective ease of onboarding to long-term performance and maintainability.


A big thanks to Phil for taking the time to contribute to this post (be sure to check out [his blog](http://notes.eatonphil.com/first-few-hurdles-writing-a-scheme-interpreter.html), too). We’re always happy to hear about new projects in D. [BSDScheme](https://github.com/eatonphil/bsdscheme) is released under [the three-clause BSD license](https://github.com/eatonphil/bsdscheme/blob/master/LICENSE.md), so it’s a great place to start for anyone looking for an interesting open-source D project to contribute to or learn from. Have fun!
