---
author: DBlogAdmin
comments: false
date: 2019-08-25 13:22:04+00:00
layout: post
link: https://dlang.org/blog/2019/08/25/saoc-2019-projects-and-participants/
slug: saoc-2019-projects-and-participants
title: SAOC 2019 Projects and Participants
wordpress_id: 2177
categories:
- Community
- D Foundation
- SAoC
permalink: /saoc-2019-projects-and-participants/
redirect_from: /2019/08/25/saoc-2019-projects-and-participants/
---

![Symmetry Investments logo](http://dlang.org/blog/wp-content/uploads/2018/09/logo-1.png)Last Sunday, August 18, was the deadline for [Symmetry Autumn of Code 2019](https://dlang.org/blog/symmetry-autumn-of-code/) applications. We received a total of eight applications, which is the same number we saw last year. This time around we were able to accept more than three: five of the applicants will be participating.

The applications were reviewed by the five members of the SAOC 2019 Committee. Each member independently ranked the applications in order of preference. Points were assigned based on the rankings and the top five applications were accepted.

Before we get into the details of the projects, on behalf of the D Language Foundation and the SAOC team, I'd like to publicly thank all eight applicants for taking the time to submit an application. I'd also like to thank Laeeth Isharc and Symmetry Investments for sponsoring the event again this year, and our five SAOC Committee members for volunteering their time throughout the event:  John Colvin, Mathias Lang, Átila Neves,  Robert Schadek, and Ethan Watson. They will be monitoring the progress of each project through the milestone reports and ultimately will select one participant to receive an extra $1000 payment and an all-expense paid trip to DConf 2020.


## The Projects





 	
  * **Multi-Level Intermediate Representation Support for LDC **- According to [the MLIR project README](https://github.com/tensorflow/mlir), it's "a common intermediate representation" intended to "unify the infrastructure required to execute high performance machine learning models in TensorFlow and similar ML frameworks". Roberto Rosmaninho's primary project goal is to provide the LDC D compiler with "a new level of abstraction to support the integration of MLIR into the D ecosystem". Roberto is working on a Computer Science major and is an undergraduate research assistant at Federal University of Minas Gerais in Brazil. His mentor for the project is Nicholas Wilson.

 	
  * **Implement DIP 1014 and expand support for C++ STL containers** - Suleyman Sahmi's main goal is to "advance the existing work on [the] D interface to C++ STL containers". There is [a project at GitHub geared toward that end](https://github.com/dlang-cpp-interop/stl-containers/) which Manu Evans and Laeeth Isharc have been working on and which is blocked on the lack of [an implementation for DIP 1014](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1014.md), along with a few issues with the D ABI and name mangling. He first intends to implement DIP 1014, then he'll resolve several of the related DMD bugs and will use the remaining time to expand support for the C++ STL. Suleyman is a self-taught programmer from Morocco and already has become a contributor to DMD.

 	
  * **DPP with Linux kernel headers** - The DPP tool, which allows D modules to directly `#include` C and C++ headers, currently is unable to work with the Linux kernel headers. Cristian Becerescu aims to fix that. If he is able to do so with time remaining, he will work on further improvements and refinements to DPP, including ironing out issues it might have with other C library headers the community brings to his attention. Cristian is a 4th-year Computer Science and Engineering student at University Politehnica of Bucharest. He is fortunate to have two mentors for this project in the form of [Edi Staniloiu and Razvan Nitu](https://dlang.org/blog/2019/08/08/d-summer-school-postmortem/).

 	
  * **Create a CI or other infrastructure for measuring D's progress and performance** - Max Haughton, an 18-year-old British physics student, will be taking on the task of "creating a mechanism by which we can measure various properties of the D ecosystem in a deterministic manner". This includes properties such as compilation time, compile-time memory usage, and profiling the compiler to determine "why performance is what it is". He also intends to extend it to run-time performance by "forming a set of benchmarks by which we can profile Phobos and druntime both against their versions...and the version of the compiler".

 	
  * **Solve Dependency Hell: link with more than one version of the same project** - When a project has dependencies that in turn rely on different versions of the same library, steps must be taken to reconcile the version difference in order to successfully compile. If it's even possible, [it's cumbersome and introduces new difficulties](https://github.com/dlang/projects/issues/51). Tiberiu Lepadatu aims to solve this problem of Dependency Hell by making it possible to compile a project with multiple versions of the same library. This is considered as a crucial first step in making Phobos available via the DUB registry. Tiberiu is no stranger to D and has contributed to the core projects in the past. He will likely be working with Sebastian Wilzbach as his mentor.




## Getting Under Way


The SAOC participants will spend the next three weeks preparing to get their projects started. They'll be compiling their Milestones, doing research, and those without a mentor will be searching for one. Things officially kick off on September 15, with Milestone deadlines falling on the 15th of each month through January.

This year, we'll be expecting the participants to make weekly updates in the forums. We will also encourage them to spend time on IRC, Slack, or Discourse to get to know the community, discuss their projects, and find inspiration in solving the challenges they'll face. We encourage all members of the D community to show their support and help keep up motivation.

Each of these projects will improve the D ecosystem.We're fortunate to have this opportunity, along with out participation in the currently ongoing Google Summer of Code, to get so much done without the need to raise more money or dig into our Human Resource Fund. We should all be willing to do what we can to help these projects succeed.

On a personal level, I'm looking forward to working with these five programmers in the coming months and to seeing all of them make it through to the end of a successful Symmetry Autumn of Code!


