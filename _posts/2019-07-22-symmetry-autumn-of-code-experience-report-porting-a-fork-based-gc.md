---
author: FrancescoMecca
comments: false
date: 2019-07-22 13:58:53+00:00
layout: post
link: https://dlang.org/blog/2019/07/22/symmetry-autumn-of-code-experience-report-porting-a-fork-based-gc/
slug: symmetry-autumn-of-code-experience-report-porting-a-fork-based-gc
title: 'Symmetry Autumn of Code Experience Report: Porting a fork-based GC'
wordpress_id: 2148
categories:
- Community
- SAoC
permalink: /symmetry-autumn-of-code-experience-report-porting-a-fork-based-gc/
redirect_from: /2019/07/22/symmetry-autumn-of-code-experience-report-porting-a-fork-based-gc/
---

![Symmetry Investments logo](http://dlang.org/blog/wp-content/uploads/2018/09/logo-1.png)The 2018 edition of the [Symmetry Autumn of Code](https://dlang.org/blog/symmetry-autumn-of-code/) was a wonderful opportunity for me and two other students to dive into an interesting programming challenge and contribute to the D community. I am going to describe the process that led to my participation in SAOC and what this four months of work meant to me.


## Who I am


I am an MSc student in Computer Science at the University of Turin in Italy. My interests mainly revolve around type systems, language theory, formal languages and compilers, and concurrent programming techniques. I am 23 years old and I have been programming since I was 20.


## What happened before SAOC


While browsing Hacker News one day, [one comment caught my attention](https://news.ycombinator.com/item?id=14791888): “the [D] community seems second to none as far as signal to noise ratio goes”. From that day I started lurking on the D forums and later read Andrei Alexandrescu’s book, [The D Programming Language.](http://www.informit.com/articles/article.aspx?p=1381876) While I think it alone wasn’t sufficient to learn the language, it was worth the read for all the discussions about compiler internals, the compromises about the syntax, and the details on ranges and [concurrency by message passing](http://www.informit.com/articles/article.aspx?p=1609144).

After reading the book, I had enough confidence to start exploring some real-world code and I decided to dig into the codebases of [the D standard library (Phobos)](https://github.com/dlang/phobos) and [vibe.d, a popular web app framework](https://vibed.org/). My colleague Francesco Gallà, who also would participate in SAOC (see [his DConf 2019 presentation about his experience](https://www.youtube.com/watch?v=nk9gQWRvoXM&feature=youtu.be)), started learning D and we approached the community together by discussing one of the past Google Summer of Code proposals, HTTP2 support, on [the vibe.d forum](http://forum.rejectedsoftware.com/groups/rejectedsoftware.vibed/).

At the same time, I bought the [Garbage Collection Handbook](http://gchandbook.org/) as a personal reading. At the time it seemed totally unrelated to what I was doing with D.


## SAOC


On July 14, 2018, the D Language Foundation announced the Symmetry Autumn of Code. It seemed the right occasion to boost my skills with D and get even more involved with the community. I was also thrilled by the possibility of learning new methodologies for writing code with a remote mentor.

I gathered information about a past project, a concurrent garbage collector that was used in D1 in [the Tango library](http://www.dsource.org/projects/tango). I decided that my SAOC project proposal would be to port it to D2 with the goal of incorporating it [into the D runtime](https://github.com/dlang/druntime).

The original work was done by Leandro Lucarella as his master thesis, and he [documented it extensively on his blog](https://llucax.com/blog/blog/tag/cdgc). I contacted him and received confirmation that he could mentor such a project. After that, I wrote my proposal. It consisted of two documents, one about me and the other about the project.


## The concurrent GC project


The project document had the objective of showing my efforts in understanding the garbage collector currently shipping with the D runtime—a conservative [mark and sweep](https://www.geeksforgeeks.org/mark-and-sweep-garbage-collection-algorithm/) GC that does the following whenever an allocation is issued and memory needs to be reclaimed:



 	
  1. stop all threads except the current one

 	
  2. “hijack” the work of the current thread in order to run the GC routines

 	
  3. start scanning recursively every root memory pointer in order to find every memory block that has been used or is being used by the program

 	
  4. mark all GC-allocated blocks that are no longer referenced

 	
  5. resume all other threads

 	
  6. run destructors from unreachable memory that has been marked in phase 4 and free the remaining unreachable memory

 	
  7. continue execution of the current thread.


The program is paused during steps 1 to 4 (the mark phase) and memory is reclaimed during step 6, where the GC thread hijacks the flow of the thread that triggered the collection. There are various approaches for reducing the pause time, such as using threads to scan and mark the memory objects in parallel; [work is being done in that direction](https://github.com/dlang/druntime/pull/2514).

My proposal highlighted another strategy. By sacrificing memory consumption on systems that support Posix’s `fork()` system call, pause time could be drastically reduced. A fork-based concurrent GC would be represented by the following sequence of routines:



 	
  1. a thread triggers the GC collection (such a thread is called the mutator)

 	
  2. the GC clones the address space of the current program with the `fork()` system call

 	
  3. while the parent process continues execution, the child process starts the mark phase

 	
  4. at the end of the mark phase, before exiting, the child process communicates the marked bits (that can be reused) to the parent

 	
  5. the GC uses the unmarked blocks for future allocations (lazy sweep).


With this schema, the pause time of the marking phase is reduced to the duration of the `fork()` call only. There are many advantages, such as the fact that the other threads do not need to be stopped and that the reclaim phase could run concurrently without hijacking the mutator. In particular, this strategy shows its strength in potentially long-running programs that have large heaps with a high number of live objects.

The implementation resulted in a bit more than 500 lines of code, given that the calling process (called the parent) generates a duplicate of itself in a different address space. This removes the need for synchronization, which has a high overhead both in terms of runtime performance and code implementation. Moreover, Unix and Linux systems provide very efficient `fork()` implementations with [the use of COW](https://en.wikipedia.org/wiki/Copy-on-write) memory.


## Methodologies


The first thing that I did with Leandro was to correct the milestones that I had predicted. Based on his experience, we put a bigger focus on defining the tests rather than the programming. After that he specified the workflow to me. We setup a test suite in place, mainly [composed of dustmite](https://github.com/CyberShadow/DustMite/wiki), the runtime benchmarks and tests, and some D1 programs that I ported.

After discussing and applying every change to the codebase in a different branch, I had to run the tests and then open a pull request on my GitHub repository asking for a review from Leandro. Commits had to be very granular and he always provided a lot of feedback. He was always prompt in replying and we had a number of exchanges by email before applying a change. Many times we discussed benchmarks and regressions, and sometimes I asked for help with debugging. I can confidently say that I spent more time researching, reading, debugging, and discussing code than I spent writing it.

I was alone in managing my time and my commitment. Leandro and the D Foundation were always present in discussing things by email, but they didn’t force any time sheet on me nor did they micromanage my work.


## The end of SAOC


At the end of the four months of SAOC, I had a working implementation, but I decided to delay a pull request to the D runtime in order to work on some profiling code that could help developers understand in which cases the fork-based garbage collector brings advantages. After a precise garbage collector was [announced for DMD 2.085.0](https://dlang.org/changelog/2.085.0.html#gc_precise), I decided to adapt my work to it. It was very straightforward given the clarity of the added code and the separation of concerns in place.

Leandro was available for mentoring even after SAOC. We exchanged many emails and he showed me how the D garbage collector is being profiled [at Sociomantic](https://www.sociomantic.com/).


## The pull request


A pull request to the D runtime was my final milestone. I was ready at the beginning of February, but I started to procrastinate. I'd had no previous communication with any of the reviewers and I was timorous about engaging with them. I spent a lot of time refactoring my code back and forth and delaying my pull request. At a certain point, I even considered abandoning the final milestone and providing the GC as a library. In the meantime, Rainer Scheutze published a threaded implementation of the mark phase that reduced the mark time in the GC and I lost faith in my project.

Luckily for me, I had the opportunity to attend [DConf 2019 in London](https://dconf.org/2019/index.html). There I found many great people who talked with me and convinced to resume my work. I had a brief discussion with Rainer and I started testing against his implementation (I also found a related bug in the CPU detection code) and on the last day, during [the annual Hackathon](http://dconf.org/2019/talks/hackathon1.html), I finally [opened the pull request](https://github.com/dlang/druntime/pull/2604).

Since May 11, I have discussed changes to my code, reduced the number of lines of code, refactored and collapsed some functions, and resolved bugs related to program termination. The pull request is still open given that there were many rough edges, but overall I am very satisfied about the feedback received and with the whole process. Reviewers responded to every commit and provided guidance when needed. Whenever something wasn’t clear, I replied on GitHub or asked for help in [the #D IRC on freenode.net](irc://freenode.net/D).

In retrospect, I should have opened the PR much earlier and presented the reviewers with every doubt that I had along the path.


## Experience analysis


SAOC helped me in understanding the dynamics behind contributing to a community project such as D. I was already spending a fixed amount of time reading the forum every day and I started to direct some of that effort to GitHub. Checking pull requests, commits, and issues was expensive work, but it was necessary to gain knowledge about the methodologies regarding the development of Phobos and the runtime.

I also learned that there are many changes that go unnoticed if you don’t closely follow the discussions on the PR queue. One example [is Manu Evans’s work](https://github.com/dlang/druntime/pulls?q=is%3Apr+author%3ATurkeyMan+is%3Aclosed) on `core.stdcpp` that is very difficult to follow when it’s scattered across different forum threads and lacks a proper announcement.

I think that overall, communication was a weak point of my experience. Regarding SAOC, we defined the work in detail but we didn’t put any effort on communicating the status of my work to the community. That could have triggered helpful suggestions from the community that could have helped me to discover holes in my implementation. Again, I benefited from this great amount of freedom in managing my work, but I think it could have coexisted with some effort on communicating more.

Finally, I had previously used D for higher level code, but SAOC forced me to discover D from a new angle; I dived into low-level code. I was amazed by the flexibility of the language and found myself becoming familiar with this different style of writing code very quickly.


## Conclusion


The Symmetry Autumn of Code was a very rewarding experience. It was a period of discovery, self-growth, and involvement in a new community.

I am very grateful to [Symmetry Investments](http://symmetryinvestments.com/) and the D Language Foundation for this opportunity. Moreover, I want to thank my mentor, Leandro, for all of his help and for all the positive exchanges that we had.
