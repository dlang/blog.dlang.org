---
author: DBlogAdmin
comments: false
date: 2016-07-07 13:22:57+00:00
layout: post
link: https://dlang.org/blog/2016/07/07/project-highlight-auburn-sounds/
slug: project-highlight-auburn-sounds
title: 'Project Highlight: Auburn Sounds'
wordpress_id: 76
categories:
- Project Highlights
permalink: /project-highlight-auburn-sounds/
redirect_from: /2016/07/07/project-highlight-auburn-sounds/
---

One of the questions some people ask when evaluating a language for the first time is, _who's making money with it_? That's no exception with D. While there are [a number of companies](https://dlang.org/orgs-using-d.html) using D in production, there are also people making money with the language out of their homes. [The last Project Highlight](http://dlang.org/blog/2016/06/24/project-highlight-the-powernex-kernel/) looked at a freely available open source project. This one is about a proprietary set of audio plugins developed and sold by one programmer.

Guillaume Piolat has been using D since 2007. In that time, he has been an active member of the community, maintaining and contributing to several open source projects including [Derelict ](https://github.com/DerelictOrg)and [GFM](http://d-gamedev-team.github.io/gfm/). Now, he has begun to build an audio plugin business that he calls [Auburn Sounds](https://www.auburnsounds.com/).


I have a long love/hate relationship with D and honestly I prefer the programs I've made with it over other languages. It's the language I feel the most _free_ with and it has always served me well. I'm just not comfortable with the low productivity level of C++, so I was willing to take a long term bet.


That bet is now paying off for him, but it hasn't been completely free of challenges.


I did not know how hard wrapping plugin formats would be, especially on OS X. It turned out you could "derelictify" Carbon and Cocoa and call the Obj-C runtime without much difficulty, just with some work.


By _derelictify_, he means creating [dynamic bindings](http://derelictorg.github.io/dynstat.html) in D to the C APIs of shared libraries that can then be loaded at run time via system APIs like `dlopen`. The [DerelictUtil ](https://github.com/DerelictOrg/DerelictUtil)library can be used to create loaders for such bindings, hiding the platform-specific details behind a simple API, and [the Derelict project](https://github.com/DerelictOrg) provides a number of bindings that do just that. Hence the term _derelictify_. It's worth noting here that D has some limited support for [interfacing directly with Objective-C](https://dlang.org/spec/objc_interface.html).

Other issues included bugs in the tool chain and less-than-ideal code generation.


I stumbled upon some DMD backend bugs and some bugs in DUB, but all of them were fixed in the end. Additionally, DMD codegen wasn't competitive. Fortunately, LDC has made some incredible progress, bringing top codegen to both Windows and OS X . It's something that I was expecting, but not so soon! This particular bet really paid off.


It's commonly recommended in the D community to use DMD for its blazing fast compile times during development and, for the projects that really need it, [LDC ](http://wiki.dlang.org/LDC)or [GDC ](http://gdcproject.org/)for production to take advantage of the fact that they typically produce binaries with better performance.

[DUB ](http://code.dlang.org/download)is a build tool and package manager for D projects. A number of libraries have been registered in [the DUB Registry](http://code.dlang.org/), all of which are available to use as dependencies in any DUB-managed project. The tool will soon be shipping with DMD in an upcoming release of the compiler.

Now that he has several completed plugins available, how does Guillaume feel about having chosen D?


Auburn Sounds has existed for fifteen months, and in the past nine I've not thought of going back to C++ a single time. I don't use D-specific features aggressively. At first, I was thinking that I would need D's meta-programming support, like [Design by Introspection](http://dconf.org/2015/talks/alexandrescu.html), to create an efficient audio library, but it turned out having almost no abstraction worked well enough. The thing that matters most for this project is codegen quality, speed of development, platform support and low mental overhead.

Other benefits include the fluidity of [DUB](http://code.dlang.org/download), the availability of [VisualD](http://rainers.github.io/visuald/visuald/StartPage.html), and the quality of some third-party libraries like [imageformats](https://github.com/lgvz/imageformats), [DerelictUtil](https://github.com/DerelictOrg/DerelictUtil), and especially [ae.utils.graphics](https://github.com/p0nce/ae-graphics). Speed of compilation and development count a lot, but they aren't something you really notice once you've grown accustomed to them.


Guillaume intends to continue to use D to develop more audio plugins and improve the ones he has already made available. His latest, [Panagement](https://www.auburnsounds.com/products/Panagement.html), has both free and paid versions.


Panagement solves two problems in audio mixing: giving stereo content to a track quickly and fixing regular panning, which doesn't sound that great on headphones. It's a top dog in a sub-niche that traditionally doesn't interest people a lot. It's also the first plugin I've released entirely built with LDC.


In addition to Panagement, you can also currently find a [voice octaver](https://www.auburnsounds.com/products/Graillon.html) for sale and three other plugins freely available in their full versions: a [binaural panner](https://www.auburnsounds.com/products/Psypan.html), a [physical synthesizer](https://www.auburnsounds.com/products/Koch.html), and a [distortion plugin](https://www.auburnsounds.com/products/Distort.html).

The D community undoubtedly collectively wishes Guillaume, long one of their own, the best of luck. If you are a one man shop or a small team using D to produce commercial software, let us know [in the D Forums](http://forum.dlang.org/)!
