---
author: LodovicoGiaretta
comments: false
date: 2016-10-14 12:09:25+00:00
layout: post
link: https://dlang.org/blog/2016/10/14/gsoc-report-std-experimental-xml/
slug: gsoc-report-std-experimental-xml
title: 'GSoC Report: std.experimental.xml'
wordpress_id: 448
categories:
- GSoC
- Guest Posts
- Phobos
permalink: /gsoc-report-std-experimental-xml/
redirect_from: /2016/10/14/gsoc-report-std-experimental-xml/
---

_Lodovico Giaretta is currently pursuing a Bachelor Degree in Computer Science at the University of Trento, Italy. He participated in Google Summer of Code 2016, working on a new XML module for D's standard library, Phobos._



* * *



![GSoC-icon-192](http://dlang.org/blog/wp-content/uploads/2016/09/GSoC-icon-192.png)I started coding in high school with Pascal. I immediately fell in love with programming, so I started studying it by myself and learned both Java and C++. But when I was using Java, I was missing the powerful metaprogramming facilities and the low level features of C++. When I was using C++, I was missing the simplicity and usability of Java. So I started looking for a language that "filled the gap" between these two worlds. After looking into many languages, I finally found D. Despite being more geared towards C++, D provides a very high level of productivity, as correct code is easier to read and write. As an example, I was programming in D for several months before I was bitten by a segfault for the first time. It easily became one of my favorite languages.

The apparent lack of libraries, my lack of time, and the need to use other languages for university projects made me forget D for some time, at least until someone told me about Google Summer of Code. When I discovered that the D Foundation was participating, I immediately decided to take part and found that there was the need for a new XML library. So I contacted Craig Dillabaugh and Robert Schadek and started to plan my adventure. I want to take this occasion to thank them for their great continuous support, and the entire community for their feedback and help.

This was my first public codebase and my first contribution to a big open source project, so I didn't really know anything about project management. The advice about this field from my mentor Robert has been fundamental for my success; he helped me improve my workflow, keep my efforts focused towards the goal, and set up correctness tests and performance benchmarks. Without his help, I would never have been able to reach this point.

The first thing to do when writing a library is to pick a set of principles that will guide development. This choice is what will give the library its peculiar shape, and by having a look around one finds that there are XML libraries that want to be minimal in terms of codebase size, or very small in terms of binary size, or fully featured and 101% adherent to the specification. For [`std.experimental.xml`](https://github.com/lodo1995/experimental.xml), I decided to focus on genericity and extensibility. The processing is divided in many small, quite simple stages with well-defined interfaces implemented by templated components. The result is a pipeline that is fully customizable; you can add or substitute components anywhere, and add custom validation steps and custom error handlers.

From an XML library, a programmer expects different high level constructs: a SAX parser, a DOM parser, a DOM writer and maybe some extensions like XPath. He also expects to be able to process different kinds of input and, for `std.experimental.xml`, to "hack in" his own logic in the process. This requires a simple, yet very flexible, intermediate representation, which is produced by the parsing stage and can be easily manipulated, validated, and transformed into whatever high-level construct is needed. For this, I chose a concept called `Cursor`, a pointer inside an XML document, which can be queried for properties of a given XML node or advanced to a subsequent one. It's akin to [Java's StAX](https://docs.oracle.com/javase/tutorial/jaxp/stax/why.html) (Streaming API for XML), from which I took inspiration. In `std.experimental.xml`, all validations and transformations are implemented as chains of `Cursor`s, which are then usually processed by a SAX parser or a DOM builder, but can also be used directly in user code, providing more control and speed.

Talking about speed, which in XML processing can be very important, I have to admit that I didn't spend much time on optimization, leaving a lot of space for future performance improvements. Yet, the library is fast enough to guarantee that, for big files (where performance matters), an SSD (Solid State Drive) is needed to move the bottleneck from the fetching to the processing of the data. Being this is an extensible and configurable library, the user can choose his tradeoffs with fine granularity, trading input validation and higher level constructs for speed at will.

To conclude, the GSoC is finished, but the library is not. Although most parts are there, some bits are still missing. As a new university semester has started, time is becoming a rare and valuable resource, but I'll do my best to finish the work in a short time so that Phobos can finally have a modern XML library to be proud of. I also have a plan to add more advanced functionality, like XML Schemas and XPath, but I don't know when I'll manage to work on that, as it is quite a lot to do.
