---
author: DBlogAdmin
comments: false
date: 2016-12-30 13:19:26+00:00
layout: post
link: https://dlang.org/blog/2016/12/30/project-highlight-voxelman/
slug: project-highlight-voxelman
title: 'Project Highlight: Voxelman'
wordpress_id: 533
categories:
- Game Development
- Project Highlights
permalink: /project-highlight-voxelman/
redirect_from: /2016/12/30/project-highlight-voxelman/
---

![](http://dlang.org/blog/wp-content/uploads/2016/12/mrsmith33-200x200.png)If you spend any time over at [r/VoxelGameDev](https://www.reddit.com/r/VoxelGameDev/), you may have seen posts about [Voxelman](https://github.com/MrSmith33/voxelman), the plugin-driven game engine MrSmith33 is developing with D. His real name is Andrey Penechko, and he started work on Voxelman after he was inspired by Minecraft to think about all the cool things he could do with a voxel engine, particularly the low-level optimization tricks he could use in implementing one. Then he jumped in and started figuring things out.


I started the project somewhere in 2011 or 2012. It began with creating an [SDL](http://libsdl.org/) window and getting some triangles on the screen. Then I did cubes, then a single chunk. It was a simple, single-threaded thing. I did it all with a fixed camera and only had rudimentary camera controls.


For that initial version of the project, he was using C++, but he found himself stuck from a lack of knowledge about the language. So he started searching to see what else was out there. That led him to D.


I don't really remember how I found D. I was in need of some statically typed compiled language other than C++. I was frustrated about all the source file organisation, the need of forward declarations, header separation and the include system. In D, it was as simple as writing code. I bought a cheap 10 inch tablet just to read [Andrei's book](https://www.amazon.com/gp/product/0321635361/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321635361&linkCode=as2&tag=dlang-20&linkId=BOLS7NQK6MXCZTMG), because my 3.2" PPC was too small to read the whole thing. I enjoyed reading every single bit of it.


His ultimate goal with the project is to provide a platform for which people can create and share plugins and game worlds.


Ideally a complete project build should have the engine source and tools (launcher, source editor, compiler). Players should be able to initiate a connection to any server in the server list, then the launcher will download any missing plugins, compile a new executable and start the engine with the list of plugins. Currently, a build of Voxelman is less than 3MB in size. I think that this is a good property to have.


The major sticking point he sees with this approach is the dependency DMD has on the Microsoft tools for 64-bit (and 32-bit COFF) support on Windows (specifically the Windows SDK and the Microsoft linker). Even though the MS linker is considered the system linker, it's not uncommon to see Cygwin and or one of the various distributions of MinGW installed instead of the MS tools. In a perfect world, he could tell people to download the D compiler and they would have everything they need. But it's not a deal-breaker, so he's not letting it stop him.

Voxelman uses a client-server architecture, where the server can be launched in a dedicated process or as part of the client's. This is managed by a launcher which, in addition to launching the game, can be used to compile projects, manage the world, and find servers to connect with.

World and mesh generation is multi-threaded and, as in most such engines, the model is chunk-based. The chunk management implementation is informed by the concept of [entity component systems](http://www.gamedev.net/page/resources/_/technical/game-programming/understanding-component-entity-systems-r3013), with a chunk's world position serving as its entity ID and layers functioning as components.


Each dimension is broken into chunks. A chunk is a 32³ array of blocks. Each chunk can have a set of data layers (currently blocks and block entities). Each layer is essentially an immutable snapshot. It can be of different storage types (uniform, where all blocks are the same,  or a compressed or full array, where the layer stores an array of data). Those layers then can be freely transmitted between threads, with reference counting done in the main thread. When a layer is no longer needed it's deleted.


Immutable chunk data makes for fast auto saves of chunk snapshots in a separate IO thread.


When a chunk is received on the client side, it can be sent to a worker thread and the geometry will be generated. Snapshots are sent to the IO thread when save points occur, and they can still be used in the main thread, sent to the client, or processed by other worker threads. One can easily use an old snapshot while several new ones are in use. Whenever a layer is being modified, data is copied into a write buffer, changes are made, and at a commit point at the end of the frame, all write buffers are committed to chunk storage.


Andrey calls his plugin system "semi-hackish".


All plugins inherit from an `IPlugin` interface. Then, each plugin registers itself in a global table of plugins from a [shared static constructor](https://dlang.org/spec/module.html#staticorder). The global table has lists for server and client plugins. The engine adds those plugins to the plugin manager based on a provided plugin pack. The plugin manager implements the initialization sequence. When starting initialization, you have lots of dependencies, so you need to run things in a specific order.


He has found a lot of things to like about D. As major pros, he cites [the module system](https://dlang.org/spec/module.html) ("no forward declarations"), [foreach ](https://dlang.org/spec/statement.html#ForeachStatement)loops ("99% of loops in my code are these guys"), [associative arrays](https://dlang.org/spec/hash-map.html), [delegates](https://dlang.org/spec/function.html#closures), and [templates ](https://dlang.org/spec/template.html)("They're beautiful; you simply add another set of parentheses and you're done"). He also loves D's [dynamic arrays](https://dlang.org/spec/arrays.html#dynamic-arrays) ([slices](https://dlang.org/d-array-article.html)).


They are a perfect design, with the pointer and the length bundled together. You can append to them, concatenate them, and change their length.


As minor pros, he lists D's [Compile-Time Function Execution](https://dlang.org/spec/function.html#interpretation) and its [code generation](https://dlang.org/mixin.html) and compile-time [introspection](https://dlang.org/spec/traits.html) features. Unlike some D users, he also counts the garbage collector in that group. He has implemented a mix of GC-ed and non-GCed memory in Voxelman.


High-level stuff is fully in GC memory. I call something high-level if it has only one instance, so I use interfaces/classes for the high-level parts. Low-level things are mostly stack allocated, using structs (which are POD in D), and the most performance sensitive and memory consuming parts use manual memory management (via [Mallocator](https://dlang.org/phobos/std_experimental_allocator_mallocator.html)). This includes chunk storage and chunk meshes.


He also has a list of rough corners. He doesn't like that support for DLLs is [not yet fully functional](http://dconf.org/2016/talks/thaut.html) and reliable. He has found problems when trying to use `shared` (for example, the [`Mutex`](http://dlang.org/phobos/core_sync_mutex.html) class [cannot be used with it](http://forum.dlang.org/thread/pikvwyoexbklnvdmedxr@forum.dlang.org)). He also finds all the use cases of the [is expression](http://dlang.org/spec/expression.html#IsExpression) confusing, saying the syntax "feels like regular expressions for templates; very powerful and concise, but hard to understand."

His difficulties with `shared` actually took him down an interesting path that ultimately had a positive impact on performance.


I started my multi-threading by using the `send` and `receive` functions from [`std.concurrency`](https://dlang.org/phobos/std_concurrency.html). I found that I needed to send messages of variable length. For example, when loading or saving chunks, you need to send all the layers to another thread. This involved allocating arrays for all the layers and also required the use of `shared`.

This situation led me to the implementation of a lock-free message queue, where each message is just a stream of bytes. You write variables on one end and read them from the other. This is obviously a single producer, single consumer queue.

A disadvantage was the use of a fixed-size circular array. You need to make sure that the queue doesn't fill up. This was a point where I found a good book that explains how atomics work: [C++ Concurency in Action: Practical Multithreading](https://www.amazon.com/C-Concurrency-Action-Practical-Multithreading/dp/1933988770/ref=as_li_ss_tl?ie=UTF8&qid=1483101629&sr=8-1&keywords=c+++concurrency+in+action&linkCode=ll1&tag=dlang-20&linkId=ed739632531b6940a3468edb36871d8f). This is one of the places in D's documentation where you feel a lack of pointers on where to find relevant information on a specific topic.

So the new solution doesn't require any allocations and is actually faster than the built-in one. Later I added a notification system via [`Semaphore`](http://dlang.org/phobos/core_sync_semaphore.html), so that worker threads wait when out of work.


If you're looking for an open source D game to contribute to, [Voxelman](https://github.com/MrSmith33/voxelman) is waiting for you. You can [read more about some of its internals](https://www.reddit.com/r/VoxelGameDev/comments/4znnyo/voxelman_v070_block_entities_dimensions_rails/) on reddit, check out [some images on imgur](http://imgur.com/a/L5g1B), and watch [some videos on YouTube](https://www.youtube.com/channel/UCFiCQez_ZT2ZoBBJadUv3cA). I'll leave you with this example of it in action:


