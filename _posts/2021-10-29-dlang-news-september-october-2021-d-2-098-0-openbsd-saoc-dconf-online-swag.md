---
author: DBlogAdmin
comments: false
date: 2021-10-29 14:34:14+00:00
layout: post
link: https://dlang.org/blog/2021/10/29/dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag/
slug: dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag
title: 'DLang News September/October 2021: D 2.098.0, OpenBSD, SAOC, DConf Online
  Swag'
wordpress_id: 2997
categories:
- Community
- Compilers &amp; Tools
- DConf
- DMD Releases
- LDC Releases
- News
- SAoC
permalink: /dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag/
redirect_from: /2021/10/29/dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag/
---

![Digital Mars D logo]({{ '/assets/images/dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag/d6.png' | relative_url }})

Version 2.098.0 of the D programming language is now available in the form of DMD 2.098.0 (the reference D compiler) and LDC 1.28.0 (the LLVM-based D compiler), D has come to OpenBSD, cool things are happening thanks to the Symmetry Autumn of Code, and DConf Online 2021 t-shirts are available for purchase.

Read on for the deets.



## DMD 2.098.0



This release comes with [17 major changes and 160 fixed Bugzilla issues from 62 contributors](https://dlang.org/changelog/2.098.0.html) across the core repositories. The number of fixed issues may well be a record high. [The 2.097.0 release](https://dlang.org/changelog/2.097.0.html) had 144, and [the 2.094.0 release](https://dlang.org/changelog/2.094.0.html) had 119, but a cursory look at several other major releases shows numbers ranging from the high 40s to under 100, with counts in the 50s showing up frequently. This is the sort of trend we were hoping to see [when Razvan Nitu came on board](https://dlang.org/blog/2021/05/18/a-pull-request-managers-perspective/) as our Pull Request and Issue Manager, and we couldn’t be more pleased.

There are two items of note that I’d like to point out from the new release, and then I have a little more to say about the work Razvan is doing.



### ImportC



[The ImportC compiler](https://dlang.org/spec/importc.html) is a major enhancement to D that allows the D compiler to directly compile C source code. Walter has been working on it for a few months now, and this is the first release in which it’s available. ImportC enables the compiler to inline C function calls and even [evaluate them at compile time via CTFE](https://tour.dlang.org/tour/en/gems/compile-time-function-evaluation-ctfe). ImportC targets C11 and does not currently handle preprocessor directives, so any C source you do intend to compile must first be run through a preprocessor. It’s not yet complete, but if you have a use case for it, any help in finding and reporting ImportC bugs is welcome. Contributions to fix said bugs doubly so!



### Fork-based garbage collector



This release also [includes an optional concurrent garbage collector for Posix systems](https://dlang.org/changelog/2.098.0.html#forkgc). This is cool in and of itself, but more so because the project came to fruition thanks to the Symmetry Autumn of Code. It was originally developed for D1 by Leandro Lucarella but was never included in an official release (using alternative GCs back then required more than just a simple command-line switch). In 2018, for the inaugural edition of SAOC, Francesco Mecca undertook to port the GC to D2. This resulted in a pull request to DRuntime that was ultimately merged in time for this release by Rainer Schuetze.

To use the new GC, provide the DRuntime option `--DRT-gcopt=fork:1` on the command-line of any program compiled against DRuntime 2.098.0+ (this **is not a compiler option**, but an option to any program linked with DRuntime). It can also be configured programmatically via:


    <code><span id="cb1-1"><a tabindex="-1" href="#cb1-1" aria-hidden="true"></a><span class="in">extern</span>(C) <span class="fu">__gshared</span> <span class="bu">string</span><span class="op">[]</span> rt_options <span class="op">=</span> <span class="op">[</span> <span class="st">"gcopt=fork:1"</span> <span class="op">];</span></span></code>
See the D documentation [for more GC configuration options](https://dlang.org/spec/garbage.html#gc_config).



### Shrinking the pull-request queues



Razvan has been managing pull requests across several of our repositories, but he’s been laser-focused on reducing the number of PRs in the phobos and druntime repositories, with dmd his next target. This isn’t just about lowering the PR count. He’s been reviving old PRs with the original author where he can (he tells me he was surprised how many PR authors were responsive, even after no activity on a PR for a few years) and has tried to rebase and resolve those where he can’t. Here are some statistics he’s gathered on PR activity so far this year across the phobos, druntime, and dmd repositories:





  * **phobos**: 568 PRs created, 650 PRs closed


  * **druntime**: 283 PRs created, 311 PRs closed


  * **dmd**: 1140 PRs created, 1126 closed



At the time he sent me the stats on October 29th, the number of open PRs in phobos had gone down from 160 to 77 and druntime from 130 to 96. The number of open PRs in dmd has remained fairly constant at around 230.

We want to thank Razvan for all the work he is doing, [Symmetry Investments](https://symmetryinvestments.com/) for sponsoring his position, the volunteer members of the “strike teams” Razvan has assembled to squash as many bugs as possible, and every contributor who has donated and continues to donate their time and effort to improving our favorite programming language.



## LDC 1.28.0



[The latest release of LDC](https://github.com/ldc-developers/ldc/releases/tag/v1.28.0) implements D 2.098.0 (D frontend, DRuntime, and Phobos) and is compatible with LLVM 6.0 - 12.0.

A major item in this release is that LDC now supports dynamic casts across binary boundaries. DLL support has long been a weak point in D, often requiring the programmer to resort to `extern(C)` functions that return handles (pointers, references) to D objects. Martin Kinkelin has worked to improve the situation in LDC, motivated primarily by the desire to provide the standard library and runtime as a DLL on Windows.

Thanks to Martin and all the LDC contributors for the work they do to keep LDC releases in sync with those of DMD. If you benefit from their efforts, [please consider sponsoring Martin](https://github.com/sponsors/kinke) (and LDC by extension) on GitHub!



## D on OpenBSD



The D ecosystem grows primarily because of the efforts of volunteers who step forward to fill in the blanks. New D projects pop up all the time, but it’s pretty rare to hear that someone has brought D to a new platform. Brian Callahan has done just that.

Brian has been on a mission to bring D to OpenBSD. In August of this year, he popped into the D forums [with an announcement](https://forum.dlang.org/post/fhbyggtwoiiewwnnllvo@forum.dlang.org) that GDC, the GCC-based D compiler maintained by Iain Buclaw, was now available in the OpenBSD ports tree as part of GCC 11. In early October, he let us know [that DMD was coming to the platform](https://forum.dlang.org/thread/gtialrlzimgzfyztjtiv@forum.dlang.org). Then in late October, [he had the same news about LDC](https://forum.dlang.org/post/fjukhxxcjslptauzcvoc@forum.dlang.org). Instructions for installing DMD on OpenBSD [are on the download page](https://dlang.org/download.html) (and can be extrapolated to LDC and GDC).

We are grateful to Brian for the work he has done to make this happen. We’re looking forward to his upcoming DConf Online 2021 talk, [Life Outside the Big 4: The Adventure of D on OpenBSD](https://dconf.org/2021/online/index.html#brian):




The journey of D from pie-in-the-sky to a package officially offered in the OpenBSD package repository serves as a model story for other platforms who want to offer D to their userbase. We will walk through the many interconnected parts required to get a D package on OpenBSD, what the future is like for D outside the Big 4, how you can get started with D on your platform, and how those of us who enjoy life outside the Big 4 can be a positive force for D and the D community.






## SAOC News



The SAOC 2021 progress bar is past the 25% mark. The first milestone wrapped up on October 15, and the participants have been posting weekly progress reports [in the General Forum](https://forum.dlang.org/group/general). It’s always interesting to read about the challenges they encounter and their solutions. But the latest SAOC isn’t the only edition about which there is news to report.

I’ve written above about the SAOC 2018 forking GC project that has found its way into the latest release of DRuntime. I can’t begin to tell you how pleased I am that another SAOC project has come into its own.

For SAOC 2020, Adela Vais set out to [implement a D backend](https://github.com/adelavais/bison) for [the venerable Bison parser generator](http://www.gnu.org/software/bison/). Not only did Adela successfully complete SAOC, she saw her project through to its ultimate goal. The D backend was officially released as part of [Bison 3.81 in September](https://savannah.gnu.org/forum/forum.php?forum_id=10047).

We want to offer Adela our congratulations and a huge round of applause for a job well done! Getting a project of this scope accepted into a GNU codebase is no mean feat.



## DConf Online 2021 T-Shirts



[DConf Online 2021](https://dconf.org/2021/online/index.html) is less than a month away. The D Language Foundation will be providing DConf Online 2021 swag to the DConf speakers and prizes to viewers asking questions in the post-talk live stream Q & A sessions. The cost of the items and their shipping are the only DConf Online expenses, and they’re covered by the D Language Foundation General Fund.

Direct [donations to the General Fund and our more targeted funds](https://dlang.org/foundation/donate.html) are always appreciated, but you can also help support the D programming language and DConf Online by purchasing a DConf Online 2021 T-Shirt or other D swag in [the DLang Swag Emporium](https://www.zazzle.com/store/dlang_swag?rf=238129799288374326). All proceeds go straight into the General Fund. You get some swag along with our gratitude, and we get a couple of bucks. That’s a pretty good deal!



## Looking Forward



As we near the end of 2021, we are looking forward to 2022 and beyond. The D programming language, its ecosystem, and its community have come a long way from the gaggle of curious coders who first took an interest in a one-man project [by the guy who had created the game Empire and the Zortech C++ compiler](http://www.classicempire.com/).

The primary means of contributing to the core D projects went from emailing patches to Walter, to posting patches on Bugzilla, to committing to a Subversion repository, to submitting pull requests on GitHub. The web site went from being [a few basic HTML pages of the D spec on digitalmars.com](http://web.archive.org/web/20011212082225/http://www.digitalmars.com/d/index.html) maintained only by Walter, to [a simple HTML site](http://web.archive.org/web/20111014012905/http://dlang.org/) designed by a community member under the dlang.org domain, to [the more complex collection of pages and scripts](https://dlang.org/) that today is [maintained in Ddoc by multiple contributors](https://github.com/dlang/dlang.org). The ecosystem has gone from random libraries and tools hosted by individuals on myriad services, to [centralized hosting at dsource.org](http://www.dsource.org/), to [the package repository at code.dlang.org](https://code.dlang.org/).

These are just some examples of major changes over the years, each in response to growth: as the community grew in size, some of the processes and systems began to burst at the seams. To continue to grow, something had to change. Such improvements have nearly always been the result of community action: discussion and debate in the forums eventually would lead to a champion stepping forward to make it happen. Community action has been the driving force of D since [Walter first announced the “D alpha compiler” in late 2001](https://digitalmars.com/d/archives/2226.html#N2226%22). That’s still true today. We have a handful of paid positions, but we are still primarily driven by volunteers.

The see-a-problem-and-fix-it philosophy that carried D to where we are today has served us well, and we hope it will continue to do so into the future. But that alone is no longer enough. We are bursting at the seams again, and have been for some time. In the monthly foundation meetings, we’ve been discussing specific issues, both low level and high, and how to solve them. But there’s one thing that’s been missing from the equation: _organization_.

Razvan Nitu’s position as Pull Request & Issue Manager grew out of an email discussion, prompted by Laeeth Isharc, and was a year in the making. We are grateful for every volunteer who has and continues to make themselves available to review pull requests. Razvan is here not to replace them, but to complement them. They can continue as they have done. What Razvan brings to the mix is organization. He’s there to make sure fewer issues and PRs fall through the cracks, to ensure that as many issues as possible that can be resolved _are_ resolved.

In November, the D Language Foundation and a couple of contributors are meeting with a community member who has graciously volunteered his time and expertise to advise us on how to bring the disparate servers in the D community under Foundation management and multiple admins. The end goals are to eliminate the financial burden on the volunteers who maintain these services and, hopefully, reduce the response time when it comes to solving server-related issues or making changes. In other words, organization.

I’m in the middle of revising the Vision Document that we put together over the summer. I’m not just editing it, though. I’m expanding it. My vision of the vision document has evolved since [we first discussed a “goal-oriented task list” in our June meeting](https://forum.dlang.org/thread/iqtdfbbbidqoeqyxgqax@forum.dlang.org). I said at the time that I didn’t “know what the initial version of the final list will look like”. I feel that what we came up with falls short of meeting the need it was intended to fill. Now, I’m pretty sure of what it needs to look like. At the moment, I’m swamped in preparations for DConf Online 2021, so I’ve put the document on the backburner. I plan to pick it up again in early December and present my revisions at the last foundation meeting of the year for approval. If all goes well, it should be published on dlang.org in January. This will be a living document, updated to reflect current priorities as time goes by.

Mathias Lang is working on a proposal to bring organization into even more of our processes. It’s a modified version of the governance proposal [he brought to the September foundation meeting](https://forum.dlang.org/thread/fnzuguuewsvzswpwiwar@forum.dlang.org), the aim of which is to formalize a core team to oversee the day-to-day guidance and management of the D ecosystem. I hope that this will take what already happens in our monthly meetings to the next level. I see this as a means to establish a framework for creating workgroups that can oversee specific tasks and projects, bringing more opportunities for follow-up and follow-through. It should also help provide guidance and establish priorities (e.g., via revisions to the vision document) so that independent contributors can direct their efforts not just to the issues they care about, but those that are seen as a priority by the core team. (I want to emphasize that this is my personal view. Mathias has yet to complete the proposal. But my view is informed by what we discussed in the September meeting.)

With these and future steps aimed at better organizing our community, we intend to level up our ecosystem: motivate library development, improve the onboarding experience, increase retention, make it easier to contribute, and generally resolve the long-standing issues that tarnish the experience of using the best programming language we know. We ask our current volunteers to keep volunteering, and those who aren’t yet doing so to keep an eye out for the right opportunity to pitch in. Together, we can get to where we all want to go.
