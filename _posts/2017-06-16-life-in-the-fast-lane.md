---
author: DBlogAdmin
comments: false
date: 2017-06-16 13:37:55+00:00
layout: post
link: https://dlang.org/blog/2017/06/16/life-in-the-fast-lane/
slug: life-in-the-fast-lane
title: Life in the Fast Lane
wordpress_id: 888
categories:
- Code
- GC
permalink: /life-in-the-fast-lane/
redirect_from: /2017/06/16/life-in-the-fast-lane/
---

[![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)The first post](https://dlang.org/blog/2017/03/20/dont-fear-the-reaper/) I wrote in the [GC series](http://dlang.org/blog/the-gc-series/) introduced the D garbage collector and the language features that use it. Two key points that I tried to get across in the article were:



 	
  1. **The GC can only run when memory allocations are requested**. Contrary to popular misconception, the D GC isn’t generally going to decide to pause your Minecraft clone in the middle of the hot path. It will only run when memory from the GC heap is requested, and then only if it needs to.

 	
  2. **Simple C and C++ allocation strategies can mitigate GC pressure**. Don’t allocate memory in inner loops – preallocate as much as possible, or fetch it from the stack instead. Minimize the total number of heap allocations. These strategies work because of point #1 above. The programmer can dictate when it is possible for a collection to occur simply by being smart about when GC heap allocations are made.


The strategies in point #2 are fine for code that a programmer writes herself, but they aren't going to help at all with third-party libraries. For those situations, D provides built-in mechanisms to guarantee that no GC allocations can occur, both in the language and the runtime. There are also command-line options that can help make sure the GC stays out of the way.

Let’s imagine a hypothetical programmer named J.P. who, for reasons he considers valid, has decided he would like to avoid garbage collection completely in his D program. He has two immediate options.


#### The GC chill pill


One option is to make a call to `GC.disable` when the program is starting up. This doesn't stop allocations, but puts a hold on collections. That means _all_ collections, including any that may result from allocations in other threads.

```d
void main() {
    import core.memory;
    import std.stdio;
    GC.disable;
    writeln("Goodbye, GC!");
}
```
Output:

    Goodbye, GC!
This has the benefit that all language features making use of the GC heap will still work as expected. But, considering that allocations are still going without any cleanup, when you do the math you'll realize this might be problematic. If allocations start to get out of hand, something's gotta give. From [the documentation](http://dlang.org/phobos/core_memory.html#.GC.disable):


Collections may continue to occur in instances where the implementation deems necessary for correct program behavior, such as during an out of memory condition.


Depending on J.P.'s perspective, this might not be a good thing. But if this constraint is acceptable, there are some additional steps that can help keep things under control. J.P. can make calls to `GC.enable` or `GC.collect` as necessary. This provides greater control over collection cycles than the simple C and C++ allocation strategies.


#### The GC wall


When the GC is simply intolerable, J.P. can turn to the `@nogc` attribute. Slap it at the front of the `main` function and thou shalt suffer no collections.

```d
@nogc
void main() { ... }
```
This is the ultimate GC mitigation strategy. `@nogc` applied to `main` will guarantee that the garbage collector will never run anywhere further along the callstack. No more caveats about collecting “where the implementation deems necessary”.

At first blush, this may appear to be a much better option than `GC.disable`. Let’s try it out.

```d
@nogc
void main() {
    import std.stdio;
    writeln("GC be gone!");
}
```
This time, we aren’t going to get past compilation:

    Error: @nogc function 'D main' cannot call non-@nogc function 'std.stdio.writeln!string.writeln'
What makes `@nogc` tick is the compiler’s ability to enforce it. It’s a very blunt approach. If a function is annotated with `@nogc`, then any function called from inside it must also be annotated with `@nogc`. As may be obvious, `writeln` is not.

That’s not all:

```d
@nogc 
void main() {
    auto ints = new int[](100);
}
```
The compiler isn’t going to let you get away with that one either.

    Error: cannot use 'new' in @nogc function 'D main'
Any language feature that allocates from the GC heap is out of reach inside a function marked `@nogc` (refer to [the first post in this series](https://dlang.org/blog/2017/03/20/dont-fear-the-reaper/) for an overview of those features). It’s turtles all the way down. The big benefit here is that it guarantees that third-party code can’t use those features either, so can’t be allocating GC memory behind your back. Another downside is that any third-party library that is not `@nogc` aware is not going to be available in your program.

Using this approach requires a number of workarounds to make up for non-`@nogc` language features and library functions, including several in the standard library. Some are trivial, some are not, and others can’t be worked around at all (we’ll dive into the details in a future post). One example that might not be obvious is throwing an exception. The idiomatic way is:

```d
throw new Exception("Blah");
```
Because of the `new` in that line, this isn’t possible in `@nogc` functions. Getting around this requires preallocating any exceptions that will be thrown, which in turn runs into the issue that any exception memory allocated from the regular heap still needs to be deallocated, which leads to ideas of reference counting or stack allocation… In other words, it’s a big can of worms. There’s currently [a D Improvement Proposal](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1008.md) from Walter Bright intended to stuff all the worms back into the can by making `throw new Exception` work without the GC when it needs to.

It’s not an insurmountable task to get around the limitations of `@nogc main`, it just requires a good bit of motivation and dedication.

One more thing to note about `@nogc main` is that it doesn’t banish the GC from the program completely. D has support for [static constructors and destructors](https://dlang.org/spec/class.html#static-constructor). The former are executed by the runtime before entering `main` and the latter upon exiting. If any of these exist in the program and are not annotated with `@nogc`, then GC allocations and collections can technically be present in the program. Still, `@nogc` applied to `main` means there won't be any collections running once `main` is entered, so it's effectively the same as having no GC at all.


#### Working it out


Here’s where I’m going to offer an opinion. There’s a wide range of programs that can be written in D without disabling or cutting the GC off completely. The simple strategies of minimizing GC allocations and keeping them out of the hot path will get a lot of mileage and _should be preferred_. It can’t be repeated enough given how often it’s misunderstood: D’s GC will only have a chance to run when the programmer allocates GC memory and it will only run if it needs to. Use that knowledge to your advantage by keeping the allocations small, infrequent, and isolated outside your inner loops.

For those programs where more control is actually needed, it probably isn’t going to be necessary to avoid the GC entirely. Judicious use of `@nogc` and/or the `core.memory.GC` API can often serve to avoid any performance issues that may arise. Don’t put `@nogc` on `main`, put it on the functions where you really want to disallow GC allocations. Don’t call `GC.disable` at the beginning of the program. Call it instead before entering a critical path, then call `GC.enable` when leaving that path. Force collections at strategic points, such as between game levels, with `GC.collect`.

As with any performance tuning strategy in software development, it pays to understand as fully as possible what’s actually happening under the hood. Adding calls to the `core.memory.GC` API in places where you _think_ they make sense could potentially make the GC do needless work, or have no impact at all. Better understanding can be achieved with a little help from the toolchain.

The DRuntime [GC option](https://dlang.org/spec/garbage.html#gc_config) `--DRT-gcopt=profile:1` can be passed to a compiled program (not to the compiler!) for some tune-up assistance. This will report some useful GC profiling data, such as the total number of collections and the total collection time.

To demonstrate, **gcstat.d** appends twenty values to a dynamic array of integers.

```d
void main() {
    import std.stdio;
    int[] ints;
    foreach(i; 0 .. 20) {
        ints ~= i;
    }
    writeln(ints);
}
```
Compiling and running with the GC profile switch:

```bash
dmd gcstat.d
gcstat --DRT-gcopt=profile:1
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
        Number of collections:  1
        Total GC prep time:  0 milliseconds
        Total mark time:  0 milliseconds
        Total sweep time:  0 milliseconds
        Total page recovery time:  0 milliseconds
        Max Pause Time:  0 milliseconds
        Grand total GC time:  0 milliseconds
GC summary:    1 MB,    1 GC    0 ms, Pauses    0 ms <    0 ms
```
This reports one collection, which almost certainly happened as the program was shutting down. The runtime terminates the GC as it exits which, in the current implementation, will generally trigger a collection. This is done primarily to run destructors on collected objects, even though D does not require destructors of GC-allocated objects to ever be run (a topic for a future post).

DMD supports a command-line option, `-vgc`, that will display every GC allocation in a program, including those that are hidden behind language features like the array append operator.

To demonstrate, take a look at **inner.d**:

```d
void printInts(int[] delegate() dg)
{
    import std.stdio;
    foreach(i; dg()) writeln(i);
} 

void main() {
    int[] ints;
    auto makeInts() {
        foreach(i; 0 .. 20) {
            ints ~= i;
        }
        return ints;
    }

    printInts(&makeInts);
}
```
Here, `makeInts` is an inner function. A pointer to a non-static inner function is not a function pointer, but a `delegate` (a context pointer/function pointer pair; if an inner function is `static`, a pointer of type `function` is produced instead). In this particular case, the delegate makes use of a variable in its parent scope. Here’s the output of compiling with `-vgc`:

```bash
dmd -vgc inner.d
inner.d(11): vgc: operator ~= may cause GC allocation
inner.d(7): vgc: using closure causes GC allocation
```
What we’re seeing here is that memory needs to be allocated so that the delegate can carry the state of `ints`, making it a _closure_ (which is not itself a type – the type is still `delegate`). Move the declaration of `ints` inside the scope of `makeInts` and recompile. You’ll find that the closure allocation goes away. A better option is to change the declaration of `printInts` to look like this:

```d
void printInts(scope int[] delegate() dg)
```
Adding `scope` to any function parameter ensures that any references in the parameter cannot be escaped. In other words, it now becomes impossible to do something like assign `dg` to a global variable, or return it from the function. The effect is that there is no longer a need to create a closure, so there will be no allocation. See the documentation for more on [function pointers, delegates and closures](https://dlang.org/spec/function.html#closures), and [function parameter storage classes](https://dlang.org/spec/function.html#parameters).


#### The gist


Given that the D GC is very different from those in languages like Java and C#, it’s certain to have different performance characteristics. Moreover, D programs tend to produce far less garbage than those written in a language like Java, where almost everything is a reference type. It helps to understand this when embarking on a D project for the first time. The strategies an experienced Java programmer uses to mitigate the impact of collections aren't likley to apply here.

While there is certainly a class of software in which no GC pauses are ever acceptable, that is an arguably small set. Most D projects can, and should, start out with the simple mitigation strategies from point #2 at the top of this article, then adapt the code to use `@nogc` or `core.memory.GC` as and when performance dictates. The command-line options demonstrated here can help ferret out the areas where that may be necessary.

As time goes by, it’s going to become easier to micromanage garbage collection in D programs. There’s a concerted effort underway to make Phobos, D’s standard library, as `@nogc`-friendly as possible. Language improvements such as Walter’s proposal to modify how exceptions are allocated should speed that work considerably.

Future posts in [this series](http://dlang.org/blog/the-gc-series/) will look at how to allocate memory outside of the GC heap and use it alongside GC allocations in the same program, how to compensate for disabled language features in `@nogc` code, strategies for handling the interaction of the GC with object destructors, and more.

_Thanks to Vladimir Panteleev, Guillaume Piolat, and Steven Schveighoffer for their valuable feedback on drafts of this article._

_The article has been amended to remove a misleading line about Java and C#, and to add some information about multiple threads._
