---
author: DBlogAdmin
comments: false
date: 2017-06-05 14:15:39+00:00
layout: post
link: https://dlang.org/blog/2017/06/05/compile-time-sort-in-d/
slug: compile-time-sort-in-d
title: Compile-Time Sort in D
wordpress_id: 841
categories:
- Code
permalink: /compile-time-sort-in-d/
redirect_from: /2017/06/05/compile-time-sort-in-d/
---

Björn Fahller recently wrote a blog post showing how to implement [a compile-time quicksort in C++17](http://playfulprogramming.blogspot.kr/2017/06/constexpr-quicksort-in-c17.html). It's a skillful demonstration that employs the evolving C++ feature set to write code that, while not quite concise, is more streamlined than previous iterations. He concludes with, "...the usefulness of this is very limited, but it is kind of cool, isn't it?"

There's quite a bit of usefulness to be found in evaluating code during compilation. The coolness (of which there is much) arises from the possibilities that come along with it. Starting from Björn's example, this post sets out to teach a few interesting aspects of compile-time evaluation in the D programming language.

The article came to my attention from Russel Winder's provocative query in the D forums, "Surely D can do better", which was quickly resolved with a "No Story"-style answer by Andrei Alexandrescu. "There is nothing to do really. Just use standard library sort," he quipped, and followed with code:

**Example 1**

```d
void main() {
    import std.algorithm, std.stdio;
    enum a = [ 3, 1, 2, 4, 0 ];
    static b = sort(a);
    writeln(b); // [0, 1, 2, 3, 4]
}
```
Though it probably won't be obvious to those unfamiliar with D, the call to `sort` really is happening at compile time. Let's see why.


### Compile-time code is runtime code


It's true. There are no hurdles to jump over to get things running at compile time in D. Any compile-time function is also a runtime function and can be executed in either context. However, not all runtime functions qualify for CTFE (Compile-Time Function Evaluation).

The fundamental requirements for CTFE eligibility are that a function must be portable, free of side effects, contain no inline assembly, and the source code must be available. Beyond that, the only thing deciding whether a function is evaluated during compilation vs. at run time is the context in which it's called.

[The CTFE Documentation](http://dlang.org/spec/function.html#interpretation) includes the following statement:


In order to be executed at compile time, the function must appear in a context where it must be so executed...


It then lists a few examples of where that is true. What it boils down to is this: if a function can be executed in a compile-time context where it must be, then it will be. When it can't be excecuted (it doesn't meet the CTFE requirements, for example), the compiler will emit an error.


### Breaking down the compile-time sort


Take a look once more at **Example 1**.

```d
void main() {
    import std.algorithm, std.stdio;
    enum a = [ 3, 1, 2, 4, 0 ];
    static b = sort(a);
    writeln(b);
}
```
The points of interest that enable the CTFE magic here are lines 3 and 4.

The `enum` in line 3 is a [manifest constant](http://dlang.org/spec/enum.html#manifest_constants). It differs from other constants in D (those marked `immutable` or `const`) in that it exists only at compile time. Any attempt to take its address is an error. If it's never used, then its value will never appear in the code.

When an `enum` is used, the compiler essentially pastes its value in place of the symbol name.

    enum xinit = 10;
    int x = xinit;
    
    immutable yinit = 11;
    int y = yinit;
Here, `x` is initialized to the literal `10`. It's identical to writing `int x = 10`. The constant `yinit` is initialized with an `int` literal, but `y` is initialized with the value of `yinit`, which, though known at compile time, is not a literal itself. `yinit` will exist at run time, but `xinit` will not.

In **Example 1**, the static variable `b` is initialized with the manifest constant `a`. In [the CTFE documentation](http://dlang.org/spec/function.html#interpretation), this is listed as an example scenario in which a function must be evaluated during compilation. A static variable declared in a function can only be initialized with a compile-time value. Trying to compile this:

**Example 2**

```d
void main() {
    int x = 10;
    static y = x;
}
```
Will result in this:

    Error: variable x cannot be read at compile time
Using a function call to initialize a static variable means the function must be executed at compile time and, therefore, it will be if it qualifies.

Those two pieces of the puzzle, the manifest constant and the static initializer, explain why the call to `sort` in **Example 1** happens at compile time without any metaprogramming contortions. In fact, the example could be made one line shorter:

**Example 3**

```d
void main() {
    import std.algorithm, std.stdio;
    static b = sort([ 3, 1, 2, 4, 0 ]);
    writeln(b);
}
```
And if there's no need for `b` to stick around at run time, it could be made an `enum` instead of a static variable:

**Example 4**

```d
void main() {
    import std.algorithm, std.stdio;
    enum b = sort([ 3, 1, 2, 4, 0 ]);
    writeln(b);
}
```
In both cases, the call to `sort` will happen at compile time, but they handle the result differently. Consider that, due to the nature of `enum`s, the change will produce an equivalent of this: `writeln([ 0, 1, 2, 3, 4 ])`. Because the call to `writeln` happens at run time, the array literal might trigger [a GC allocation](http://dlang.org/blog/2017/03/20/dont-fear-the-reaper/) (though it could be, and sometimes will be, optimized away). With the static initializer, there is no runtime allocation, as the result of the function call is used at compile time to initialize the variable.

It's worth noting that `sort` isn't directly returning a value of type `int[]`. Take a peek at [the documentation](https://dlang.org/library/std/algorithm/sorting/sort.html) and you'll discover that what it's giving back is a [`SortedRange`](https://dlang.org/library/std/range/sorted_range.html). Specifically in our usage, it's a `SortedRange!(int[], "a < b")`. This type, like arrays in D, exposes all of the primitives of a [random-access range](https://dlang.org/library/std/range/primitives/is_random_access_range.html), but additionally provides  functions that only work on sorted ranges and can take advantage of their ordering (e.g. [`trisect`](https://dlang.org/library/std/range/sorted_range.trisect.html)). The array is still in there, but wrapped in an enhanced API.



### To CTFE or not to CTFE



I mentioned above that all compile-time functions are also runtime functions. Sometimes, it's useful to distinguish between the two inside the function itself. D allows you to do that with the `__ctfe` variable. Here's an example from my book, '[Learning D](http://amzn.to/1IlQkZX)'.

**Example 5**

```d
string genDebugMsg(string msg) {
    if(__ctfe)
        return "CTFE_" ~ msg;
    else
        return "DBG_" ~ msg;
}

pragma(msg, genDebugMsg("Running at compile-time."));
void main() {
    writeln(genDebugMsg("Running at runtime."));
}
```
The [`msg` pragma](https://dlang.org/spec/pragma.html#msg) prints a message to `stderr` at compile time. When `genDebugMsg` is called as its second argument here, then inside that function the variable `__ctfe` will be `true`. When the function is then called as an argument to `writeln`, which happens in a runtime context, `__ctfe` is `false`.

It's important to note that `__ctfe` is **not** a compile-time value. No function knows if it's being executed at compile-time or at run time. In the former case, it's being evaluated by an interpreter that runs inside the compiler. Even then, we can make a distinction between compile-time and runtime values inside the function itself. The result of the function, however, will be a compile-time value when it's executed at compile time.



### Complex compile-time validation



Now let's look at something that doesn't use an out-of-the-box function from the standard library.

A few years back, Andrei published '[The D Programming Language](http://amzn.to/1ZTDmqH)'. In the section describing CTFE, he implemented three functions that could be used to validate the parameters passed to a hypothetical [linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator). The idea is that the parameters must meet a set of criteria, which he lays out in the book (buy it for the commentary -- it's well worth it), for generating the largest period possible. Here they are, minus the unit tests:

**Example 6**

```d
// Implementation of Euclid’s algorithm
ulong gcd(ulong a, ulong b) { 
    while (b) {
        auto t = b; b = a % b; a = t;
    }
    return a; 
}

ulong primeFactorsOnly(ulong n) {
    ulong accum = 1;
    ulong iter = 2;
    for (; n >= iter * iter; iter += 2 - (iter == 2)) {
        if (n % iter) continue;
        accum *= iter;
        do n /= iter; while (n % iter == 0);
    }
    return accum * n;
}

bool properLinearCongruentialParameters(ulong m, ulong a, ulong c) { 
    // Bounds checking
    if (m == 0 || a == 0 || a >= m || c == 0 || c >= m) return false; 
    // c and m are relatively prime
    if (gcd(c, m) != 1) return false;
    // a - 1 is divisible by all prime factors of m
    if ((a - 1) % primeFactorsOnly(m)) return false;
    // If a - 1 is multiple of 4, then m is a multiple of 4, too. 
    if ((a - 1) % 4 == 0 && m % 4) return false;
    // Passed all tests
    return true;
}
```
The key point this code was intended to make is the same one I made earlier in this post: `properLinearCongruentialParameters` is a function that can be used in both a compile-time context and a runtime context. There's no special syntax required to make it work, no need to create two distinct versions. 

Want to implement a linear congruential generator as a templated struct with the RNG parameters passed as template arguments? Use `properLinearCongruentialParameters` to validate the parameters. Want to implement a version that accepts the arguments at run time? `properLinearCongruentialParameters` has got you covered. Want to implement an RNG that can be used at both compile time and run time? You get the picture.

For completeness, here's an example of validating parameters in both contexts.

**Example 7**

```d
void main() {
    enum ulong m = 1UL << 32, a = 1664525, c = 1013904223;
    static ctVal = properLinearCongruentialParameters(m, a, c);
    writeln(properLinearCongruentialParameters(m, a, c));
}
```
If you've been paying attention, you'll know that `ctVal` must be initialized at compile time, so it forces CTFE on the call to the function. And the call to the same function as an argument to `writeln` happens at run time. You can have your cake and eat it, too.



### Conclusion



Compile-Time Function Evaluation in D is both convenient and painless. It can be combined with other features such as [templates](https://dlang.org/spec/template.html) (it's particularly useful with template parameters and constraints), [string mixins](http://dlang.org/spec/statement.html#MixinStatement) and [import expressions](http://dlang.org/spec/expression.html#ImportExpression) to simplify what might otherwise be extremely complex code, some of which wouldn't even be possible in many languages without a preprocessor. As a bonus, Stefan Koch is currently working on [a more performant CTFE engine](https://dlang.org/blog/2017/04/10/the-new-ctfe-engine/) for the D frontend to make it even more convenient. Stay tuned here for more news on that front.

_Thanks to the multiple members of the D community who reviewed this article._
