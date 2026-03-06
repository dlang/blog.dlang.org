---
author: DBlogAdmin
comments: false
date: 2021-03-04 13:17:55+00:00
excerpt: In this continuation of the GC series, we explore what destruction means
  in the context of D's support for both garbage collection and manually-managed memory.
layout: post
link: https://dlang.org/blog/2021/03/04/symphony-of-destruction-structs-classes-and-the-gc-part-one/
slug: symphony-of-destruction-structs-classes-and-the-gc-part-one
title: 'Symphony of Destruction: Structs, Classes and the GC (Part One)'
wordpress_id: 2789
categories:
- Code
- GC
- Tutorials
permalink: /symphony-of-destruction-structs-classes-and-the-gc-part-one/
redirect_from: /2021/03/04/symphony-of-destruction-structs-classes-and-the-gc-part-one/
---

![Digital Mars D logo]({{ '/assets/images/symphony-of-destruction-structs-classes-and-the-gc-part-one/d6.png' | relative_url }})

This post is part of [a broader series](https://dlang.org/blog/the-gc-series/) on garbage collection in D. The motivation is to explore how destructors and the GC interact. To do that, we first need a bit of background. We do not go into a broader discussion on the ins and outs of object destruction, only what is most relevant to the interaction of destructors and the GC.

I've split the discussion into two blog posts. Here in Part One, we look at how deterministic and non-deterministic destruction differ, consider the consequences of having a single destructor for both scenarios, and finally establish two simple guidelines that will help us avoid those consequences. In Part Two, we'll go further and explore how we can still write solid destructors when circumstances dictate that the guidelines don't apply.



## Deterministic destruction



Destruction is deterministic when it is predictable, meaning the programmer can, simply by following the flow of the code, point out where and when an object's destructor is invoked. This is possible with struct instances allocated on the stack, as the compiler will insert calls to their destructors at well-defined points for automatic and deterministic destruction.

There are two basic rules for automatic destruction:





  1. The destructors of all stack-allocated structs in a given scope are invoked when the scope exits.


  2. Destructors are invoked in reverse lexical order (i.e., the opposite of the order in which the declarations appear in the source).



With these two rules in mind, we can examine the following example and accurately predict its output.


```d
import std.stdio;
struct Predictable
{
    int number;
    this(int n)
    {
        writeln("Constructor #", n);
        number = n;
    }
    ~this()
    {
        writeln("Destructor #", number);
    }
}

void main()
{
    Predictable s0 = Predictable(0);
    {
        Predictable s1 = Predictable(1);
    }
    Predictable s2 = Predictable(2);
}
```
We see that both `s0` and `s2` are directly within the scope of the `main` function, so their destructors will run when `main` exits. Given that the declaration of `s2` comes after that of `s0`, the destructor of `s2` will run before that of `s0`.

We also see that `s1` is declared in an anonymous inner scope between the declarations of `s0` and `s2`. This scope exits before `s2` is constructed, so the destructor of `s1` will execute before the constructor of `s2`.

With that, we can expect the following output:


```d
Constructor #0      // declaration of s0
Constructor #1      // declaration of s1
Destructor #1       // anonymous scope exits, s1 destroyed
Constructor #2      // declaration of s2
Destructor #2       // main exits, s2 then s0 destroyed
Destructor #0
```
Compiling and executing the example proves us accurate seers.

The programmer can implement deterministic destruction manually, as is necessary when destroying instances allocated on the non-GC heap, e.g., with `malloc` or `std.experimental.allocator`. In an earlier post, [Go Your Own Way (Part Two: The Heap)](https://dlang.org/blog/2017/09/25/go-your-own-way-part-two-the-heap/), I covered how to use `std.conv.emplace` to allocate instances on the non-GC heap and briefly mentioned that [destructors can be invoked manually via `destroy`](https://dlang.org/phobos/object.html#.destroy). That's a function template declared in the automatically imported `object` module so that it's always available. We won't retread the allocation discussion, but an example of manual destruction isn't out of bounds for this post.

In the following example, we'll reuse the definition of the `Predictable` struct and a `destroyPredictable` function to manually invoke the destructors. For completeness, I've included functions for allocating and deallocating `Predictable` instances from the non-GC heap: `allocatePredictable` and `deallocatePredictable`. If it isn't clear to you what these two functions are doing, please read [the blog post I mentioned above](https://dlang.org/blog/2017/09/25/go-your-own-way-part-two-the-heap/).


```d
void main()
{
    Predictable* s0 = allocatePredictable(0);
    scope(exit) { destroyPredictable(s0); }
    {
        Predictable* s1 = allocatePredictable(1);
        scope(exit) { destroyPredictable(s1); }
    }
    Predictable* s2 = allocatePredictable(2);
    scope(exit) { destroyPredictable(s2); }
}

void destroyPredictable(Predictable* p)
{
    if(p) {
        destroy(*p);
        deallocatePredictable(p);
    }
}

Predictable* allocatePredictable(int n)
{
    import core.stdc.stdlib : malloc;
    import std.conv : emplace;
    auto p = cast(Predictable*)malloc(Predictable.sizeof);
    return emplace!Predictable(p, n);
}

void deallocatePredictable(Predictable* p)
{
    import core.stdc.stdlib : free;
    free(p);
}
```
Running this program will result in precisely the same output as the previous example. In the `destroyPredictable` function, we dereference the struct pointer when calling `destroy` because there is no overload that takes a pointer. There are specializations for classes, interfaces, and structs passed by reference and a general catch-all that takes all other types by reference. Destructors are invoked on types that have them. Before exiting, the function sets the argument to its default `.init` value through the reference.

Note that if we were to give `destroy` a pointer without first dereferencing it, the code would still compile. The pointer would be accepted by reference and simply set to `null`, the default `.init` value for pointers, but the struct's destructor would not be invoked (i.e., the pointer is "destroyed", not the struct instance).

Inserting `writeln(*p)` immediately after `destroy(*p)` should print


    Predictable(0)

for each destroyed instance. (The default `.init` state for a struct in D is the aggregate of the `.init` property of each of its members; in this case, the sole member, being of type `int`, has an `.init` property of `0`, so the struct's default `.init` state is `Predictable(0)`. This [can be changed in the struct definition](https://dlang.org/spec/struct.html#default_struct_init), e.g., `struct Predictable { int id = 1; }`.)

`destroy` is not restricted to instances allocated on the non-GC heap. Any aggregate type instance (`struct`, `class`, or `interface`) is a valid argument no matter where it was allocated.



## Non-deterministic destruction



In languages with support for objects and a garbage collector, the responsibility for destroying object instances allocated on the GC heap falls to the GC. This is known as _finalization_. Before reclaiming an object's memory, the GC _finalizes_ the object by invoking its _finalizer_.

Finalization, though convenient, comes with a price. In Java's particular circumstances, the price was determined to be so high that its maintainers deprecated the `Object.finalize` method and [left a scary warning about its use in the documentation](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/lang/Object.html#finalize()). It's worth quoting here:




The finalization mechanism is inherently problematic. Finalization can lead to performance issues, deadlocks, and hangs. Errors in finalizers can lead to resource leaks; there is no way to cancel finalization if it is no longer necessary; and no ordering is specified among calls to finalize methods of different objects. Furthermore, there are no guarantees regarding the timing of finalization. The finalize method might be called on a finalizable object only after an indefinite delay, if at all.




Finalization in D isn't quite the bugbear it is in Java, but we do see [a less dramatic warning about it in the D documentation](https://dlang.org/spec/class.html#destructors):




The garbage collector is not guaranteed to run the destructor for all unreferenced objects. Furthermore, the order in which the garbage collector calls destructors for unreferenced objects is not specified.




Although there's no mention of "finalization" or "finalizers" here, that's precisely what the text is referring to. The core message is the same in both warnings: finalization is non-deterministic and cannot be relied on.

Unlike structs, classes in D are reference types by default. Some consequences: the programmer never has direct access to the underlying class instance; instances declared uninitialized are `null` by default; the normal use case is to allocate instances via `new`. When a class is instantiated in D, it is usually going to be managed by the GC and its destructor will serve as a finalizer.

As an experiment, let's change the definition of `struct Predictable` in our first example to `class Unpredictable` and use `new` to allocate the instances like so:


```d
import std.stdio;
class Unpredictable
{
    int number;
    this(int n)
    {
        writeln("Constructor #", n);
        number = n;
    }
    ~this()
    {
        writeln("Destructor #", number);
    }
}

void main()
{
    Unpredictable s0 = new Unpredictable(0);
    {
        Unpredictable s1 = new Unpredictable(1);
    }
    Unpredictable s2 = new Unpredictable(2);
}
```
We'll see that the output is drastically different:


    Constructor #0
    Constructor #1
    Constructor #2
    Destructor #0
    Destructor #1
    Destructor #2

Anyone familiar with the characteristics of the default DRuntime constructor can predict for this very simple program that all the destructors will be run when the GC's cleanup function is executed as the D runtime shuts down, and that they will be executed in the order in which they were declared (an implementation detail; and note that destruction at shut down [can be disabled via a command line argument](https://dlang.org/spec/garbage.html#gc_config)). But in a more complex program, this ability to predict breaks down. Destructors can be invoked by the GC at _almost_ any time and in any order.

To be clear, the GC will only perform its cleanup duties if and when it finds more memory is needed to fulfill a specific allocation request. In other words, it isn't constantly running in the background, marking objects unreachable and calling destructors willy nilly. To that extent, we can predict when the GC has the _possibility_ to perform its duties. Beyond that, all bets are off. We cannot predict with accuracy if any destructors will be invoked during any given allocation request or the order in which they will be invoked. This uncertainty has ramifications for how one implements destructors for any GC-managed type.

For starters, destructors of GC-managed objects should never perform any operation that can potentially result in a GC allocation request. Attempting to do so can result in an `InvalidMemoryOperationError` at run time. I use the word "potentially" because some operations can indirectly cause the error in certain circumstances, but not in others. Some examples: attempting to index an associative array can trigger an attempt to allocate a `RangeError` if the key is not present; a failed `assert` will result in allocation of an `AssertError`; calling any function not annotated with `@nogc` means GC operations are always possible in the call stack. These and any such operations should be avoided in the destructors of GC-managed objects. (The first seven items on [the list of operations disallowed in `@nogc` functions](https://dlang.org/spec/function.html#nogc-functions) are collectively a good guide.)

A larger issue is that one cannot rely on _any_ resource still being valid when a destructor is called by the GC. Consider a class that attempts to close a socket handle in its destructor; it's quite possible that the destructor won't be called until after the program has already shutdown the network interface. There is no scenario in which the runtime can catch this. In the best case, such circumstances will result in a silent failure, but they could also result in crashes during program shutdown or even sooner.

What it comes down to is that GC-allocated objects should never be used to manage any resource that depends upon deterministic destruction for cleanup.



## Designing for destruction



For the D neophyte, it can appear as if destructors in D are useless. Given that both struct and class instances can be allocated from memory that may or may not be managed by the GC, that destructors of GC-managed objects are not guaranteed to run, and that destructors are forbidden to perform GC operations during finalization, how can we ever rely on them?

In practice, it's not as bad as it may seem. Issues do arise for the unwary, but armed with a basic awareness of the nature of D destructors, it turns out that it's pretty easy to avoid having problems. This is especially true if the programmer adopts two fairly simple rules.



### 1. Pretend class destructors don't exist



Class instances will nearly always be allocated with `new`. That means their destructors will nearly always be non-deterministic. Much of what one would want to do in a destructor is somehow dependent on program state: either the destructor itself expects a certain state (like writing to a log file that is expected to be open), or the program expects the destructor to have modified the program state in a specific manner (like releasing a resource handle).

Non-deterministic destruction means that all expectations about program state are thrown out the window. That log file might have already been closed, so the message will never be written (I hope it wasn't important). That system resource handle may never be released until the program ends (I hope that particular resource isn't scarce). Even if it seems through testing that a class destructor is working the way it's intended, it's quite likely down to the fact that the testing has not uncovered the case where it breaks. In a long-running program, that case will inevitably pop up at some point. Have fun debugging when your production game server starts randomly crashing.

So when using classes in D, pretend they have no destructors. Pretend that they are Java classes with a deprecated `finalize` method.



### 2. Don't allocate structs on the GC heap when they have destructors



Since we're pretending classes have no destructors, then we're going to turn to structs for all of our destructive needs. Allocating structs as value objects on the stack will cover many use cases, but sometimes we may need to allocate them on the heap. When that situation arises, do not allocate any destructor-bearing struct with `new`. If we allocate a struct that has a destructor on the GC heap, it completely defeats our purpose of avoiding class destructors in the first place. That destructor we intended to be deterministic is now non-deterministic, so we may as well have just used a class.

As we have seen, struct instances can be allocated on the non-GC heap (e.g., with `malloc`) and their destructors manually invoked with the `destroy` function. If we **need** deterministic destruction **and** we absolutely **must have** a heap-allocated struct, then we **cannot** allocate that struct on the GC heap.



## Guidelines schmuidelines



I'm sure someone is reading the above guidelines and thinking, "If I have to pretend that classes have no destructors, then why do classes have destructors?" Well, you don't _have to_.

There is no One True Path to follow when deciding if an object should be implemented as a class or struct. Personally, I will always prefer structs over classes, and I will only reach for a class if I need something structs can't give me easily (like a hierarchy) or efficiently. Other people will consider if the object they need to represent has an identity, e.g., an `Actor` in a simulation versus the `Vertex` that defines its 3D coordinates. POD (Plain Old Data) types should always be structs, but beyond that it's largely a matter of preference.

My two guidelines are based on my experience and that of others with whom I've spoken. They are intended to help you keep the full implications of D's distinction between classes and structs at the forefront of your thoughts when architecting your program. They are not commandments that every D programmer must follow.

Realistically, most D programmers will encounter circumstances at one time or another in which the guidelines do not apply. For example, when mixing GC-managed memory and manually-managed memory in the same program, it's quite possible for a struct intended for stack use to wind up on the GC heap if the programmer is unaware of the circumstances. And some D programmers will always prefer classes over structs because that's just the way they want it, and so will simply choose to ignore the guidelines. That's no problem as long as they fully understand the consequences.

So what does that mean? How do you get over the non-deterministic nature of class destructors if your `Actor` class absolutely must have a destructor, or if you prefer to always follow The Way of the Class? How do you prevent structs intended for the stack from being GC-allocated? These are things we'll be looking at in Part Two. See you there.

_Thanks to Ali Çehreli and Max Haughton for their feedback. And to Adam D. Ruppe for his conversation on the topic in Discord and the title suggestion (it fits in more nicely with the series than the 'Appetite For Destruction' I had intended to go with)_
