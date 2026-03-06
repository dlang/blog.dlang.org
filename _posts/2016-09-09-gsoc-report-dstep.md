---
author: WojSzes
comments: false
date: 2016-09-09 14:04:20+00:00
layout: post
link: https://dlang.org/blog/2016/09/09/gsoc-report-dstep/
slug: gsoc-report-dstep
title: 'GSoC Report: DStep'
wordpress_id: 211
categories:
- Compilers &amp; Tools
- GSoC
- Guest Posts
permalink: /gsoc-report-dstep/
redirect_from: /2016/09/09/gsoc-report-dstep/
---

_Wojciech Szęszoł is a Computer Science major at the University of Wrocław. As part of Google Summer of Code 2016, he chose to make improvements to Jacob Carlborg's [DStep](https://github.com/jacob-carlborg/dstep), a tool to generate D bindings from C and Objective-C header files._



* * *



![GSoC-icon-192](http://dlang.org/blog/wp-content/uploads/2016/09/GSoC-icon-192.png)It was December of last year and I was writing an image processing project for a course at [my university](https://international.uni.wroc.pl/en/s3.php). I would normally use [Python](https://www.python.org/), but the project required some custom processing, so I wasn't able to use [numpy](http://www.numpy.org/). And writing the inner loops of image processing algorithms in plain Python isn't the best idea. So I decided to use D.

I've been conscious about the existence of the D language for as long as I can remember, but I'd never convinced myself before to try it out. The first thing I needed to do was to load an image. At the time, I didn't know that there is [a DUB repository](https://code.dlang.org/) containing bindings to image loading libraries, so I started writing bindings to [libjpeg](http://libjpeg.sourceforge.net/) by myself. It didn't end very well, so I thought there should be a tool that will do the job for me. That's when I found [DStep](https://github.com/jacob-carlborg/dstep) and [htod](https://dlang.org/htod.html).

Unluckily, the capabilities of DStep weren't satisfying (mostly the lack of any kind of support for the preprocessor) and htod didn't run on Linux. I ended up coding my project in C++, but as GSoC ([Google Summer of Code](https://summerofcode.withgoogle.com/)) was lurking on the horizon, I decided that I should give it a try with DStep. I began by contacting Craig Dillabaugh (_Ed. Note: Craig volunteers to organize GSoC projects using D_) to learn if there was any need for developing such a project. It sparked some discussion on [the forum](http://forum.dlang.org/), the idea was accepted, and, more importantly, Russel Winder agreed to be the mentor of the project. After some time I needed to prepare and submit an official proposal. There was an interview and fortunately I was accepted.

The first commit I made for DStep is dated to February, 1. It was a proof of concept that C preprocessor definitions can be translated to D using [libclang](http://clang.llvm.org/doxygen/group__CINDEX.html). Then I improved the testing framework by replacing the old [Cucumber-based](https://cucumber.io/) tests with some written in D. I made a few more improvements before the actual GSoC coding period began.

During GSoC, I added support for translation of preprocessor macros (constants and functions). It required implementing a parser for a small part of the C language as the information from libclang was insufficient. I implemented translation of comments, improved formatting of the output code (e.g. made DStep keep the spacing from C sources), fixed most of the issues from [the GitHub issue list](https://github.com/jacob-carlborg/dstep/issues) and ported DStep to Windows. While I was coding I was getting support from Jacob Carlborg. He did a great job by reviewing all of the commits I made. When I didn't know how to accomplish something with D, I could always count on help on [forum.dlang.org](http://forum.dlang.org/).

DStep was the first project of such a size that I coded in D. I enjoyed its modern features, notably [the module system](http://dlang.org/spec/module.html), [garbage collector](http://dlang.org/spec/garbage.html), [built-in arrays](http://dlang.org/spec/arrays.html), and [powerful templates](http://dlang.org/spec/template.html). I used [unittest blocks](http://dlang.org/spec/unittest.html) a lot. It would be nice to have named unit tests, so that they can be run selectively. From the perspective of a newcomer, the lack of consistency and symmetry in some features is troubling, at least before getting used to it. For example there is [a built-in hash map](http://dlang.org/spec/hash-map.html) and no hash set, some identifiers that should be keywords starts with `@` (_Ed. Note: see the [Attributes documentation](http://dlang.org/spec/attribute.html)_), etc. I was very sad when I read that the `in` keyword is not yet fully implemented. Despite those little issues, the language served me very well overall. I suppose I will use D for my personal toy projects in the future. And for DStep of course. I have some unfinished business with it :).

I would like to encourage all students to take part in future editions of GSoC. And I must say the [D Language Foundation](https://dlang.org/foundation.html) is a very good place to do this. I learned a lot during this past summer and it was a very exciting experience.
