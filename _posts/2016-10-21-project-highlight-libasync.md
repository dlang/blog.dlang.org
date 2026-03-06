---
author: DBlogAdmin
comments: false
date: 2016-10-21 13:31:10+00:00
layout: post
link: https://dlang.org/blog/2016/10/21/project-highlight-libasync/
slug: project-highlight-libasync
title: 'Project Highlight: libasync'
wordpress_id: 459
categories:
- Project Highlights
- Web Development
permalink: /project-highlight-libasync/
redirect_from: /2016/10/21/project-highlight-libasync/
---

[![d6](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)libasync ](https://github.com/etcimon/libasync)is a cross-platform event loop library written completely in D.  It was created, and continues to be maintained, by Etienne Cimon, who started it as a native driver for [vibe.d](http://vibed.org/), a modular asynchronous I/O framework most often used for web app development in D.


In 2014 or so, I was looking for a framework to power my future web development projects. I wasn't going to use an interpreted language, as binary executables were too attractive. I found vibe.d appealing because, coming from C++, it was relatively simple and featureful. So I studied it, along with the D programming language and [the Phobos standard library](http://dlang.org/phobos/index.html).


vibe.d has always used libevent under the hood by default. This is where Etienne ran into a problem that bothered him.


I stumbled on some workflow issues when deploying vibe.d apps to other operating systems which may or may not have the right version of libevent in the package repository. I didn't want to package a DLL with my server, or have to go through dependency hell with my software, and I wanted everything to be consistently written in D to reduce the mental complexity of switching programming languages or to debug other issues.


So he decided to study up on the system APIs across the platforms supported by DMD (Windows, Linux, *BSD and OS X) and create his own event loop library in D. Now he, and anyone using libasync, can issue a single command with [DUB](https://code.dlang.org/getting_started) to compile and execute a web application without needing to worry about external event loop dependencies.

libasync takes advantage of D's delegates to provide a very intuitive interface.

```d
void testDNS() {
	auto dns = new shared AsyncDNS(g_evl);
	dns.handler((NetworkAddress addr) {
		writeln("Resolved to: ", addr.toString(), ", it took: ", g_swDns.peek().usecs, " usecs");
	}).resolveHost("127.0.0.1");
}
```
Etienne says of the code snippet above:


The D garbage collector will keep the `AsyncDNS` object in `dns` alive for as long as the delegate used in the parameter of `dns.handler` is alive in the heap, which is in this object. The delegate syntax is more simple to declare than Javascript, and it is also type-safe. This DNS resolver will work on any platform thrown at it, thanks to D's compile-time [version conditions](http://dlang.org/spec/version.html#VersionCondition).


libasync makes use of the asynchronous I/O facilities available on each supported platform and provides a number of event-handlers out of the box.


Cross-platform event handlers have been defined for DNS resolution, UDP Messages, (Buffered/Unbuffered) TCP Connections, TCP Listeners, File Operations, Thread-local (Notifiers) and Cross-thread Signals, Timers and File Watchers. The intrinsics involve EPoll for Linux, KQueue for OS X and BSD, and overlapped I/O for Windows. With all of these features thoroughly tested through a vibe.d driver, libasync has become a very fast and reliable library which I use in all of my projects. My benchmarks show it as being a little slower than the libevent driver in vibe.d, though its self-explanatory code base makes it seamless to understand, maintain, and deploy.


A libasync driver has been added to vibe.d and work is going on to improve the library's performance.


The stability of the underlying OS features makes for very little need for changes, although there is a big improvement involving [the proactor pattern](https://en.wikipedia.org/wiki/Proactor_pattern) in the works for [libasync](https://github.com/Calrama/libasync/tree/develop) and [a new architecture for vibe.d](https://github.com/vibe-d/eventcore/commit/95ccc347d587610e1ba7545ce4c01eedf241febc). Together, those two developments are likely to increase the library's performance significantly.


If you find yourself needing an event loop in D and want to give libasync a spin, you can visit [the library's page at the DUB repository](https://code.dlang.org/packages/libasync) for information on how to add it as a dependency to your own DUB-managed projects. libasync, in turn, has only one dependency itself, another library maintained by Etienne that provides a set of allocators and allocator-friendly containers called [memutils](https://github.com/etcimon/memutils).

It wasn't so long ago that anyone using D who wanted something like libasync or memutils would need to either roll their own or bind to a C library. The ever-expanding list of libraries in [the DUB repository](https://code.dlang.org/), created and made available by members of the D community like Etienne, make it much easier to jump into D today than ever before.
