---
author: AtilaNeves
comments: false
date: 2017-04-28 14:20:24+00:00
layout: post
link: https://dlang.org/blog/2017/04/28/automem-hands-free-raii-for-d/
slug: automem-hands-free-raii-for-d
title: 'automem: Hands-Free RAII for D'
wordpress_id: 743
categories:
- GC
- Guest Posts
permalink: /automem-hands-free-raii-for-d/
redirect_from: /2017/04/28/automem-hands-free-raii-for-d/
---

_Átila Neves has used both C++ and D professionally. He's responsible for several D libraries and tools, like [unit-threaded](https://github.com/atilaneves/unit-threaded), [cerealed](https://github.com/atilaneves/cerealed), and [reggae](https://github.com/atilaneves/reggae)._



* * *



![](http://dlang.org/blog/wp-content/uploads/2017/04/stamps-RAM-300px-300x73.png)Garbage collected languages tend to suffer from a framing problem, and D is no exception. Its inclusion of a mark-and-sweep garbage collector makes safe memory management easy and convenient, but, thanks to a widespread perception that GC in general is a performance killer, alienates a lot of potential users due to its mere existence.

Coming to D as I did from C++, the one thing I didn't like about the language initially was the GC. I've since come to realize that my fears were mostly unfounded, but the fact remains that, for many people, the GC is reason enough to avoid the language. Whether or not that is reasonable given their use cases is debatable (and something over which reasonable people may disagree), but the existence of the perception is not.

A lot of work has been done over the years to make sure that D code can be written which doesn't depend on the GC. The `@nogc` annotation is especially important here, and I don't think it's been publicized enough. A `@nogc main` function is a compile-time guarantee that the program will not ever allocate any GC memory. For the types of applications that need those sorts of guarantees, this is invaluable.

But if not allocating from the GC heap, where does one get the memory? Still in the experimental package of the standard library, [`std.experimental.allocator`](https://dlang.org/phobos/std_experimental_allocator.html) provides building blocks for composing allocators that should satisfy any and all memory allocation needs where the GC is deemed inappropriate. Better still, via the [`IAllocator`](https://dlang.org/phobos/std_experimental_allocator.html#.IAllocator) interface, one can even switch between GC and custom allocation strategies as needed at runtime.

I've recently used `std.experimental.allocator` in order to achieve `@nogc` guarantees and, while it works, there's one area in which the experience wasn't as smooth as when using C++ or Rust: disposing of memory. Like C++ and Rust, D has RAII. As is usual in all three, it's considered bad form to explicitly release resources. And yet, in the current state of affairs, while using the D standard library one has to manually dispose of memory if using `std.experimental.allocator`. D makes it easier than most languages that support exceptions, due to [`scope(exit)`](https://dlang.org/spec/statement.html#ScopeGuardStatement), but in a language with RAII that's just boilerplate. And as the good lazy programmer that I am, I abhor writing code that doesn't need to be, and shouldn't be, written. An itch developed.

The inspiration for the solution I came up with was C++; ever since C++11 I've been delighted with using `std::unique_ptr` and `std::shared_ptr` and basically never again worrying about manually managing memory. D's standard library has `Unique` and `RefCounted` in [`std.typecons`](https://dlang.org/phobos/std_typecons.html) but they predate `std.experimental.allocator` and so "bake in" the allocation strategy. Can we have our allocation cake and eat it too?

Enter [automem](https://github.com/atilaneves/automem), a library I wrote providing C++-style smart pointers that integrate with `std.experimental.allocator`. It was clear to me that the design had to be different from the smart pointers it took inspiration from. In C++, it's assumed that memory is allocated with `new` and freed with `delete` (although it's possible to override both). With custom allocators and no real obvious default choice, I made it so that the smart pointers would allocate memory themselves. This makes it so one can't allocate with one allocator and deallocate with a different one, which is another benefit.

Another goal was to preserve the possibility of `Unique`, like `std::unique_ptr`, to be a zero-cost abstraction. In that sense the allocator type must be specified (it defaults to `IAllocator`); if it's a value type with no state, then it takes up no space. In fact, if it's a singleton (determined at compile-time by probing where `Allocator.instance` exists), then it doesn't even need to be passed in to the constructor! As in much modern D code, [Design by Instropection](http://dconf.org/2015/talks/alexandrescu.html) pays its dues here. Example code:

```d
struct Point {
    int x;
    int y;
}

{
    // must pass arguments to initialise the contained object
    // but not an allocator instance since Mallocator is
    // a singleton (Mallocator.instance) returns the only
    // instantiation
    
    auto u1 = Unique!(Point, Mallocator)(2, 3);
    assert(*u1 == Point(2, 3));
    assert(u1.y == 3); // forwards to the contained object

    // auto u2 = u1; // won't compile, can only move
    typeof(u1) u2;
    move(u1, u2);
    assert(cast(bool)u1 == false); // u1 is now empty
}
// memory freed for the Point structure created in the block
```
`RefCounted` is automem's equivalent of C++'s `std::shared_ptr`. Unlike `std::shared_ptr` however, it doesn't always do an atomic reference count increment/decrement. The reason is that it leverage's D's type system to determine when it has to; if the payload is `shared`, then the reference count is changed atomically. If not, it can't be sent to other threads anyway and the performance penalty doesn't have to be paid. C++ always does an atomic increment/decrement. Rust gets around this with two types, `Arc` and `Rc`. In D the type system disambiguates. Another win for Design by Introspection, something that really is only possible in D. Example code:

```d
{
    auto s1 = RefCounted!(Point, Mallocator)(4, 5);
    assert(*s1 == Point(4, 5));
    assert(s1.x == 4);
    {
        auto s2 = s1; // can be copied, non-atomic reference count
    } // ref count goes to 1 here

} // ref count goes to 0 here, memory released
```
Given that the allocator type is usually specified, it means that when using a `@nogc` allocator (most of them), the code using automem can itself be made `@nogc`, with RAII taking care of any memory management duties. That means compile-time guarantees of no GC allocation for the applications that need them.

I hope automem and `std.experimental.allocator` manage to solve D's GC framing problem. Now it should be possible to write `@nogc` code with no manual memory disposal in D, just as it is in C++ and Rust.
