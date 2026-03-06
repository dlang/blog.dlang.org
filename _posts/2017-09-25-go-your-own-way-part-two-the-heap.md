---
author: DBlogAdmin
comments: false
date: 2017-09-25 14:11:07+00:00
excerpt: This post is part of an ongoing series on garbage collection in the D Programming
  Language, and the second of two regarding the allocation of memory outside of the
  GC. Part One discusses stack allocation. Here, we’ll look at allocating memory from
  the non-GC heap.
layout: post
link: https://dlang.org/blog/2017/09/25/go-your-own-way-part-two-the-heap/
slug: go-your-own-way-part-two-the-heap
title: 'Go Your Own Way (Part Two: The Heap)'
wordpress_id: 1104
categories:
- Code
- GC
- The Language
permalink: /go-your-own-way-part-two-the-heap/
redirect_from: /2017/09/25/go-your-own-way-part-two-the-heap/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)This post is part of [an ongoing series](https://dlang.org/blog/the-gc-series/) on garbage collection in the D Programming Language, and the second of two regarding the allocation of memory outside of the GC. [Part One](https://dlang.org/blog/2017/07/07/go-your-own-way-part-one-the-stack/) discusses stack allocation. Here, we’ll look at allocating memory from the non-GC heap.

Although this is only my fourth post in the series, it’s the third in which I talk about ways to _avoid_ the GC. Lest anyone jump to the wrong conclusion, that fact does not signify an intent to warn programmers away from the D garbage collector. Quite the contrary. Knowing how and when to avoid the GC is integral to understanding how to efficiently embrace it.

To hammer home a repeated point, efficient garbage collection requires reducing stress on the GC. As highlighted in [the first](https://dlang.org/blog/2017/03/20/dont-fear-the-reaper/) and subsequent posts in this series, that doesn’t necessarily mean avoiding it completely. It means being judicious in how often and how much GC memory is allocated. Fewer GC allocations means fewer opportunities for a collection to trigger. Less total memory allocated from the GC heap means less total memory to scan.

It’s impossible to make any accurate, generalized statement about what sort of applications may or may not feel an impact from the GC; such is highly application specific. What can be said is that it may not be necessary for many applications to temporarily avoid or disable the GC, but when it is, it’s important to know how. Allocating from the stack is an obvious approach, but D also allows allocating from the non-GC heap.



### The ubiquitous C



For better or worse, C is everywhere. Any software written today, no matter the source language, is probably interacting with a C API at some level. Despite the C specification defining no standard ABI, its platform-specific quirks and differences are understood well enough that most languages know how to interface with it. D is no exception. In fact, all D programs have access to the C standard library by default.

[The `core.stdc` package](https://github.com/dlang/druntime/tree/master/src/core/stdc), part of [DRuntime](https://github.com/dlang/druntime), is a collection of D modules translated from C standard library headers. When a D executable is linked, the C standard library is linked along with it. All that need be done to gain access is to import the appropriate modules.


```d
import core.stdc.stdio : puts;
void main() 
{
    puts("Hello C standard library.");
}
```
Some who are new to D may be laboring under a misunderstanding that functions which call into C require an `extern(C)` annotation, or, after Walter’s Bright’s recent ‘[D as a Better C](https://dlang.org/blog/2017/08/23/d-as-a-better-c/)’ article, must be compiled with `-betterC` on the command line. Neither is true. Normal D functions can call into C without any special effort beyond the presence of an `extern(C)` declaration of the function being called. In the snippet above, [the declaration of `puts`](https://github.com/dlang/druntime/blob/master/src/core/stdc/stdio.d#L1063) is in the `core.stdc.stdio` module, and that’s all we need to call it.



#### `malloc` and friends



Given that we have access to C’s standard library in D, we therefore have access to the functions `malloc`, `calloc`, `realloc` and `free`. All of these can be made available by importing `core.stdc.stdlib`. And thanks to D’s slicing magic, using these functions as the foundation of a non-GC memory management strategy is a breeze.


```d
import core.stdc.stdlib;
void main() 
{
    enum totalInts = 10;
    
    // Allocate memory for 10 ints
    int* intPtr = cast(int*)malloc(int.sizeof * totalInts);

    // assert(0) (and assert(false)) will always remain in the binary,
    // even when asserts are disabled, which makes it nice for handling
    // malloc failures    
    if(!intPtr) assert(0, "Out of memory!");

    // Free when the function exits. Not necessary for this example, but
    // a potentially useful strategy for temporary allocations in functions 
    // other than main.
    scope(exit) free(intPtr);

    // Slice the D pointer to get a more manageable length/pointer pair.
    int[] intArray = intPtr[0 .. totalInts];
}
```
Not only does this bypass the GC, it also bypasses D’s default initialization. A GC-allocated array of type `T` would have all of its elements initialized to `T.init`, which is `0` for `int`. If mimicking D’s default initialization is the desired behavior, more work needs to be done. In this example, we could replace `malloc` with `calloc` for the same effect, but that would only be correct for integrals. `float.init`, for example, is `float.nan` rather than `0.0f`. We’ll come back to this later in the article.

Of course, it would be more idiomatic to wrap both `malloc` and `free` and work with slices of memory. A minimal example:


```d
import core.stdc.stdlib;

// Allocate a block of untyped bytes that can be managed
// as a slice.
void[] allocate(size_t size)
{
    // malloc(0) is implementation defined (might return null 
    // or an address), but is almost certainly not what we want.
    assert(size != 0);

    void* ptr = malloc(size);
    if(!ptr) assert(0, "Out of memory!");
    
    // Return a slice of the pointer so that the address is coupled
    // with the size of the memory block.
    return ptr[0 .. size];
}

T[] allocArray(T)(size_t count) 
{ 
    // Make sure to account for the size of the
    // array element type!
    return cast(T[])allocate(T.sizeof * count); 
}

// Two versions of deallocate for convenience
void deallocate(void* ptr)
{   
    // free handles null pointers fine.
    free(ptr);
}

void deallocate(void[] mem) 
{ 
    deallocate(mem.ptr); 
}

void main() {
    import std.stdio : writeln;
    int[] ints = allocArray!int(10);
    scope(exit) deallocate(ints);
    
    foreach(i; 0 .. 10) {
        ints[i] = i;
    }

    foreach(i; ints[]) {
        writeln(i);
    }
}
```
`allocate` returns `void[]` rather than `void*` because it carries with it the number of allocated bytes in its `length` property. In this case, since we’re allocating an array, we could instead rewrite `allocArray` to slice the returned pointer immediately, but anyone calling `allocate` directly would still have to take into account the size of the memory. The disassociation between arrays and their length in C is [a major source of bugs](https://digitalmars.com/articles/b44.html), so the sooner we can associate them the better. Toss in some templates for `calloc` and `realloc` and you’ve got the foundation of a memory manager based on the C heap.

On a side note, the preceding three snippets (yes, even the one with the `allocArray` template) work with and without `-betterC`. But from here on out, we’ll restrict ourselves to features in normal D code.



#### Avoid leaking like a sieve



When working directly with slices of memory allocated outside of the GC heap, be careful about appending, concatenating, and resizing. By default, the append (`~=`) and concatenate (`~`) operators on built-in dynamic arrays and slices will allocate from the GC heap. Concatenation will always allocate a new memory block for the combined string. Normally, the append operator will allocate to expand the backing memory only when it needs to. As the following example demonstrates, it always needs to when it’s given a slice of non-GC memory.


```d
import core.stdc.stdlib : malloc;
import std.stdio : writeln;

void main()
{
    int[] ints = (cast(int*)malloc(int.sizeof * 10))[0 .. 10];
    writeln("Capacity: ", ints.capacity);

    // Save the array pointer for comparison
    int* ptr = ints.ptr;
    ints ~= 22;
    writeln(ptr == ints.ptr);
}
```
This should print the following:


    Capacity: 0
    false
A capacity of `0` on a slice indicates that the next append will trigger an allocation. Arrays allocated from the GC heap normally have space for extra elements beyond what was requested, meaning some appending can occur without triggering a new allocation. It’s more like a property of the memory backing the array rather than of the array itself. Memory allocated from the GC does some internal bookkeeping to keep track of how many elements the memory block can hold so that it knows at any given time if a new allocation is needed. Here, because the memory for `ints` was not allocated by the GC, none of that bookkeeping is being done by the runtime on the existing memory block, so it _must_ allocate on the next append (see Steven Schveighoffer’s ’[D Slices](https://dlang.org/d-array-article.html) article for more info).

This isn’t necessarily a bad thing when it’s the desired behavior, but anyone who’s not prepared for it can easily run into ballooning memory usage thanks to leaks from `malloc`ed memory never being deallocated. Consider these two functions:


```d
void leaker(ref int[] arr)
{
    ...
    arr ~= 10;
    ...
}

void cleaner(int[] arr)
{
    ...
    arr ~= 10;
    ...
}
```
Although arrays are reference types, meaning that modifying existing elements of an array argument inside a function will modify the elements in the original array, they are passed by value as function parameters. Any activity that modifies the structure of an array argument, i.e. its `length` and `ptr` properties, only affects the local variable inside the function. The original will remain unchanged unless the array is passed by reference.

So if an array backed by the C heap is passed to `leaker`, the append will cause a new array to be allocated from the GC heap. Worse, if `free` is subsequently called on the `ptr` property of the original array, which now points into the GC heap rather than the C heap, we’re in undefined behavior territory. `cleaner`, on the other hand, is fine. Any array passed into it will remain unchanged. Internally, the GC will allocate, but the `ptr` property of the original array still points to the original memory block.

As long as the original array isn’t overwritten or allowed to go out of scope, this is a non-issue. Functions like `cleaner` can do what they want with their local slice and things will be fine externally. Otherwise, if the original array is to be discarded, you can prevent all of this by tagging functions that you control with `@nogc`. Where that’s either not possible or not desirable, then either a copy of the pointer to the original `malloc`ed memory must be kept and `free`ed at some point after the reallocation takes place, custom appending and concatenation needs to be implemented, or the allocation strategy needs to be reevaluated.

Note that [`std.container.array`](https://dlang.org/phobos/std_container_array.html) contains an `Array` type that does not rely on the GC and may be preferable over managing all of this manually.



#### Other APIs



The C standard library isn’t the only game in town for heap allocations. A number of alternative `malloc` implementations exist and any of those can be used instead. This requires manually compiling the source and linking with the resultant objects, but that’s not an onerous task. Heap memory can also be allocated through system APIs, like [the Win32 HeapAlloc](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366711(v=vs.85).aspx) function on Windows (available by importing [`core.sys.windows.windows`](https://github.com/dlang/druntime/blob/master/src/core/sys/windows/windows.d)). As long as there’s a way to get a pointer to a block of heap memory, it can be sliced and manipulated in a D program in place of a block of GC memory.



### Aggregate types



If we only had to worry about allocating arrays in D, then we could jump straight on to the next section. However, we also need to concern ourselves with `struct` and `class` types. For this discussion, however, we will only focus on the former. The next couple of posts in the series will focus exclusively on classes.

Allocating an array of `struct` types, or a single instance of one, is often no different than when the type is `int`.


```d
struct Point { int x, y; }
Point* onePoint = cast(Point*)malloc(Point.sizeof);
Point* tenPoints = cast(Point*)malloc(Point.sizeof * 10);
```
Where things break down is when contructors enter the mix. `malloc` and friends know nothing about constructing D object instances. Thankfully, Phobos provides us with a function template that does.

[`std.conv.emplace`](https://dlang.org/phobos/std_conv.html#emplace) can take either a pointer to typed memory or an untyped `void[]`, along with an optional number of arguments, and return a pointer to a single, fully initialized and constructed instance of that type. This example shows how to do so using both `malloc` and the `allocate` function template from above:


```d
struct Vertex4f 
{ 
    float x, y, z, w; 
    this(float x, float y, float z, float w = 1.0f)
    {
        this.x = x;
        this.y = y;
        this.z = z;
        this.w = w;
    }
}

void main()
{
    import core.stdc.stdlib : malloc;
    import std.conv : emplace;
    import std.stdio : writeln;
    
    Vertex4f* temp1 = cast(Vertex4f*)malloc(Vertex4f.sizeof);
    Vertex4f* vert1 = emplace(temp1, 4.0f, 3.0f, 2.0f); 
    writeln(*vert1);

    void[] temp2 = allocate(Vertex4f.sizeof);
    Vertex4f* vert2 = emplace!Vertex4f(temp2, 10.0f, 9.0f, 8.0f);
    writeln(*vert2);
}
```
Another feature of `emplace` is that it also handles default initialization. Consider that `struct` types in D need not implement constructors. Here’s what happens when we change the implementation of `Vertex4f` to remove the constructor:


```d
struct Vertex4f 
{
    // x, y, z are default inited to float.nan
    float x, y, z;

    // w is default inited to 1.0f
    float w = 1.0f;
}

void main()
{
    import core.stdc.stdlib : malloc;
    import std.conv : emplace;
    import std.stdio : writeln;

    Vertex4f vert1, vert2 = Vertex4f(4.0f, 3.0f, 2.0f);
    writeln(vert1);
    writeln(vert2);    
    
    auto vert3 = emplace!Vertex4f(allocate(Vertex4f.sizeof));
    auto vert4 = emplace!Vertex4f(allocate(Vertex4f.sizeof), 4.0f, 3.0f, 2.0f);
    writeln(*vert3);
    writeln(*vert4);
}
```
This prints the following:


    Vertex4f(nan, nan, nan, 1)
    Vertex4f(4, 3, 2, 1)
    Vertex4f(nan, nan, nan, 1)
    Vertex4f(4, 3, 2, 1)
So `emplace` allows heap-allocated struct instances to be initialized in the same manner as stack allocated struct instances, with or without a constructor. It also works with the built-in types like `int` and `float`. Just always remember that `emplace` is intended to initialize and construct a _single instance_, not an array of instances.

If the aggregate type has a destructor, it should be invoked before its memory is deallocated. This can be achieved [with the `destroy` function](https://dlang.org/phobos/object.html#.destroy) (always available through the implicit import of `std.object`).



### std.experimental.allocator



The entirety of the text above describes the fundamental building blocks of a custom memory manager. For many use cases, it may be sufficient to forego cobbling something together by hand and instead take advantage of the D standard library’s [`std.experimental.allocator`](https://dlang.org/phobos/std_experimental_allocator.html) package. This is a high-level API that makes use of low-level techniques like those described above, along with [Design by Introspection](https://www.youtube.com/watch?v=es6U7WAlKpQ), to facilitate the assembly of different types of allocators that know how to allocate, initialize, and construct arrays and type instances. Allocators like [`Mallocator`](https://dlang.org/phobos/std_experimental_allocator_mallocator.html) and [`GCAllocator`](https://dlang.org/phobos/std_experimental_allocator_gc_allocator.html) can be used to grab chunks of memory directly, or combined with other [building blocks](https://dlang.org/phobos/std_experimental_allocator_building_blocks.html) for specialized behavior. See the [emsi-containers library](https://github.com/economicmodeling/containers) for a real-world example.



### Keeping the GC informed



Given that it’s rarely recommended to disable the GC entirely, most D programs allocating outside the GC heap will likely also be using memory from the GC heap in the same program. In order for the GC to properly do its job, it needs to be informed of any non-GC memory that contains, or may potentially contain, references to memory from the GC heap. For example, a linked list whose nodes are allocated with `malloc` might contain references to classes allocated with `new`.

The GC can be given the news via `GC.addRange`.


```d
import core.memory;
enum size = int.sizeof * 10;
void* p1 = malloc(size);
GC.addRange(p1, size);

void[] p2 = allocate!int(10);
GC.addRange(p2.ptr, p2.length);
```
When the memory block is no longer needed, the corresponding `GC.removeRange` can be called to prevent it from being scanned. This **does not deallocate** the memory block. That will need to be done manually via `free` or whatever allocator interface was used to allocate it. Be sure to [read the documentation](https://dlang.org/phobos/core_memory.html#addRange) before using either function.

Given that one of the goals of allocating from outside the GC heap is to reduce the amount of memory the GC must scan, this may seem self-defeating. That’s the wrong way to look at it. If non-GC memory is going to hold references to GC memory, then it’s vital to let the GC know about it. Not doing so can cause the GC to free up memory to which a reference still exists. `addRange` is a tool specifically designed for that situation. If it can be guaranteed that no GC-memory references live inside a non-GC memory block, such as a `malloc`ed array of vertices, then `addRange` need not be called on that memory block.



#### A word of warning



Be careful when passing typed pointers to `addRange`. Because the function was implemented with the C like approach of taking a pointer to a block of memory and the number of bytes it contains, there is an opportunity for error.


```d
struct Item { SomeClass foo; }
auto items = (cast(Item*)malloc(Item.sizeof * 10))[0 .. 10];
GC.addRange(items.ptr, items.length);
```
With this, the GC would be scanning a block of memory exactly ten bytes in size. The `length` property returns the number of elements the slice refers to. Only when the type is `void` (or the element type is one-byte long, like `byte` and `ubyte`) does it equate to the size of the memory block the slice refers to. The correct thing to do here is:


    GC.addRange(items.ptr, items.length * Item.sizeof);
However, until DRuntime is updated with an alternative, it may be best to implement a wrapper that takes a `void[]` parameter.


```d
void addRange(void[] mem) 
{
    import core.memory;
    GC.addRange(mem.ptr, mem.length);
}
```
Then calling `addRange(items)` will do the correct thing. The implicit conversion of the slice to `void[]` in the function call will mean that `mem.length` is the same as `items.length * Item.sizeof`.



### The GC series marches on



This post has covered the very basics of using the non-GC heap in D programs. One glaring omission, in addition to `class` types, is what to do about destructors. I’m saving that topic for the post about classes, where it is highly relevant. That’s the next scheduled post in the GC series. Stay tuned!

Thanks to Walter Bright, Guillaume Piolat, Adam D. Ruppe, and Steven Schveighoffer for their valuable feedback on a draft of this article.
