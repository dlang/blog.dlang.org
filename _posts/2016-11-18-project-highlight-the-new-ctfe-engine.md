---
author: DBlogAdmin
comments: false
date: 2016-11-18 10:52:50+00:00
layout: post
link: https://dlang.org/blog/2016/11/18/project-highlight-the-new-ctfe-engine/
slug: project-highlight-the-new-ctfe-engine
title: 'Project Highlight: The New CTFE Engine'
wordpress_id: 491
categories:
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-the-new-ctfe-engine/
redirect_from: /2016/11/18/project-highlight-the-new-ctfe-engine/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png) CTFE ([Compile-Time Function Execution](https://en.wikipedia.org/wiki/Compile_time_function_execution)) is today a core feature of the D Programming Language. D creator Walter Bright first implemented it in DMD as an extension of the [constant folding logic](http://www.compileroptimizations.com/category/constant_folding.htm) that was already there. Don Clugston (of [FastDelegate](http://www.codeproject.com/Articles/7150/Member-Function-Pointers-and-the-Fastest-Possible) fame) made a pass at improving it and, according to Walter, "[took it much further](https://dlang.org/blog/2016/08/30/ruminations-on-d-an-interview-with-walter-bright/)". Since that time, usage of CTFE has shown up in one D project after another, including in D's standard library. For example, Dmitry Olshansky employed it in his overhaul of [std.regex](https://dlang.org/phobos/std_regex.html) to great effect.

On the last day of [DConf 2016](http://dconf.org/2016/), Stefan Koch gave a lightning talk on his thoughts about CTFE in D. At the end of the talk, [in response to a question](https://youtu.be/yIH_0ew-maI?t=1h20s) from Andrei Alexandrescu on how D's implementation could be improved, he said the following:


CTFE is really a hack. You can see that it's a hack. It's implemented as a hack. It is the most useful hack that I've ever seen, and it is definitely a hacker's tool to do stuff that are like magic. But to be fast, it would need to be heavily redesigned, reimplemented, possibly executed in multiple threads, because it is used for stuff that we could never have envisioned when it was invented.


Not long after that, Stefan [opened a discussion](https://forum.dlang.org/thread/rnzxxfvmeeytnvkmwcxj@forum.dlang.org) on the fourms and took up the torch to improve the CTFE engine. As to why he got started on this journey in the first place, Stefan says, "I started work on the CTFE engine because I said so at DConf." But, of course, there's more to it than that.


I have pretty heavy-weight CTFE needs (I worked on a compile-time trans-compiler). Also my [CTFE SQLite reader](https://github.com/UplinkCoder/sqlite-d/commits/master?after=GSBqrGVYDAOjVxlBsjyzfNjZFUgrMTA0) is failing if you want to read a database bigger then 2MB at ctfe.


His investigations into the performance of the CTFE interpreter shed light on its problems.


The current interpreter interprets every AST-Node it sees directly. This leaves very little space to collect information about the code that is being interpreted. It doesn't know when something will be used as a reference, so it needs to copy every variable on every mutation. It has to do a deep-copy for this. That means it copies the whole chain of mutations every time.


To clarify, he offers the following example.


Imagine `foreach(i;0 .. 10) { a = i; }`. On the first iteration we save `a` = 0` and set `a``` to `1`. On the second iteration we save `a``` = 1` and `a````= 0` and we set `a````` to 2` , then `a`````` = 1` and `a``````` = 0` and so on. As you can see, the memory requirements just shoot up. It's basically a factorial function with a very small coefficient. That is why for very small workloads this extreme overhead is not noticeable.

That flaw looked unfixable. Indeed the whole architecture in [dinterpret.d](https://github.com/dlang/dmd/blob/master/src/dinterpret.d) is very convoluted and hard to understand. I did a few experiments on improving memory-management of the interpreter but it proved fruitless.


Once he realized there was going to be no quick fix, Stefan sat down and drew up a plan to avoid digging himself into the same hole the current interpreter was in. The result of his planning led him down a road he hadn't expected to travel.


Direct Interpretation was out of the question since it would give the new engine too little time to analyze data-flow and decided whether a copy was really needed or not. I had to implement an Intermediate Representation. It had to be portable to different evaluation back-ends. I ended up with a solution, inspired by OpenGL, of defining my interface in the form of function calls an evaluation back end had to implement. That meant I would not be able to simply modify the current interpreter. This made the start very steep, but it is a decision I do not regret.


His implementation consists of a front end and a back end.


The front end walks the AST and issues calls to the back end. And the back end transforms those calls into actual bytecode. This bytecode is interperted by the back end as soon as the front end requires it.


In terms of functionality, he likens the current implementation to an immediate mode graphics API, and his revamp to retained mode. In this case, though, it's the immediate mode that's the memory hog.

You can read about his progress in the [CTFE Status](https://forum.dlang.org/thread/tdhdsvmknnpiugnngkex@forum.dlang.org) thread, where he has been posting frequent updates. His updates include problems he encounters, features he implements, and performance statistics. Eventually, every compiler that uses the DMD front end will benefit from his improvements.
