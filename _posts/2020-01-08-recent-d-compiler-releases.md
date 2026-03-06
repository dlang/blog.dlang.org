---
author: DBlogAdmin
comments: false
date: 2020-01-08 08:43:40+00:00
layout: post
link: https://dlang.org/blog/2020/01/08/recent-d-compiler-releases/
slug: recent-d-compiler-releases
title: Recent D Compiler Releases
wordpress_id: 2267
categories:
- DMD Releases
- LDC Releases
- News
permalink: /recent-d-compiler-releases/
redirect_from: /2020/01/08/recent-d-compiler-releases/
---

![Digital Mars D logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)The LDC team closed out the old year with [release 1.19.0 of the LLVM-based D compiler](https://github.com/ldc-developers/ldc/releases/tag/v1.19.0), and the core D team opened the new year with [version 2.090.0 of the reference D compiler](https://dlang.org/changelog/2.090.0.html), DMD. And if you haven't yet heard, there was some big news about [the GCC-based D compiler](https://github.com/D-Programming-GDC/gcc), GDC, a while back. Time to catch up!


### LDC 1.19.0


This release updates the LDC compiler to D front end version 2.089.1, which was the current version when the compiler was released on the day after Christmas. The prebuilt packages are [based on LLVM 9.01](http://releases.llvm.org/).

Among the big items in this release is some love for Android. The prebuilt DRuntime/Phobos library is now available for all supported Android targets. This release can be used in conjunction with [Adam Ruppe's D Android project](https://github.com/adamdruppe/d_android), a collection of helper programs and interfaces, [currently in beta](https://forum.dlang.org/post/cqdqhrporufpzkwjwkrw@forum.dlang.org), to facilitate D development on Android with LDC.

Windows users will find that the bundled MinGW-based link libraries for Windows development have been upgraded. They are now derived from `.def` files from [the MinGW-w64 7.0.0 package](http://mingw-w64.org/doku.php). These libraries allow you to use the Windows system libraries without needing to install the Windows SDK.


### DMD 2.090.0


The latest version of [DMD was announced on January 7th](https://forum.dlang.org/post/qv1mji$f81$1@digitalmars.com). It ships with 10 major changes and [71 closed issues courtesy of 48 contributors](https://dlang.org/changelog/2.090.0.html).

With this release, it's now possible to do more with lazy parameters. D [has long supported lazy parameters](https://dlang.org/spec/function.html#lazy-params):


An argument to a lazy parameter is not evaluated before the function is called. The argument is only evaluated if/when the parameter is evaluated within the function. Hence, a lazy argument can be executed 0 or more times.


Under the hood, they are implemented as delegates. Now, it's possible to get at the underlying delegate by taking the address of the parameter, an operation which was previously illegal.

```d
import std.stdio;

void chillax(lazy int x)
{
    auto dg = &x;
    assert(dg() == 10);
    writeln(x);
}

void main()
{
    chillax(2 * 5);
}
```
This release also renders obsolete [a D idiom](https://p0nce.github.io/d-idioms/#GC-proof-resource-class) used by those who find themselves with a need to distinguish between finalization (non-deterministic object destruction usually initiated by the garbage collector) and normal destruction (deterministic object destruction) from inside a class or struct destructor.

With the current GC implementation, it's illegal to perform some GC operations during finalization. However, D does not provide for separate finalizers and destructors. There is only `~this`, which is referred to as a destructor even though it fills both roles. This sometimes presents difficulties when implementing destructors for types that are intended to be used with both GC and non-GC allocation. Any cleanup activity that touches the GC could throw an `InvalidMemoryOperationError`. Hence the need for the aforementioned workaround.

Now it's possible to call the static `GC` member function, `core.memory.GC.inFinalizer`, to get your bearings in a destructor. It returns `true` if the current thread is performing object finalization, in which case you don't want to be taking any actions that touch on GC operations. (I've been waiting for something like this before writing the next article [in my GC series](https://dlang.org/blog/the-gc-series/).)


### GDC


Thanks to the hard work of Iain Buclaw, Johannes Pfau, and all of the volunteers who have maintained and contributed to it over the years, GDC was [accepted into GCC 9](https://gcc.gnu.org/gcc-9/) in late 2018 and made available as part of the GCC 9.1 package released in May of last year. GCC 9.2 was released last August. This version of GDC implements version 2.076 of the D front end. [You can build it yourself](https://github.com/D-Programming-GDC/gcc) or install it from the same place you install the GCC 9.x series.
