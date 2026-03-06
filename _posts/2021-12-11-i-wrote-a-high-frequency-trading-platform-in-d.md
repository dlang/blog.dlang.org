---
author: GeorgeToutoungis
comments: false
date: 2021-12-11 13:49:07+00:00
layout: post
link: https://dlang.org/blog/2021/12/11/i-wrote-a-high-frequency-trading-platform-in-d/
slug: i-wrote-a-high-frequency-trading-platform-in-d
title: I Wrote a High-Frequency Trading Platform In D
wordpress_id: 3012
categories:
- Guest Posts
- User Stories
permalink: /i-wrote-a-high-frequency-trading-platform-in-d/
redirect_from: /2021/12/11/i-wrote-a-high-frequency-trading-platform-in-d/
---

![]({{ '/assets/images/i-wrote-a-high-frequency-trading-platform-in-d/brain02.png' | relative_url }})

I’ve used the D programming language to implement a high-frequency trading (HFT) platform. I’ve been quite satisfied with the experience and thought I’d share how I got here. It wasn’t a direct path.

In 2008, I stumbled across a book on Amazon [called Learn to Tango with D](http://amzn.to/1qds4Sj). That grabbed my curiosity, so I decided to research D further. That led me to Digital Mars and Walter Bright. I had first heard of Walter when I learned about [Zortech C++, the first native C++ compiler](http://www.edm2.com/index.php/Zortech_C%2B%2B). His work had been a huge influence on my C++ learning experience. So I was immediately interested in the language just because it was his, and excited to learn that he was working with Andrei Alexandrescu on version 2. Still, I decided to wait until they were further along with the new version before I dove in.

In 2010, I bought Andrei’s [The D Programming Language](http://amzn.to/1ZTDmqH) as soon as it was published and started reading. At the time, I was working [at BNP Paribas](https://group.bnpparibas/en/) using C++ to optimize their HFT platform, so high performance was prevalent in my thoughts. When I saw that D’s classes were reference types, with functions that are virtual by default, I was disappointed. I didn’t see how this could be useful for low-latency programming. I became too busy with work to explore further at the time, so I put the book and the language aside.

In 2014, I began preparing for a new adventure. As part of that, I started working on a feed handler framework from scratch in C++, using my own long-maintained C++ library of low-level components useful in low-latency, high-performance applications. Andrei’s book came to my attention again, so I decided to give it another look.

This time, I read the book through to the end and learned that my initial impression had been misplaced. I found that I liked D’s metaprogramming features and its support for programming in a functional style. By the end of the book, I was ready to give D a try.

I started by porting my C++ library and feed handler to D. It wasn’t difficult. I use very little inheritance in my C++ code, preferring composition and concrete classes. I found myself quite productive with D’s structs, templates, and mixins. All the while, I kept a close eye on performance benchmarks. When D turned out to give me the same performance as my C++ code, I was sold. I found D to be much more elegant, cleaner, more readable, and easier to maintain. I made the switch and never looked back.

My goal was to develop a complete HFT system using D. The system would consist of different subsystems:





  * Feed-Handler Framework: receives market data from exchanges; builds the books for all securities; publishes the updates to the other subsystems.


  * Strategies Framework: receives market data updates from feed handlers; facilitates communications with the Order Management System; allows for plugging into it strategies that make decisions on stock trades.


  * Order Management System: communicates with the exchange and the strategies framework; maintains a database of orders.


  * Signal Generator: receives market data updates from feed handlers; generates different signals as indicator values, predictions of stock prices, etc.; sends the different signals to strategies.



Ultimately, I found a new data structure and better design for my feed-handler framework. I developed the new version completely in D. This implementation heavily uses templates. I like D’s template syntax and generally find the error messages clearer than the complex error messages I was used to from C++. I needed to drop down to assembly for some specific x86 instructions and it was straightforward to do in D.

Later, I needed to work with configuration files. I prefer to write my config files in [Lua, a lightweight scripting language](https://www.lua.org/) that is easy to integrate into a program as an extension via its C API. For this, I found a D Lua binding called DerelictLua. Using, again, D’s metaprogramming facilities, I developed a very easy and practical way to interface with Lua on top of DerelictLua. _Editor’s Note:_ DerelictLua has since been discontinued; new projects should [use its successor, bindbc-lua, instead](https://code.dlang.org/packages/bindbc-lua).

The feed handler on the Bats market comes on 31 simultaneous channels, so it is more efficient to use multithreading. For this, I chose not to use the multithreading facilities provided by Phobos. I felt I needed more control in such a low-latency environment, particularly the ability to map each thread to a specific core. I opted to use the pthreads library and its affinity feature. D’s C ABI compatibility made it a straightforward thing to do.

I’m running on FreeBSD. For my intercommunication needs, I’m using kernel queues and sockets. The same functionality is available on macOS, my preferred development platform. D did not get in my way in using these APIs on either macOS or FreeBSD. It was as seamless as using kernel queues from C.

A few notes about problems and limitations:





  * I encountered one compiler bug. I found a workaround, so it wasn’t a blocker. I was able to reproduce it with a few lines of code and contacted the D community. They solved the problem and had a fix in a later version of the compiler.


  * I did not use D’s garbage collector. This is not a strike against D or its GC, though. In a low-latency system like this, even the use of `malloc` and `free` can be costly, so I’m not going to take a chance on a nondeterministic system with unpredictable latency. Instead, I used my library to handle allocation/deallocation via free lists, with memory preallocated upfront. As a consequence, I also decided not to use D’s standard library for anything.


  * I had to work with fixed-size ASCII strings that are not NUL-terminated and are, instead, padded with spaces at the end. Without the standard library, I found it easier to manipulate them C-style via pointers.



I was the sole developer on this project but completed it successfully in a relatively short period. Big credit to D and its productivity, readability, and ease of modifications.
