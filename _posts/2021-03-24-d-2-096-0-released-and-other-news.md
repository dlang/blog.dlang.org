---
author: DBlogAdmin
comments: false
date: 2021-03-24 12:13:14+00:00
layout: post
link: https://dlang.org/blog/2021/03/24/d-2-096-0-released-and-other-news/
slug: d-2-096-0-released-and-other-news
title: D 2.096.0 Released and Other News
wordpress_id: 2815
permalink: /d-2-096-0-released-and-other-news/
redirect_from: /2021/03/24/d-2-096-0-released-and-other-news/
---

![Digital Mars D logo]({{ '/assets/images/d-2-096-0-released-and-other-news/d6.png' | relative_url }})

The latest version of DMD, the D reference compiler, [is now available for download](https://dlang.org/download.html). [The changelog notes](https://dlang.org/changelog/2.096.0.html) 17 major changes and 81 resolved Bugzilla issues [from 54 contributors](https://dlang.org/changelog/2.096.0.html#contributors). After we get into some notable items from the changelog, we'll turn our attention to other items of note from the D community: a new release of LDC is right around the corner and one of GDC just beyond the horizon; the Symmetry Autumn of Code wrapped up with a surprise ending; and there are two sometimes forgotten places where anyone can go to find ways to contribute and sharpen their D coding skills.



## DMD 2.096.0



This release of DMD is one of those where so many of the improvements are highlight-worthy that it's difficult to decide which ones to focus on: there are more improvements to [the experimental C++ header generation](https://dlang.org/changelog/2.096.0.html#dtoh-improvements) like those I highlighted in the previous release; [support for DWARF debug info](https://dlang.org/changelog/2.096.0.html#dwarf-switch) is surely important to a significant segment of the D community; and changing plain `synchronized` statements [to use runtime-allocated mutexes](https://dlang.org/changelog/2.096.0.html#runtime-synchronized) fixes a critical issue.

The [full changelog is there](https://dlang.org/changelog/2.096.0.html) with all the details for anyone who wants them. The features below are a couple that I believe warrant a bit of exposition.



### New C-compatible complex types



The complex types `cfloat`, `cdouble`, and `creal` (and their imaginary `i`-prefixed counterparts) have been a part of D [for a very long time](https://digitalmars.com/d/1.0/cppcomplex.html), but for much of that time they have been on the block to face future deprecation.

What became clear over time is that they are accompanied by a high maintenance cost, requiring special cases in the frontend to maintain compatibility with other language features. With that and their specialized nature, they have the wrong cost-benefit ratio. [The `std.complex` module](https://dlang.org/phobos/std_complex.html) introduced the `Complex` type to shift that cost-benefit ratio in the other direction.

For several reasons, the built-in types have yet to be deprecated, but new D code should be written to use `std.complex`. One potential problem there is that the library type is not compatible with the C `_Complex` type, so anyone needing that compatibility has continued to reach for the built-ins.

[This release introduces three new aliased types](https://dlang.org/changelog/2.096.0.html#complex_types) that are automatically configured at compile-time to be ABI compatible with C's `_Complex` and should be used in place of the built-ins when interacting with C or C++: `c_complex_float`, `c_complex_double`, and `c_complex_real`. They are declared in the `stdc.config` module, which must be imported to use them (and [which includes other aliased types](https://github.com/dlang/druntime/blob/master/src/core/stdc/config.d) that have ABI variations between C compilers, such as `c_long` and `c_long_double`).



### Postblit and copy constructor priority



[The postblit constructor](https://dlang.org/spec/struct.html#struct-postblit) has long been the second step in D's approach to copy constructing struct instances:





  1. "blit" (copy) the fields from the source to the destination


  2. invoke the destination's postblit constructor



The first step would always take place, the second only if the struct declaration included an implementation of `this(this)`. The idea behind this is that the first step does a shallow copy, and the postblit constructor can take care of any extra work that is required, such as making "deep copies" (copying referenced data) or incrementing a reference count. The postblit constructor does not have access to the source object.

Unfortunately, time and usage uncovered issues with postblit, particularly in how it behaves in the presence of qualifiers like `const`, `immutable`, and `shared`. Changing the behavior of such a long-lived and deeply ingrained feature is problematic due to the impact on existing code. Instead, the approval of [DIP 1018 introduced copy constructors](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1018.md) to the language.

Now, newly written code should make use of copy constructors rather than postblits, but postblits are not deprecated and remain in the language. Consequently, the two features need to learn to play nice together. 

Until now, when both a postblit and a copy constructor were present, priority was given to the postblit. Unfortunately, there was a corner case that slipped by.

The example in the changelog looks like this:


```d
// library code using postblit
struct A
{
    this(this) {}
}

// new code using copy constructor
struct B
{
    A a;
    this(const scope ref B) {}
}
```
Because `A` implements a postblit, then given an instance `b` of `B`, the postblit of `a` should be invoked anytime a copy of `b` is made. Since the user defined no postblit for `B`, the compiler will generate one that, when invoked, will in turn invoke the postblit of `a`.

Now that postblit constructors have priority over copy constructors, the programmer who implemented B is expecting its copy constructor to run, but it never will.

With 2.096.0, the above code will result in a deprecation message informing the programmer that a generated postblit will be invoked instead of the copy constructor. The programmer then has three options:





  * disable the postblit with `@disable this(this);`


  * implement a postblit, which will then have priority over any copy constructors


  * remove all copy constructors from `B` in preference to the generated postblit





## LDC



Up-to-date beta versions of the LLVM-based D compiler, LDC, tend to come not too long after DMD releases. As I write, the LDC maintainer Martin Kinkelin [is working toward the next beta release](https://github.com/ldc-developers/ldc/pull/3678), and that will support D 2.096.0. Keep an eye on [the D Announce forum](https://forum.dlang.org/group/announce) and [the LDC release page](https://github.com/ldc-developers/ldc/releases) for news of the release.



## GDC



Using the current release of GDC, the D compiler which is distributed as part of [the GNU Compiler Collection (GCC)](https://gcc.gnu.org/), the `__VERSION__` constant reports `2.076`. LDC and DMD moved on from that version with a D frontend that was ported from C++ to D, but to facilitate bootstrapping, GDC had to stick with the C++ version of the runtime for inclusion into GCC. Maintainer Iain Buclaw has been backporting some fixes and improvements from upstream, so the 2.076 of GDC is no longer at full bug/feature parity with DMD 2.076. Just one example of many, [a performance boost to `static foreach`](https://github.com/dlang/dmd/pull/11303) that was merged into DMD last year is also included in the GDC that shipped with GCC 10.2.1.

The upside of this is that GDC is rock-solid stable. The downside is that code that compiles on the latest DMD and LDC may fail to compile on GDC. But in between backporting bugfixes, his day job, and his normal life, Iain has been expending his free time on replacing the old C++ frontend with the newer D implementation. Given time constraints and the amount of work left to do, version 11 of GDC will remain on D 2.076. Once the GCC feature freeze is lifted in May, he will start laying the foundations for making the switch to the newer frontend in a future GCC release.



## Symmetry Autumn of Code 2020



[The latest edition of SAOC](https://dlang.org/blog/symmetry-autumn-of-code/) kicked off in September of 2020. With the sponsorship of Symmetry Investments, four students were to work through four milestones to complete projects that would benefit the D ecosystem. Upon successful completion of each of the first three milestones, each student would receive $1,000. At the end of the fourth milestone, the SAOC judges would award one of them with a final $1,000 payment and a free trip to the next real-world DConf.

All four students completed the first two milestones, but two of them were unable to continue past the third. Milestone 4 ended on January 15th. At the end of the month, the two remaining students submitted their final milestone reports and a summary of their experience working on their projects: what they learned, how reality compared to their expectations, and what their plans are going forward. With those documents and the mentors' student evaluations in hand, the SAOC judges had to decide which of the two should be awarded the final payment.

Their decision was neither obvious nor easy. Both of the students did outstanding work throughout the event. Their milestone reports sufficiently documented their progress. They kept up with their forum updates as required. Their mentors gave them glowing reviews. There was very little to separate them. In the end, the judges looked at the state of both projects and determined that one would have a broader and more immediate benefit to the D community than the other, and so made a decision.

But they didn't leave it there. The SAOC judges felt that both students had done so well that they both should be rewarded, but our sponsor had only allocated funding for one reward. The judges devised alternative scenarios and asked our sponsor which of the options they would support. They received an answer, and I informed the students in the last week of February (after they had patiently waited several days beyond our initial deadline).

Robert Aron was selected to receive the final payment and a free trip to the next real-world DConf. His project: implementing D clients for Google APIs. By the end, he had two fully functional libraries for talking to Google Drive and Google Calendar, and an in-progress library for interfacing with Gmail, all of which [you can find on GitHub](https://github.com/Robert-Aron293/google-api-dlang-client). He also has a work-in-progress [template-based Google API generator](https://github.com/Robert-Aron293/google-apis-template-based-generator/blob/main/template.mustache). He intends to finish both it and the GMail client. He also plans to eventually generate libraries for [Google APIs that use RPC](https://github.com/googleapis/googleapis).

The "runner-up" is Adela Vais. With Symmetry's permission, she was offered a choice between a final $1,000 payment and a free trip to the next real-world DConf (I'll leave it to her to tell folks which she chose). The initial goal of her SAOC project was to implement a D [GLR parser for GNU Bison](https://www.gnu.org/software/bison/manual/html_node/GLR-Parsers.html). After seeing the state of the existing D [LALR(1) pull parser](https://en.wikipedia.org/wiki/LALR_parser) support in Bison, she shifted her goal to first implement its missing features and add support for [a push parser](https://www.gnu.org/software/bison/manual/bison.html#Push-Decl) before moving on to the GLR parser. By the end of the event, the push parser was 95% complete, and she intended to take it all the way and then turn to the GLR parser support.

Congratulations to both Robert and Adela! And a special thanks to Laeeth Isharc and Symmetry Investments for providing this opportunity to young programmers every year. Robert and Adela told us they learned more from this experience than they had expected to, and they were obviously proud of the work they had done.



## D ecosystem projects and tasks



[The DLang Project Idea Repository](https://github.com/dlang/projects) was created during [DConf 2019 in London](https://www.youtube.com/playlist?list=PLIldXzSkPUXWORGtUrnTo2ylziTHR8_Sq) after a discussion among some of the attendees. Not only is it a solid source of project ideas for potential participants in Google Summer of Code and SAOC applicants, but it's also useful for anyone looking to make a meaningful contribution to the D community.

The [D Ecosystem Task List](https://github.com/dlang/ecotasks) came about after discussions with representatives of companies using D in production. Any company that is willing to allow one of their employees at least one day per quarter to work on smaller tasks that benefit the D ecosystem can use the repository to get ideas and indicate which task they're working on. Funkwerk, who [have been using D in production since 2008](https://dlang.org/blog/d-in-production/), have been the only ones to take us up on this so far. Their employees have contributed improvements to [D-Scanner](https://github.com/dlang-community/D-Scanner), [dfmt](https://github.com/dlang-community/dfmt), [and dub](https://github.com/dlang/dub). We are grateful to them for all they've done.

The task list isn't limited to company employees. It's a list of broad categories, not specific tasks, for anyone who has an hour or two they'd like to spend on writing D code. So if you are looking for D experience or would simply like to contribute, and you don't have time for a full-on project, the Ecosystem Task List is a great place to go for ideas. And if you have ideas for other broad task categories that can be added to the list, please submit a pull request!
