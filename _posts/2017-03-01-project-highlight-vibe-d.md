---
author: DBlogAdmin
comments: false
date: 2017-03-01 13:26:59+00:00
layout: post
link: https://dlang.org/blog/2017/03/01/project-highlight-vibe-d/
slug: project-highlight-vibe-d
title: 'Project Highlight: vibe.d'
wordpress_id: 671
categories:
- Project Highlights
- Web Development
permalink: /project-highlight-vibe-d/
redirect_from: /2017/03/01/project-highlight-vibe-d/
---

![](http://dlang.org/blog/wp-content/uploads/2017/03/product-icon.png)Since the day Sönke Ludwig first announced [vibe.d](http://vibed.org/) on [the D Forums](http://www.digitalmars.com/d/archives/digitalmars/D/announce/Introducing_vibe.d_23330.html#N23330), the project has been a big hit in the D community. It's the [exclusive subject](http://amzn.to/1qdrvrH) of one book, has [a chapter of its own](http://amzn.to/1IlQkZX) in another, and has been proven in production both commercially and otherwise. As so many projects do, it all started out of frustration.


I was dissatisfied with existing network web libraries (in particular with Node.js, which was the new big thing back then, because it was also built on an asynchronous I/O model). D 2.0 gained cross platform fiber support through the integration of [DRuntime](https://github.com/dlang/druntime), which seemed like a perfect opportunity to avoid the shortcomings of Node.js's programming model ("callback hell"). Together with D's strong type checking and the high performance of natively compiled applications this had the ideal preconditions for creating a network framework.


From the initial release, work progressed on adding web and REST interface generators ([vibe.web.web ](http://vibed.org/api/vibe.web.web/)and [vibe.web.rest](http://vibed.org/api/vibe.web.rest/), respectively).


This was made possible by D's advanced meta programming facilities, string mixins and compile-time reflection in particular. The eventual addition of user-defined attributes to the language enabled some important advances later on, such as the recently added [authorization framework](http://vibed.org/api/vibe.web.auth/).


Vibe.d is at its core an I/O and concurrency framework that makes heavy use of fibers which run in a quasi-parallel framework.


Every time an operation (e.g. reading from a socket) needs to wait, the fiber yields execution, so another fiber can run instead. Each fiber uses up very little memory compared to a full thread and switching between fibers is very cheap. This enables highly scalable applications that behave like normal multithreaded applications (save for the low-level issues associated with real multithreading).


At a higher level, it can serve as a web framework for backend development and provides functionality for protocols like HTTP and SMTP, database connectivity, and the parsing of data formats. A number of third-party packages that extend or complement vibe.d can be found in [the DUB repository](https://code.dlang.org/) (Sönke is also the creator and maintainer of [DUB](https://code.dlang.org/getting_started), the D build tool and package manager).

Big changes are currently afoot with the project. Beginning with the release of vibe.d 0.7.27 in February 2016, work began on splitting the monolithic project into independent DUB packages. One goal is to make it possible to use one vibe.d component without pulling them all in, reducing build times in the process.


Another goal is to employ modern D idioms where possible and to improve memory usage and performance as far as possible. It is surprising how much D evolved in just the short amount of time that vibe.d has been alive!


[Diet-NG](https://github.com/rejectedsoftware/diet-ng), vibe.d's template engine based on [Jade](https://www.npmjs.com/package/jade), was the first to be granted independence. It went through a complete rewrite that adds a strong test suite, makes use of D's ranges where possible, provides a more flexible API, and eliminates dependencies on other vibe.d packages. Now he's working on the core package.


The [vibe-core package](https://github.com/vibe-d/vibe-core) encapsulates the whole event and fiber logic, including I/O, tasks, concurrency primitives and general operating system abstraction. The original design was based heavily on classes and interfaces and had a very high level operating system abstraction layer, resulting in several downsides. For example, there was a dependence on the GC and virtual function calls could be an issue on certain platforms. One of the main goals was to minimize performance overhead in the new implementation.


As part of his experimentation with different API idioms and slimming down the code base, he produced the [eventcore library](https://github.com/vibe-d/eventcore).


The API follows a proactor pattern, meaning that a callback gets invoked whenever a certain asynchronous operation finishes. This is in contrast to the reactor pattern that is exposed by the non-blocking Posix APIs. It was chosen mainly so that asynchronous I/O APIs, such as Windows overlapped I/O and POSIX AIO, could be supported.

On top of eventcore, the fiber logic was implemented in a completely general way, using a central fiber scheduler and a generic `asyncAwait` function. This means that lots of corner cases are handled in a much more robust way now and that improvements in all areas can be made much faster and with fewer chances of breaking anything.


Next on the list for independence is the HTTP package. Sönke plans to completely rebuild the package from the ground up, adding HTTP/2 support and making it possible to enable allocation-free request/response processing.


Other obvious candidates are MongoDB and Redis clients, JSON and BSON support, the serialization framework, and the Markdown parser. The goal for each of these packages is always to take a look at the code and employ modern D idioms during the process.


If you've never taken D for a spin, [vibe.d](http://vibed.org/) is a fun playground in which to do so. It's not difficult to get up and running, and easier still if you have experience with such frameworks in other languages. It's also ready to put into production in its current state, despite the leading zero in the version number. As Sönke makes progress on breaking it up into separate packages, it will almost certainly become an even more integral part of the growing D community.
