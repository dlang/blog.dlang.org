---
author: DBlogAdmin
comments: false
date: 2018-10-17 15:09:40+00:00
excerpt: When interacting with C APIs, it’s almost a given that arrays are going to
  pop up in one way or another (perhaps most often as strings, a subject of a future
  article in the “D and C” series). Although D arrays are implemented in a manner
  that is not directly compatible with C, the fundamental building blocks are the
  same. This makes compatibility between the two relatively painless as long as the
  differences are not forgotten. This article is the first of a few exploring those
  differences.
layout: post
link: https://dlang.org/blog/2018/10/17/interfacing-d-with-c-arrays-part-1/
slug: interfacing-d-with-c-arrays-part-1
title: 'Interfacing D with C: Arrays Part 1'
wordpress_id: 1723
categories:
- Code
- D and C
- Tutorials
permalink: /interfacing-d-with-c-arrays-part-1/
redirect_from: /2018/10/17/interfacing-d-with-c-arrays-part-1/
---

This post is [part of an ongoing series](https://dlang.org/blog/the-d-and-c-series/) on working with both D and C in the same project. [The previous post](https://dlang.org/blog/2017/12/05/interfacing-d-with-c-getting-started/) showed how to compile and link C and D objects. This post is the first in a miniseries focused on arrays.

When interacting with C APIs, it’s almost a given that arrays are going to pop up in one way or another (perhaps most often as strings, a subject of a future article in the “D and C” series). Although D arrays are implemented in a manner that is not directly compatible with C, the fundamental building blocks are the same. This makes compatibility between the two relatively painless as long as the differences are not forgotten. This article is the first of a few exploring those differences.

When using a C API from D, it’s sometimes necessary to translate existing code from C to D. A new D program can benefit from existing examples of using the C API, and anyone porting a program from C that uses the API would do well to keep the initial port as close to the original as possible. It’s on that basis that we’re starting off with a look at the declaration and initialization syntax in both languages and how to translate between them. Subsequent posts in this series will cover multidimensional arrays, the anatomy of a D array, passing D arrays to and receiving C arrays from C functions, and how the GC fits into the picture.

My original concept of covering this topic was much smaller in scope, my intent to brush over the boring details and assume that readers would know enough of the basics of C to derive the why from the what and the how. That was before I gave a D tutorial presentation to a group among whom only one person had any experience with C. I’ve also become more aware that there [are regular users of the D forums](https://forum.dlang.org/) who have never touched a line of C. As such, I’ll be covering a lot more ground than I otherwise would have (hence a two-part article has morphed into at least three). I urge those for whom much of said ground is old hat not to get complacent in their skimming of the page! A comfortable experience with C is more apt than none at all to obscure some of the pitfalls I describe.



### Array declarations



Let’s start with a simple declaration of a one-dimensional array:


    int c0[3];
This declaration allocates enough memory on the stack to hold three `int` values. The values are stored contiguously in memory, one right after the other. `c0` may or may not be initialized, depending on where it’s declared. Global variables and `static` local variables are default initialized to `0`, as the following C program demonstrates.

**definit.c**


```c
#include <stdio.h>

// global (can also be declared static)
int c1[3];

void main(int argc, char** argv)
{
    static int c2[3];       // static local
    int c3[3];              // non-static local

    printf("one: %i %i %i\n", c1[0], c1[1], c1[2]);
    printf("two: %i %i %i\n", c2[0], c2[1], c2[2]);
    printf("three: %i %i %i\n", c3[0], c3[1], c3[2]);
}
```
For me, this prints:


    one: 0 0 0
    two: 0 0 0
    three: -1 8 0
The values for `c3` just happened to be lying around at that memory location. Now for the equivalent D declaration:


    int[3] d0;
_[Try it online](https://run.dlang.io/is/moXqNt)_

Here we can already find the first gotcha.

A general rule of thumb in D is that C code pasted into a D source file should either work as it does in C or fail to compile. For a long while, C array declaration syntax fell into the former category and was a legal alternative to the D syntax. It has since been deprecated and subsequently removed from the language, meaning `int d0[3]` will now cause the compiler to scold you:


    Error: instead of C-style syntax, use D-style int[3] d0
It may seem an arbitrary restriction, but it really isn’t. At its core, it’s about consistency at a couple of different levels.

One is that [we read declarations in D from right to left](https://dlang.org/spec/declaration.html). In the declaration of `d0`, everything flows from right to left in the same order that we say it: “(d0) is an (array of three) (integers)”. The same is not true of the C-style declaration.

Another is that the type of `d0` is actually `int[3]`. Consider the following pointer declarations:


    int* p0, p1;
The type of both `p0` and `p1` is `int*` (in C, only `p0` would be a pointer; `p1` would simply be an `int`). It’s the same as all type declarations in D—type on the left, symbol on the right. Now consider this:


    int d1[3], d2[3];
    int[3] d4, d5;
Having two different syntaxes for array declarations, with one that splits the type like an infinitive, sets the stage for the production of inconsistent and potentially confusing code. By making the C-style syntax illegal, consistency is enforced. Code readability is a key component of maintainability.

Another difference between `d0` and `c0` is that the elements of `d0` will be default initialized no matter where or how it’s declared. Module scope, local scope, static local… it doesn’t matter. Unless the compiler is told otherwise, variables in D are always default initialized to the predefined value specified by [the `init` property of each type](https://dlang.org/spec/property.html#init). Array elements are initialized to the `init` property of the element type. As it happens, `int.init == 0`. Translate **definit.c** to D and see it for yourself (open up [run.dlang.io and give it a go](https://run.dlang.io/)).

When translating C to D, this default initialization business is a subtle gotcha. Consider this innocently contrived C snippet:


    // static variables are default initialized to 0 in C
    static float vertex[3];
    some_func_that_expects_inited_vert(vertex);
A direct translation straight to D will not produce the expected result, as `float.init == float.nan`, not `0.0f`!

When translating between the two languages, always be aware of which C variables are not explicitly initialized, which are expected to be initialized, and [the default initialization value for each of the basic types](https://dlang.org/spec/type.html#basic-data-types) in D. Failure to account for the subtleties may well lead to debugging sessions of the hair-pulling variety.

Default initialization can easily be disabled in D with `= void` in the declaration. This is particularly useful for arrays that are going to be loaded with values before they’re read, or that contain elements with an `init` value that isn’t very useful as anything other than a marker of uninitialized variables.


    float[16] matrix = void;
    setIdentity(matrix);
On a side note, the purpose of default initialization is not to provide a convenient default value, but to make uninitialized variables stand out (a fact you may come to appreciate in a future debugging session). A common mistake is to assume that types like `float` and `char`, with their “not a number” (`float.nan`) and invalid UTF–8 (`0xFF`) initializers, are the oddball outliers. Not so. Those values are great markers of uninitialized memory because they aren’t useful for much else. It’s the integer types (and `bool`) that break the pattern. For these types, the entire range of values has potential meaning, so there’s no single value that universally shouts “Hey! I’m uninitialized!”. As such, integer and `bool` variables are often left with their default initializer since `0` and `false` are frequently the values one would pick for explicit initialization for those types. Floating point and character values, however, should generally be explicitly initialized or assigned to as soon as possible.



### Explicit array initialization



C allows arrays to be explicitly initialized in different ways:


```d
int ci0[3] = {0, 1, 2};  // [0, 1, 2]
int ci1[3] = {1};        // [1, 0, 0]
int ci2[]  = {0, 1, 2};  // [0, 1, 2]
int ci3[3] = {[2] = 2, [0] = 1}; // [1, 0, 2]
int ci4[]  = {[2] = 2, [0] = 1}; // [1, 0, 2]
```
What we can see here is:




  * elements are initialized sequentially with the constant values in the initializer list

  * if there are fewer values in the list than array elements, then all remaining elements are initialized to `0` (as seen in `ci1`)

  * if the array length is omitted from the declaration, the array takes the length of the initializer list (`ci2`)

  * designated initializers, as in `ci3`, allow specific elements to be initialized with `[index] = value` pairs, and indexes not in the list are initialized to `0`

  * when the length is omitted from the declaration and a designated initializer is used, the array length is based on the highest index in the initializer and elements at all unlisted indexes are initialized to `0`, as seen in `ci4`



Initializers aren’t supposed to be longer than the array (`gcc` gives a warning and initializes a three-element array to the first three initializers in the list, ignoring the rest).

Note that it’s possible to mix the designated and non-designated syntaxes in a single initializer:


```d
// [0, 1, 0, 5, 0, 0, 0, 8, 44]
int ci5[] = {0, 1, [3] = 5, [7] = 8, 44};
```
Each value without a designation is applied in sequential order as normal. If there is a designated initializer immediately preceding it, then it becomes the value for the next index, and all other elements are initialized to `0`. Here, `0` and `1` go to indexes `ci5[0]` and `ci5[1]` as normal, since they are the first two values in the list. Next comes a designator for `ci5[3]`, so `ci5[2]` has no corresponding value in this list and is initialized to `0`. Next comes the designator for `ci5[7]`.  We have skipped `ci5[4]`, `ci5[5]`, and `ci5[6]`,  so they are all initialized to `0`. Finally, `44` lacks a designator, but immediately follows `[7]`, so it becomes the value for the element at `ci5[8]`. In the end, `ci5` is initialized to a length of `9` elements.

Also note that designated array initializers were added to C in C99. Some C compiler versions either don't support the syntax or require a special command line flag to enable it. As such, it's probably not something you'll encounter very much in the wild, but still useful to know about when you do.

Translating all of these to D opens the door to more gotchas. Thankfully, the first one is a compiler error and won’t cause any heisenbugs down the road:


```d
int[3] wrong = {0, 1, 2};
int[3] right = [0, 1, 2];
```
Array initializers in D are array literals. The same syntax can be used to pass anonymous arrays to functions, as in `writeln([0, 1, 2])`. For the curious, the declaration of `wrong` produces the following compiler error:


    Error: a struct is not a valid initializer for a int[3]
The `{}` syntax [is used for `struct` initialization](https://dlang.org/spec/struct.html#static_struct_init) in D (not to be confused with struct literals, which can also [be used to initialize a `struct` instance](https://dlang.org/spec/struct.html#struct-literal)).

The next surprise comes in the translation of `ci1`.


```d
// int ci1[3] = {1};
int[3] di1 = [1];
```
This actually produces a compiler error:


    Error: mismatched array lengths, 3 and 1
What gives? First, take a look at the translation of `ci2`:


```d
// int ci2[] = {0, 1, 2};
int[] di2 = [0, 1, 2];
```
In the C code, there is no difference between `ci1` and `ci2`. They both are fixed-length, three-element arrays allocated on the stack. In D, this is one case where that general rule of thumb about pasting C code into D source modules breaks down.

Unlike C, D [actually makes a distinction between arrays](https://dlang.org/spec/arrays.html#static-arrays) of types `int[3]` and `int[]`. The former is, like C, a fixed-length array, commonly referred to in D as a static array. The latter, unlike C, is a dynamic-length array, commonly referred to as a dynamic array or a slice. Its length can grow and shrink as needed.

Initializers for static arrays must have the same length as the array. D simply does not allow initializers shorter than the declared array length. Dynamic arrays take the length of their initializers. `di2` is initialized with three elements, but more can be appended. Moreover, the initializer is not required for a dynamic array. In C, `int foo[];` is illegal, as the length can only be omitted from the declaration when an initializer is present.


    // gcc says "error: array size missing in 'illegalC'"
    // int illegalC[]
    int[] legalD;
    legalD ~= 10;
`legalD` is an empty array, with no memory allocated for its elements. Elements can be added via the append operator, `~=`.

Memory for dynamic arrays is allocated at the point of declaration only when an explicit initializer is provided, as with `di2`. If no initializer is present, memory is allocated when the first element is appended. By default, dynamic array memory is allocated from the GC heap (though the compiler may determine that it’s safe to allocate on the stack as an optimization) and space for more elements than needed is initialized in order to reduce the need for future allocations ([the `reserve` function can be used](https://dlang.org/phobos/object.html#.reserve) to allocate a large block in one go, without initializing any elements). Appended elements go into the preallocated slots until none remain, then the next append triggers a new allocation. [Steven Schveighoffer's excellent array article](https://dlang.org/articles/d-array-article.html) goes into the details, and also describes array features we'll touch on in the next part.

Often, when translating a declaration like `ci2` to D, the difference between the fixed-length, stack-allocated C array and the dynamic-length, GC-allocated D array isn’t going to matter one iota. One case where it does matter is when the D array is declared inside a function marked `@nogc`:


```d
@nogc void main()
{
    int[] di2 = [0, 1, 2];
}
```
_[Try it online](https://run.dlang.io/is/4AO9vT)_

The compiler ain’t letting you get away with that:


    Error: array literal in @nogc function D main may cause a GC allocation
The same error isn’t triggered when the array is static, since it’s allocated on the stack and the literal elements are just shoved right in there. New C programmers coming to D for the first time tend to reach for `@nogc` almost as if it goes against their very nature not to, so this is something they will bump into until they eventually come to the realization that [the GC is not the enemy of the people](https://dlang.org/blog/the-gc-series/).

To wrap this up, that big paragraph on designated array initializers in C is about to pull double duty. D also supports designated array initializers, just with a different syntax.


```d
// [0, 1, 0, 5, 0, 0, 0, 8, 44]
// int ci5[] = {0, 1, [3] = 5, [7] = 8, 44};
int[] di5 = [0, 1, 3:5, 7:8, 44];
int[9] di6 = [0, 1, 3:5, 7:8, 44];
```
_[Try it online](https://run.dlang.io/is/4kAt6u)_

It works with both static and dynamic arrays, following the same rules and producing the same initialization values as in C.

The main takeaways from this section are:




  * there is a distinction in D between static and dynamic arrays, in C there is not

  * static arrays are allocated on the stack

  * dynamic arrays are allocated on the GC heap

  * uninitialized static arrays are default initialized to the `init` property of the array elements

  * dynamic arrays can be explicitly initialized and take the length of the initializer

  * dynamic arrays cannot be explicitly initialized in `@nogc` scopes

  * uninitialized dynamic arrays are empty





### This is the time on the D Blog when we dance



There are a lot more words in the preceding sections than I had originally intended to write about array declarations and initialization, and I still have quite a bit more to say about arrays. In the next post, we’ll look at the anatomy of a D array and dig into the art of passing D arrays across the language divide.
