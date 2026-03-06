---
author: DBlogAdmin
comments: false
date: 2017-07-07 12:50:15+00:00
layout: post
link: https://dlang.org/blog/2017/07/07/go-your-own-way-part-one-the-stack/
slug: go-your-own-way-part-one-the-stack
title: 'Go Your Own Way (Part One: The Stack)'
wordpress_id: 949
categories:
- Code
- GC
permalink: /go-your-own-way-part-one-the-stack/
redirect_from: /2017/07/07/go-your-own-way-part-one-the-stack/
---

This is my third post in [the GC series](http://dlang.org/blog/the-gc-series/). In [the first post](https://dlang.org/blog/2017/03/20/dont-fear-the-reaper/), I introduced D's garbage collector and the language features that require it, and touched on simple strategies to use it effectively. In [the second post](https://dlang.org/blog/2017/06/16/life-in-the-fast-lane/), I showed off the tools provided by the language and library to disable or prohibit the GC in specific parts of a code base, how to use the compiler to assist in that endeavor, and recommended that D programs be written initially to embrace the GC, taking advantage of simple strategies to mitigate its impact, and later tuned to avoid it or further optimize its usage only when profiling shows it's warranted.

When garbage collection is turned off via `GC.disable` or prevented by the `@nogc` function annotation, memory will still need to be allocated from somewhere. And even when the GC is fully embraced, it's still desirable to minimize the size and quantity of GC heap allocations. That means allocating either via the stack, or via the non-GC heap. The focus of this post is the former. Non-GC heap allocations will be covered in my next post in this series.


### Stack allocation


The simplest allocation strategy in D is the same as it is in C: avoid the heap and use the stack whenever possible. When a local array is needed and the size can be known at compile time, use a static rather than a dynamic array. Structs, which are value types and stack-allocated by default, should be preferred where possible over classes, which are reference types and are usually allocated from one heap or another. D's compile-time features can present opportunities here that might not otherwise be available.


#### Static arrays


Static array declarations in D require the length to be known at compile-time.

    // OK
    int[10] nums;
    
    // Error: variable x cannot be read at compile time
    int x = 10;
    int[x] err;
Unlike dynamic arrays, static arrays can be initialized with array literals with no allocation taking place on the GC heap. The lengths must match, otherwise the compiler will emit an error.

```d
@nogc void main() {
    int[3] nums = [1, 2, 3];
}
```
Static arrays are automatically sliced when passed to any function taking a slice as a parameter, making them interchangeable with dynamic arrays.

```d
void printNums(int[] nums) {
    import std.stdio : writeln;
    writeln(nums);
}

void main() {
    int[]  dnums = [0, 1, 2];
    int[3] snums = [0, 1, 2];
    printNums(dnums);
    printNums(snums);
}
```
When compiling with `-vgc` to find the potential GC allocations in a program and eliminate them where possible, this is an easy win. Just be wary of situations like the following:

```d
int[] foo() {
    auto nums = [0, 1, 2];

    // Do work with nums...

    return nums;
}
```
Converting `nums` in this example to a static array would be a mistake. The return statement in that case would be returning a slice to stack-allocated memory, which is a programming error. Luckily, doing so will generate a compiler error.

On the other hand, if the return is conditional, it may be desirable to heap-allocate the array only when absolutely necessary rather than every time the function is called. In that scenario, a static array can be declared locally and a dynamic copy made on return. Enter the `.dup` property:

```d
int[] foo() {
    int[3] nums = [0, 1, 2];
    
    // Let x = the result of some work with nums
    bool condtion = x;

    if(condition) return nums.dup;
    else return [];
}
```
This function still uses the GC via `.dup`, but only allocates if it needs to and avoids allocation when it doesn't. Note that `[]` is equivalent to `null` in this case, a slice (or dynamic array) with a `.length` of `0` and a `.ptr` of `null`.


#### Structs vs. classes


Struct instances in D are allocated on the stack by default, but can be allocated on the heap when desired. Stack-allocated structs have deterministic destruction, with their destructors called as soon as the enclosing scope exits.

```d
struct Foo {
    int x;
    ~this() {
        import std.stdio;
        writefln("#%s says bye!", x);
    }
}
void main() {
    Foo f1 = Foo(1);
    Foo f2 = Foo(2);
    Foo f3 = Foo(3);
}
```
As expected, this prints:

    #3 says bye!
    #2 says bye!
    #1 says bye!
Classes, being reference types, are almost always allocated on the heap. Usually, that's the GC heap via `new`, though it could also be the non-GC heap through a custom allocator. But there's no rule saying they can't be allocated on the stack. The standard library template [`std.typecons.scoped` ](https://dlang.org/phobos/std_typecons.html#.scoped)allows us to easily do so.

```d
class Foo {
    int x;

    this(int x) { 
        this.x = x; 
    }
    
    ~this() {
        import std.stdio;
        writefln("#%s says bye!", x);
    }
}
void main() {
    import std.typecons : scoped;
    auto f1 = scoped!Foo(1);
    auto f2 = scoped!Foo(2);
    auto f3 = scoped!Foo(3);
}
```
Functionally, this is identical to the `struct` example above; it prints the same results. Deterministic destruction is achieved via the `core.object.destroy` function, which allows destructors to be called outside of GC collections.

Note that neither `scoped` nor `destroy` are currently usable in `@nogc` functions. This isn't necessarily a problem, as a function doesn't have to be annotated such to avoid the GC, but it can be a headache if you are trying to fit everything into a `@nogc` call tree. In future posts, we'll look at some of the design issues that crop up when using `@nogc` and how to avoid them.

Generally, when implementing custom types in D, the choice between `struct` and `class` should be dependent on the type's intended usage. POD types are obvious candidates for `struct`, whereas for types in something like a GUI system, where inheritance heirarchies and runtime interfaces are extremely useful, `class` is a more appropriate choice. Beyond those obvious cases, there are a number of other considerations which could be the focus of a separate blog post on the topic. For our purposes, just keep in mind that whether or not a type is implemented as a `struct` or a `class` need not always dictate whether or not instances can be allocated on the stack.


#### alloca


Given that D makes `alloca` available, it is also an option for stack allocation. This is a candidate especially for arrays when you want to avoid or eliminate a local GC allocation, but the array size is only known at run time. The following example allocates a dynamic array with a runtime size on the stack.

```d
import core.stdc.stdlib : alloca;

void main() {
    size_t size = 10;
    void* mem = alloca(size);

    // Slice the memory block
    int[] arr = cast(int[])mem[0 .. size];
}
```
The same caution about using `alloca` in C applies here: be careful not to blow up the stack. And as with local static arrays, don't return a slice of `arr`. Return `arr.dup` instead.


#### A simple example


Consider an implementation of a `Queue` data type. An idiomatic implementation in D is going to be a struct that's templated on the type it's intended to contain. In Java, collection usage is interface heavy and it's recommended to declare an instance using an interface type rather than the implementation type. Structs in D can't implement interfaces, but in many cases they can still be used to program to interfaces thanks to [Design by Introspection](https://youtu.be/es6U7WAlKpQ) (DbI). This allows programming to a common interface that is verified via compile-time introspection without the need for an interface type, so it can work with structs, classes and, thanks to [Universal Function Call Syntax](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394) (UFCS), even free functions (when the functions are in scope).

D's arrays are an obvious choice as the backing store for a `Queue` implementation. Moreover, there's an opportunity to make the backing store a static array when a queue is intended to be bounded with a fixed size. Since it's already a templated type, an additional parameter, a [template value parameter](http://dlang.org/spec/template.html#TemplateValueParameter) with a default value can easily be added to decide at compile time if the array should be static or not and, if so, how much space it should require.

```d
// A default Size of 0 means to use a dynamic array for the
// backing store; non-zero indicates a static array.
struct Queue(T, size_t Size = 0) 
{
    // This constant will be inferred as a boolean. By making it
    // public, a DbI template outside of this module can determine
    // whether or not the Queue might grow. 
    enum isFixedSize = Size > 0;

    void enqueue(T item) 
    {
        static if(isFixedSize) {
            assert(_itemCount < _items.length);
        }
        else {
            ensureCapacity();
        }
        push(item);
    }

    T dequeue() {
        assert(_itemCount != 0);
        static if(isFixedSize) {
            return pop();
        }
        else {
            auto ret = pop();
            ensurePacked();
            return ret;
        }
    }

    // Only available on a growable array
    static if(!isFixedSize) {
        void reserve(size_t capacity) { /* Allocate space for new items */ }
    }

private:   
    static if(isFixedSize) {
        T[Size] _items;     
    }
    else T[] _items;
    size_t _head, _tail;
    size_t _itemCount;

    void push(T item) { 
        /* Add item, update _head and _tail */
        static if(isFixedSize) { ... }
        else { ... }
    }

    T pop() { 
        /* Remove item, update _head and _tail */ 
        static if(isFixedSize) { ... }
        else { ... }
    }

    // These are only available on a growable array
    static if(!isFixedSize) {
        void ensureCapacity() { /* Alloc memory if needed */ }
        void ensurePacked() { /* Shrink the array if needed */}
    }
}
```
With this, the client can declare instances like so:

    Queue!Foo qUnbounded;
    Queue!(Foo, 128) qBounded;
`qBounded` requires no heap allocations. What happens with `qUnbounded` depends on the implementation. Moreover, compile-time introspection can be used to test if an instance is a fixed size or not. The `isFixedSize` constant is a convenience for that. Clients could alternatively use the built-in `__traits(hasMember, T, "reserve")` or the standard library function `std.traits.hasMember!T("reserve")` in one compile-time construct or another ([__traits](https://dlang.org/spec/traits.html) and [`std.traits`](https://dlang.org/phobos/std_traits.html) are great for DbI, and the latter should be preferred when it provides similar functionality), but including the constant in the type is more convenient.

```d
void doSomethingWithQueueInterface(T)(T queue)
{
    static if(T.isFixedSize) { ... }
    else { ... }
}
```
#### Conclusion


This has been a brief overview of a few options for stack allocation in D to avoid allocations from the GC heap. Making use of them when possible is an easy way to minimize the size and quantity of GC allocations, a proactive strategy for mitigating potential negative performance impacts from garbage collection.

The next post in [this series](http://dlang.org/blog/the-gc-series/) will cover some of the options available for non-GC heap allocations.
