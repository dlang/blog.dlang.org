---
author: Joakim
comments: false
date: 2018-12-04 14:09:30+00:00
excerpt: Matrix, the world’s fastest file system, was written in D and recently posted
  impressive numbers in the IO-500 Node Challenge. It was created by WekaIO, a San
  Jose, CA, based startup with engineering in Tel Aviv, Israel. Liran Zvibel, the
  co-founder and CEO of WekaIO, has been a regular speaker at DConf, talking about
  their use of D at DConf 2015, 2016, and 2018.
layout: post
link: https://dlang.org/blog/2018/12/04/interview-liran-zvibel-of-wekaio/
slug: interview-liran-zvibel-of-wekaio
title: Liran Zvibel of WekaIO on using D to Create the World's Fastest File System
wordpress_id: 1827
categories:
- Companies
- Interviews
permalink: /interview-liran-zvibel-of-wekaio/
redirect_from: /2018/12/04/interview-liran-zvibel-of-wekaio/
---

![]({{ '/assets/images/interview-liran-zvibel-of-wekaio/weka-300x109.png' | relative_url }})[Matrix, the world's fastest file system](https://www.theregister.co.uk/2018/03/22/spec_filer_benchmark_anoints_worlds_fastest_file_system/), was written in D and recently [posted impressive numbers in the IO-500 Node Challenge](https://www.theregister.co.uk/2018/11/30/wekaio). It was created by [WekaIO](https://www.weka.io), a San Jose, CA, based startup with engineering in Tel Aviv, Israel. Liran Zvibel, the co-founder and CEO of WekaIO, has been a regular speaker at DConf, talking about their use of D at DConf [2015](https://dconf.org/2015/talks/zvibel.html), [2016](https://dconf.org/2016/talks/zvibel.html), and [2018](https://dconf.org/2018/talks/zvibel.html).

WekaIO is an expression of a design goal of D, that [you can write your prototype quickly and easily in D, then continue working on the same codebase until it reaches production quality](https://dconf.org/2015/talks/smith.html), as opposed to prototyping in a different high-level programming language. Liran took some time out of his busy schedule to answer some questions about WekaIO and their use of D.

_Liran Zvibel at DConf 2018_



**Joakim**: Tell us about the enterprise storage market that WekaIO is in. How do you use D in your product?

**Liran**: Any compute environment has a mix of CPU power, networking, and storage: this is uniform across many organization types and sizes, and whether the infrastructure is in-house (“on premises”) or in the public cloud (“The cloud is just somebody else’s computer,” as they say).

The storage component provides the current state and also the history needed by compute. While compute is stateless (how many times have you rebooted your computer to fix a problem?) and the network is ephemeral, storage must be able to keep its state consistently and coherently while providing enough performance for the compute to do its job (otherwise the running jobs are IO-bound, and nobody likes that).

There are three main types of centralized storage systems:




  * Block storage systems, that provide the abstraction of a local drive that sits remotely. Several systems may get access to their own “volume” on the centralized system. The AWS equivalent for such a system is Elastic Block Store. These volumes are usually not shared between more than one server, and the reason to use them is failure resiliency, reliability, and performance (also some advanced features such as taking a point in time, backup, integration into a VM environment, etc.).

  * File systems, centralized storage that is also shared, and allows several
servers on the network to access the same data. This is the kind of system
that WekaIO provides. Traditionally, people have turned to block-based solutions if they needed high performance, then created a local file system based on that shared volume, but with WekaIO we show that we enable a shareable file system that is even faster than a local file system over block storage.

  * Object storage solutions, these enable storing objects with reduced semantics (no ability to modify, data stored is only eventually consistent, etc) to enable cost savings, and generally don’t care about performance.



When I review storage systems here, I talk mainly about block-based and file system solutions as object storage is much simpler and is implemented
using different methods.

Requirements for storage systems are:


  * Reliability

  * Performance (low-latency IOPS and throughput)

  * Features



Traditionally, these systems have a “data path” that cares about reliability and performance, and then a “control path” or “management path” that takes care of higher-level features and making reliability work at a higher level.

For many storage systems, the data path is implemented in some system programming language, such as C or C++, and the management path is implemented in a higher-level language such as Python, Java, Go, etc. Our previous company, XIV (that IBM acquired in 2007) used a private version of C that had polymorphism, generic programming, and integrated RPC, for code that was a mixture of XML, C, and weird header files.

At WekaIO, we use the D programming Language to implement both the data path and the control path, as we can use a single language to get both machine-level access with high performance when needed, and a higher-level language for the features.

**Joakim**: How did you end up choosing D? You mentioned at DConf that you
initially tried a combination of Python and C++.

**Liran**: The first part of the work we had done on the C++/Python combo was to work on an efficient in-process RPC mechanism between Python and C++, that would
allow us to refer to the same object from C++ and Python in an efficient way. That way the same object could have run in the C++ context when needing performance, and in Python when needing brevity. The idea was to start implementing the system in Python then convert pieces to C++ as needed, where inter-process communication (inter or intra servers) would be done in C++ only. At that time, we had a prototype of the system implemented in Python and we started working on the C++ part, and the RPC definition first.

When we discovered D, we realized we may be able to get a single language to handle both the high-level and performance requirements, and we started by running some pet projects to verify that this was indeed a working language. The next phase was implementing our RPC over D (which was much more elegant than the C++ version), and a tracing system that would allow us to debug (the tracing system was reviewed in my DConf 2015 talk).

The biggest limitation we had initially was the availability of an optimizing compiler, as the reference compiler, DMD, does not provide assembly that is equivalent to LLVM or even GCC when running on modern x86 processors. After DConf 2015, and with the help of David Nadlinger, we were able to get [LDC (the D compiler with an LLVM backend)](https://github.com/ldc-developers/ldc/) to compile our code and generate results that met our needs.

**Joakim**: Andrei Alexandrescu, one of the co-architects of the D language, [blogged about visiting you in Tel Aviv](https://dlang.org/blog/2017/05/22/introspection-introspection-everywhere/): how your code is very compact and that you added new features without growing the codebase much. What key features of D do you use to accomplish this and the speed and other benefits of your storage system?

**Liran**: The strongest part of D is its generic programming. We use generic programming in two ways, that usually are contradictory but with D they actually work well. One way we use generic programming is to achieve higher performance, some examples:




  * Using static polymorphism, object types get fully assembled by the compiler and runtime runs the correct code.

  * In many cases, passing compile-time arguments saves expensive memory loads and branches leading to much faster execution.

  * Compile-time introspection allows placing objects in memory differently, and also makes code run faster based on static decisions.



Then we use [`static if`](https://tour.dlang.org/tour/en/gems/template-meta-programming), [generics](https://tour.dlang.org/tour/en/basics/templates), [`foreach`](https://tour.dlang.org/tour/en/basics/foreach) and [introspection](https://tour.dlang.org/tour/en/gems/traits) to allow us to write higher-level code much easier, and also write code once that applies for many cases ([watch Andrei’s 2017 DConf keynote - Design by Introspection](https://youtu.be/29h6jGtZD-U)).

We probably use most of the language features: including [User-Defined Attributes](https://tour.dlang.org/tour/en/gems/attributes), to mark code that our introspection chooses to handle differently later or just for debugging purposes; [the built-in unit tests](https://tour.dlang.org/tour/en/gems/unittesting);
obviously [ranges (these are clever!)](https://tour.dlang.org/tour/en/basics/ranges); and [even some contract programming](https://tour.dlang.org/tour/en/gems/contract-programming).

**Joakim**: D is not that widely known yet, how do you bring developers on board? Any common hangups?

**Liran**: Our internal environment has a very proprietary form to it, with dependencies and a build environment that is unique to us, but the biggest grief we get from users trying to start using D (even WekaIO employees that try to run some pet projects) is that the first few minutes and out-of-the-box experience with [DUB as a package manager and build environment](https://code.dlang.org) are not streamlined enough. We often get comparisons to other upcoming languages where the on-boarding process is easier and more inviting to new users to write the second project after they compiled the obligatory “hello, world!” program.

Newcomers are usually unaware of D features, but as our code is so full of them, they adapt to it extremely quickly during their welcome project, so it’s not a problem at all.

**Joakim**: You mention contracting some LDC devs to do work for you. What kinds of fixes have they had to put in and could you talk more generally about the flaws in D
that you've had to solve or work around?

**Liran**: The biggest problem we had initially (over 3 years ago) was that LDC did not even compile our code. The compiler choked on it and failed. The first few iterations were just David getting the compiler to compile. Then after we got a running binary, we had a long series of changes to make sure that the code generated by LDC worked the same way as the DMD-generated code, then we got into adding performance improvements to our code and to LDC itself. The WekaIO codebase is larger than the standard project, and the current LDC that we use ([which is available on GitHub as a fork](https://github.com/weka-io/ldc/releases)) contains some template instantiation changes that are not part of the standard frontend.

Johan Engelen runs our current LDC efforts and has spent a lot of time on link-time optimization and performance-guided optimizations to get LDC and our code to run faster. Now, Johan works on running our code and tests before advancing our LDC to the next frontend releases, to make sure things still run well. We care deeply about performance, and also about the binary representation of our data structures, and some of the work Johan does each release is to verify that they don’t change.

The biggest issue with D to work around is reliance on the Garbage Collector (GC) throughout the runtime and standard library, Phobos. We have had to work on making sure we can leverage exceptions with no GC, and had to create [our own non-GC standard library](https://github.com/weka-io/mecca), as we cannot rely on Phobos for it.

We were able to get our LDC and libraries to a state that we have a very efficient runtime environment that produces extremely fast code (with no GC jitter). Our only grief is around compilation time for our large project.

**Joakim**: You've talked about sponsoring D meetups in Tel Aviv publicizing the D language. What kind of reception have you had? More generally, Tel Aviv is famous for being a startup hotbed: how does WekaIO find being situated there?

**Liran**: Tel Aviv and Israel are fertile ground for startups. We have run some D meetups and one of them even had [Andrei Alexandrescu giving a talk at the Google Campus](https://youtu.be/es6U7WAlKpQ) (our offices were not big enough to fit all the people coming). The reception of the concepts is very good, but unfortunately, we were not able to get other startups to actually start using the D Language. Having a startup in Tel Aviv is both a blessing and a curse, as there is a lot of very strong talent with a lot of experience, especially around infrastructure, but also there is a lot of demand. Alongside the startups, there are many large corporations (Google, Apple, Microsoft, Amazon, and many more) with engineering offices offering very lucrative positions, so even though there is a lot of good “supply,” the “demand” is even stronger and filling positions is not easy.



* * *



_Joakim is the resident interviewer for the D Blog. He has also [interviewed](http://arsdnet.net/this-week-in-d/mar-15.html) [members](http://arsdnet.net/this-week-in-d/jun-28.html) [of the](http://arsdnet.net/this-week-in-d/jul-12.html) [D community](http://arsdnet.net/this-week-in-d/sep-06.html) for [This Week in D](http://arsdnet.net/this-week-in-d/) and is responsible for [the Android port of LDC](https://wiki.dlang.org/Build_D_for_Android)._
