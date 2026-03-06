---
author: KaiNacke
comments: false
date: 2016-06-20 14:21:45+00:00
layout: post
link: https://dlang.org/blog/2016/06/20/making-of-ldc-1-0/
slug: making-of-ldc-1-0
title: 'Making Of: LDC 1.0'
wordpress_id: 45
categories:
- Compilers &amp; Tools
- Guest Posts
permalink: /making-of-ldc-1-0/
redirect_from: /2016/06/20/making-of-ldc-1-0/
---

_This is a guest post from Kai Nacke. A long-time contributor to the D community, Kai is the author of _[D Web Development](http://amzn.to/1qdrvrH)_ and the maintainer of _[LDC, the LLVM D Compiler](https://github.com/ldc-developers/ldc/releases/tag/v1.0.0)_._



* * *



LDC has been under development for more than 10 years. From release to release, the software has gotten better and better, but the version number has always implied that LDC was still the new kid on block. Who would use a version 0.xx compiler for production code?

These were my thoughts when I raised the question, "Version number: Are we ready for 1.0?" [in the forum](http://forum.dlang.org/post/ohuianuptmikfzzeyuqq@forum.dlang.org) about a year ago. At that time, the current LDC compiler was 0.15.1. In several discussions, the idea was born that the first version of LDC based on the frontend written in D should be version 1.0, because this would really be a major milestone. Version 0.18.0 should become 1.0!

Was LDC really as mature as I thought? Looking back, this was an optimistic view. At DConf 2015, Liran Zvibel from [Weka.IO](http://www.weka.io/) mentioned in [his talk about large scale primary storage systems](http://dconf.org/2015/talks/zvibel.html) that he couldn't use LDC because of bugs! Additionally, the beta version of 0.15.2 had some serious issues and was finally [abandoned in favor of 0.16.0](http://forum.dlang.org/thread/xbncnnefagpvvqcxmham@forum.dlang.org). And did I mention that I was busy [writing a book ](http://amzn.to/1qdrvrH)about [vibe.d](http://vibed.org/)?

Fortunately, over the past two years, more and more people began contributing to LDC. The number of active committers grew. Suddenly, the progress of LDC was very impressive: Johan added DMD-style code coverage and worked on merging the new frontend. Dan worked on an iOS version and Joakim on an Android version. Together, they made ARM a first class target of LDC. Martin and Rainer worked on the Windows version. David went ahead and fixed a lot of the errors which had occurred with the Weka code base. I spent some time working on the ports to PowerPC and AArch64. Uncounted small fixes from other contributors improved the overall quality.

Now it was obvious that a 1.x version was overdue. Shortly after DMD made the transition to the D-based frontend, LDC was able to use it. After the usual alpha and beta versions, I built the final release version on Sunday, June 5, and [officially announced it the next day](http://forum.dlang.org/post/aiedvlztjxxahoxgdssm@forum.dlang.org). Version 1.0 is shipping now!

Creating a release is always a major effort. I would like to say "Thank you!" to everybody who made this special release happen. And a big thanks to our users; your feedback is always a motivation to make the next LDC release even better.

Onward to 1.1!
