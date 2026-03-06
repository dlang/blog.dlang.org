---
author: DBlogAdmin
comments: false
date: 2017-07-28 13:23:05+00:00
layout: post
link: https://dlang.org/blog/2017/07/28/project-highlight-funkwerk/
slug: project-highlight-funkwerk
title: 'Project Highlight: Funkwerk'
wordpress_id: 1005
categories:
- Companies
- Project Highlights
permalink: /project-highlight-funkwerk/
redirect_from: /2017/07/28/project-highlight-funkwerk/
---

[![]({{ '/assets/images/project-highlight-funkwerk/funkwerk-logo.png' | relative_url }})](http://www.funkwerk.com/en/)[Funkwerk](http://www.funkwerk.com/en/) is a German company that develops intelligent communication technology. One of their projects is a passenger information system for long-distance and local transport that is deployed by long-distance rail networks in Germany, Austria, Switzerland, Finland, Norway and Luxembourg, as well as city railways in Berlin and Munich. The system is developed at the company's Munich location and, some time ago, they came to the conclusion that it needed a rewrite. According to Funkwerk's Mario Kröplin:


From a bird’s-eye view, the long-term replacement of our aged passenger information system is our primary project. At some point, it became obvious that the system was getting hard to maintain and hard to change. In 2008, a new head of the development department was hired. It was time for a change.


They wanted to do more than just rewrite the system. They also wanted to make changes in their development process, and that led to questions of how best to approach the changes so that they didn't negatively impact productivity.


There is an inertia when experienced programmers are asked to suddenly start with unit testing and code reviews. The motivation for unit testing and code reviews is higher when you learn a new programming language. Especially one where unit testing is a built-in feature. It takes a new language to break old habits.


The decision was made to select a different language for the project, preferably one with built-in support for unit testing. They also allowed themselves a great deal of leeway in making that choice.


Compared with other monolithic legacy systems, we were in the advantageous position that the passenger information system was constructed as a set of services. There is some interprocess communication between the services, but otherwise the services are largely independent. The service architecture made such a decision less serious: just make an experiment with any one of the services in the new language. Should the experiment fail, then it’s not a heavy loss.


So they cast out a net and found D. One thing about the language that struck them immediately was that it fit in a comfort zone somewhere between the languages with which their team was already familiar.


The languages in use were C++ (which rather feels like being part of the problem than part of the solution) and Java. D was presented as a better C++. With garbage collection, changing to D is easy enough for both C++ and Java programmers.


They took D for a spin with some experimentation. The first experiment was a bit of a failure, but the second was a success. In fact, it went well enough that it convinced them to move forward with D.


In retrospect, the change of programming language was one aspect that led to an agile transition. One early example of a primary D project was the replacement of the announcement service. This one gets all information about train journeys - especially about the disruptions - and decides when to play which announcements. The business rules make this quite complicated. The new service uses object-oriented programming to handle the complicated business rules nicely. So you could say it's a Java program written in D. However, as the business rules change from customer to customer, the announcement service is constantly changing. And with the development of D and with our learning, we are now in a better position to evolve the software further.


When they first started the changeover, D2 was still in its early stages of development and the infamous Phobos/Tango divide was still a thing in the D community (an issue that, for those of you out of the loop the last several years, [was put to rest](https://semitwist.com/articles/article/view/dispelling-common-d-myths#dmyths-d1d2) long ago). Since D1 was considered stable, that was an obvious choice for them. They also chose Tango over Phobos, primarily because Tango provided logging and HTTP facilities, while Phobos did not. In 2012, as D2 was stablizing and official support for D1 was being phased out, it became apparent that they would have to make a transition from D1 and Tango to D2 and Phobos, which is what they use today.

One of the D features that hooked Mario early on was its built-in support for contracts.


Contracts won me over from the beginning. No joke! When we had hard times to hunt down segmentation faults, a failed assertion boosted the diagnostics. I used the assert macro a lot in C++, but with contracts in D there is no question about whom to blame. A failed precondition: caller's fault. A failed postcondition or invariant: my fault. We have contracts everywhere. And we don't use the `-release` switch. We would rather give up performance than information that helps us fixing bugs.


As with many other active D users, D's [dynamic arrays](https://tour.dlang.org/tour/en/basics/arrays), [slices](https://tour.dlang.org/tour/en/basics/slices) and the built-in [associative arrays](https://tour.dlang.org/tour/en/basics/associative-arrays) stand out for the Funkwerk programmers


They work out of the box. It's not like in Java, where you'd better not use the arrays of the language, but the `ArrayList` of the library.


They also have put D's [ranges](https://tour.dlang.org/tour/en/basics/ranges) and [Uniform Function Call Syntax](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs) to heavy use.


With ranges and UFCS, it became possible to replace loops with intention-revealing chains of `map` and `filter` and whatever. While the loop describes how something is done, the ranges describe what is done. We enjoy setting the delay in our tests to `5.minutes`. And we use UDAs in [accessors](https://github.com/funkwerk/accessors) and [dunit](https://github.com/linkrope/dunit). It's hard to imagine what pieces of our clean code would look like in any other language.


_accessors_ is a library that uses [templates](https://tour.dlang.org/tour/en/basics/templates), [mixins](https://tour.dlang.org/tour/en/gems/string-mixins), and [UDAs](https://tour.dlang.org/tour/en/gems/attributes) to automatically generate property getters and setters. _dunit_ is a unit testing library that, in an ironic twist, Mario maintains. As it turned out, their intuition that their developers would be more motivated to write code when a language has built-in unit testing was correct (that's been cited by Walter Bright as a reason it was included in the language in the first place). However, that first failed experiment in D came down to the simplistic capabilities the built-in unit testing provides.


One of the failures of our first experiment was an inappropriate use of the `unittest` functions. You're on your own when you have to write more code per test case, for example for testing interactions of objects. With [xUnit](https://en.wikipedia.org/wiki/XUnit) testing frameworks like [JUnit](http://junit.org/junit4/), you get a toolbox and lots of instructions, like the exhaustive [http://xunitpatterns.com/](http://xunitpatterns.com/). With D, you get the single tool `unittest` and lots of examples of test cases that can be expressed as one-liners. This does not help in scenarios where such test cases are the exception. Even Phobos has examples where the `unittest` blocks are lengthy and obscure. Often, such `unittest` blocks are not clean code but a collection of "Test Smells". We soon replaced the built-in unit testing (one of the benefits we’ve seen) with [DUnit](http://www.dsource.org/projects/dmocks/wiki/DUnit).


The transition to D2 and Phobos necessitated a move away from the Tango-based DUnit. Mario found the open source (and lowercase) dunit (the original project is [here](https://github.com/jmcabo/dunit)) a promising replacement, and ultimately found himself maintaining [a fork](https://github.com/linkrope/dunit) that has evolved considerably from the original. However, the team recently discovered a way to combine xUnit testing with D's built-in `unittest`, which may lead to another transition in their unit testing.

Additionally, Phobos still had no logging package in 2012, so they needed a replacement for the Tango logger API. Mario rolled up his sleeves and implemented [log](https://github.com/linkrope/log), which, like acessors and dunit, is available under the Boost Software License.


It was meant as an improvement proposal for an early stage of  [`std.experimental.logger`](http://dlang.org/phobos/std_experimental_logger.html). I was excited that it was possible to implement the logging framework together with support for rolling log files, for `logrotate`, and for `syslog` in just 500 lines. We're still using it.


Funkwerk has also contributed code to Phobos.


I was looking for a way to implement a delay calculation. The problem is that you have an initial delay and you assume that the driver runs faster and makes shorter stops in order to catch up. But the algorithm was missing. So we made a pull request for [`std.algorithm.cumulativeFold`](http://dlang.org/phobos/std_algorithm_iteration.html#.cumulativeFold). It’s there because we strived for clean code in a forecast service.


Garbage collection has been [a topic of interest](http://dlang.org/blog/the-gc-series/) on this blog lately. Mario has an interesting anecdote from Funkwerk's usage of D's GC.


Our service was losing connections from time to time. It took a while to find that the cause was the interaction between garbage collection and networking. The signals used by the garbage collector interrupt the blocking read. The blocking read is hidden in a library, for example, in `std.socketstream`, where the retry is missing. Currently, we abuse inheritance to patch the `SocketStream` class.


The technical description of the derived class from the [Ddoc comments](https://tour.dlang.org/tour/en/gems/documentation) read:

    /**
     * When a timeout has been set on the socket, the interface functions "recv"
     * and "send" are not restarted after being interrupted by a signal handler.
     * They fail with the error "EINTR" when interrupted, especially
     * by the signal used to "stop the world" during garbage collection.
     *
     * In this case, retries avoid that the stream is mistaken to be exhausted.
     * Note, however, that these retries will likely exceed the specified timeout.
     *
     * See also: Linux manual page signal(7)
     */
After nearly a decade of active development with D, Mario and his team have been pleased with their choice. They plan to continue using the language in new projects.


Over the years, we've replaced services in the periphery of the passenger information system whenever this was justified by customer needs. We chose D for the implementation of all of the backend services. We are now working on a small greenfield project (it’s a few buses to be monitored instead of lots of trains), but we intend to grow this project into the next generation of the passenger information system.


In the coming months, we'll hear more from Mario and the Funkwerk developers about how they use D to develop their system and tooling, and what it's like to be a professional D programmer.
