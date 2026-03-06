---
author: StefanKoch
comments: false
date: 2017-04-10 12:59:16+00:00
layout: post
link: https://dlang.org/blog/2017/04/10/the-new-ctfe-engine/
slug: the-new-ctfe-engine
title: The New CTFE Engine
wordpress_id: 723
categories:
- Compilers &amp; Tools
- Core Team
- Guest Posts
permalink: /the-new-ctfe-engine/
redirect_from: /2017/04/10/the-new-ctfe-engine/
---

_Stefan Koch is the maintainer of [sqlite-d](https://github.com/UplinkCoder/sqlite-d), a native D [sqlite](https://www.sqlite.org) reader, and has contributed to projects like [SDC](https://github.com/SDC-Developers/SDC) (the Stupid D Compiler) and [vibe.d](http://vibed.org). He was also responsible for a 10% performance improvement in D's current CTFE implementation and is currently writing a new CTFE engine, the subject of this post._



* * *



For the past nine months, I've been working on a project called NewCTFE, a reimplementation of the [Compile-Time Function Evaluation (CTFE)](https://dlang.org/spec/function.html#interpretation) feature of the D compiler front-end. CTFE is considered one of the game-changing features of D.

As the name implies, CTFE allows certain functions to be evaluated by the compiler while it is compiling the source code in which the functions are implemented. As long as all arguments to a function are available at compile time and the function is pure (has no side effects), then the function qualifies for CTFE. The compiler will replace the function call with the result.

Since this is an integral part of the language, pure functions can be evaluated anywhere a compile-time constant may go. A simple example can be found in the [standard library](https://dlang.org/phobos/index.html) module, [`std.uri`](https://dlang.org/phobos/std_uri.html), where CTFE is used to compute a lookup table. It [looks like](https://github.com/dlang/Phobos/blob/master/std/uri.d) this:

```d
private immutable ubyte[128] uri_flags = // indexed by character
({

    ubyte[128] uflags;

    // Compile time initialize
    uflags['#'] |= URI_Hash;

    foreach (c; 'A' .. 'Z' + 1)
    {
        uflags[c] |= URI_Alpha;
        uflags[c + 0x20] |= URI_Alpha; // lowercase letters

    }

    foreach (c; '0' .. '9' + 1) uflags[c] |= URI_Digit;

    foreach (c; ";/?:@&=+$,") uflags[c] |= URI_Reserved;

    foreach (c; "-_.!~*'()") uflags[c] |= URI_Mark;

    return uflags;

})();
```
Instead of populating the table with magic values, a simple expressive [function literal](https://dlang.org/spec/expression.html#FunctionLiteral) is used. This is much easier to understand and debug than some opaque static array. The `({` starts a function-literal, the `})` closes it. The `()` at the end tells the compiler to immediately evoke that literal such that `uri_flags` becomes the result of the literal.

Functions are only evaluated at compile time if they need to be. `uri_flags` in the snippet above is declared in module scope. When a module-scope variable is initialized in this manner, the initializer must be available at compile time. In this case, since the initializer is a function literal, an attempt will be made to perform CTFE. This particular literal has no arguments and is pure, so the attempt succeeds.

For a more in-depth discussion of CTFE, see [this article](https://wiki.dlang.org/User:Quickfur/Compile-time_vs._compile-time) by H. S. Teoh at the D Wiki.

Of course, the same technique can be applied to more complicated problems as well; `std.regex`, for example, can build a specialized automaton for a regex at compile time using CTFE. However, as soon as [`std.regex`](https://dlang.org/phobos/std_regex.html) is used with CTFE for non-trivial patterns, compile times can become extremely high (in D everything that takes longer than a second to compile is bloat-ware :)). Eventually, as patterns get more complex, the compiler will run out of memory and probably take the whole system down with it.

The blame for this can be laid at the feet of the current CTFE interpreter's architecture. It's an AST interpreter, which means that it interprets the AST while traversing it. To represent the result of interpreted expressions, it uses DMD's AST node classes. This means that every expression encountered will allocate one or more AST nodes. Within a tight loop, the interpreter can easily generate over `100_000_000` nodes and eat a few gigabytes of RAM. That can exhaust memory quite quickly.

[Issue 12844](https://issues.dlang.org/show_bug.cgi?id=12844) complains about `std.regex` taking more than 16GB of RAM. For one pattern. Then there's [issue 6498](https://issues.dlang.org/show_bug.cgi?id=6498), which executes a simple `0` to `10_000_000` while loop via CTFE and runs out of memory.

Simply freeing nodes doesn't fix the problem; we don’t know which nodes to free and enabling the garbage collector makes the whole compiler brutally slow. Luckily there is another approach which doesn't allocate for every expression encountered. It involves compiling the function to a virtual ISA (Instruction Set Architecture). This virtual ISA, also known as bytecode, is then given to a dedicated interpreter for that ISA (in the case in which a virtual ISA happens to be the same as the ISA of the host, we call it a JIT (Just in Time) interpreter).

The NewCTFE project concerns itself with implementing such a bytecode interpreter. Writing the actual interpreter (a CPU emulator for a virtual CPU/ISA) is reasonably simple. However, compiling code to a virtual ISA is exactly as much work as compiling it to a real ISA (though, a virtual ISA has the added benefit that it can be extended for customized needs, but that makes it harder to do JIT later). That's why it took a month just to get the first simple examples running on the new CTFE engine, and why slightly more complicated ones still aren't running even after 9 months of development. At the end of the post, you'll find an approximate timeline of the work done so far.

I'll be giving [a presentation](http://dconf.org/2017/talks/koch.html) at [DConf 2017](http://dconf.org/2017/index.html), where I'll discuss my experience implementing the engine and explain some of the technical details, particularly regarding the trade-offs and design choices I've made. The current estimation is that the 1.0 goals will not be met by then, but I'll keep coding away until it's done.

Those interested in keeping up with my progress can follow my status updates [in the D forums](http://forum.dlang.org/thread/btqjnieachntljobzrho@forum.dlang.org). At some point in the future, I will write another article on some of the technical details of the implementation. In the meantime, I hope the following listing does shed some light on how much work it is to implement NewCTFE :)



 	
  * **May 9th 2016**
Announcement of the plan to improve CTFE.

 	
  * **May 27th 2016**
Announcement that work on the new engine has begun.

 	
  * **May 28th 2016**
Simple memory management change failed.

 	
  * **June 3rd 2016**
Decision to implement a bytecode interpreter.

 	
  * **June 30th 2016**
First code (taken from [issue 6498](https://issues.dlang.org/show_bug.cgi?id=6498)) consisting of simple integer arithmetic runs.

 	
  * **July 14th 2016**
ASCII string indexing works.

 	
  * **July 15th 2016**
Initial `struct` support

 	
  * **Sometime between July and August**
First switches work.

 	
  * **August 17th 2016**
Support for the special cases `if(__ctfe)` and `if(!__ctfe)`

 	
  * **Sometime between August and September**
Ternary expressions are supported

 	
  * **September 08th 2016**
First Phobos unit tests pass.

 	
  * **September 25th 2016**
Support for returning strings and ternary expressions.

 	
  * **October 16th 2016**
First (almost working) version of the LLVM backend.

 	
  * **October 30th 2016**
First failed attempts to support function calls.

 	
  * **November 01st**
DRuntime unit tests pass for the first time.

 	
  * **November 10th 2016**
Failed attempt to implement string concatenation.

 	
  * **November 14th 2016**
Array expansion, e.g. assignment to the `length` property, is supported.

 	
  * **November 14th 2016**
Assignment of array indexes is supported.

 	
  * **November 18th 2016**
Support for arrays as function parameters.

 	
  * **November 19th 2016**
Performance fixes.

 	
  * **November 20th 2016**
Fixing the broken `while(true) / for (;;)` loops; they can now be broken out of :)

 	
  * **November 25th 2016**
Fixes to `goto` and switch handling.

 	
  * **November 29th 2016**
Fixes to `continue` and `break` handling.

 	
  * **November 30th 2016**
Initial support for `assert`

 	
  * **December 02nd 2016**
Bailout on `void`-initialized values (since they can lead to undefined behavior).

 	
  * **December 03rd 2016**
Initial support for returning `struct` literals.

 	
  * **December 05th 2016**
Performance fix to the bytecode generator.

 	
  * **December 07th 2016**
Fixes to `continue` and `break` in `for` statements (`continue` _must not skip_ the increment step)

 	
  * **December 08th 2016**
Array literals with variables inside are now supported: `[1, n, 3]`

 	
  * **December 08th 2016**
Fixed a bug in `switch` statements.

 	
  * **December 10th 2016**
Fixed a nasty evaluation order bug.

 	
  * **December 13th 2016**
Some progress on function calls.

 	
  * **December 14th 2016**
Initial support for strings in switches.

 	
  * **December 15th 2016**
Assignment of static arrays is now supported.

 	
  * **December 17th 2016**
Fixing `goto` statements (we were ignoring the last `goto` to any label :)).

 	
  * **December 17th 2016**
De-macrofied string-equals.

 	
  * **December 20th 2016**
Implement check to guard against dereferencing null pointers (yes… that one was oh so fun).

 	
  * **December 22ed 2016**
Initial support for pointers.

 	
  * **December 25th 2016**
`static immutable` variables can now be accessed (yes the result is recomputed … who cares).

 	
  * **January 02nd 2017**
First Function calls are supported !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

 	
  * **January 17th 2017**
Recursive function calls work now :)

 	
  * **January 23rd 2017**
The **interpret3.d** unit-test passes.

 	
  * **January 24th 2017**
We are green on 64bit!

 	
  * **January 25th 2017**
Green on all platforms !!!!! (with blacklisting though)

 	
  * **January 26th 2017**
Fixed special case `cast(void*) size_t.max` (this one cannot go through the normal pointer support, which assumes that you have something valid to dereference).

 	
  * **January 26th 2017**
Member function calls are supported!

 	
  * **January 31st 2017**
Fixed a bug in `switch` handling.

 	
  * **February 03rd 2017**
Initial function pointer support.

 	
  * **Rest of Feburary 2017**
Wild goose chase for [issue #17220](https://issues.dlang.org/show_bug.cgi?id=17220)

 	
  * **March 11th 2017**
Initial support for slices.

 	
  * **March 15th 2017**
String slicing works.

 	
  * **March 18th 2017**
`$` in slice expressions is now supported.

 	
  * **March 19th 2017**
The concatenation operator (`c = a ~ b`) works.

 	
  * **March 22ed 2017**
Fixed a `switch` fallthrough bug.


