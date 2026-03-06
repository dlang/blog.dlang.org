---
author: WalterBright
comments: false
date: 2017-08-23 13:01:52+00:00
excerpt: 'There are large and immensely useful programs written in C, such as the
  Linux operating system and a very large chunk of the programs written for it. While
  D programs can interface with C libraries, the reverse isn''t true. C programs cannot
  interface with D ones. It''s not possible (at least not without considerable effort)
  to compile a couple of D files and link them in to a C program. The trouble is that
  compiled D files refer to things that only exist in the D runtime library, and linking
  that in (it''s a bit large) tends to be impractical.


  That is, until Better C came along.'
layout: post
link: https://dlang.org/blog/2017/08/23/d-as-a-better-c/
slug: d-as-a-better-c
title: D as a Better C
wordpress_id: 1044
categories:
- BetterC
- Code
- Core Team
- The Language
permalink: /d-as-a-better-c/
redirect_from: /2017/08/23/d-as-a-better-c/
---

_[Walter Bright](http://walterbright.com/) is the BDFL of the D Programming Language and founder of [Digital Mars](http://digitalmars.com/). He has decades of experience implementing compilers and interpreters for multiple languages, including Zortech C++, the first native C++ compiler. He also created [Empire, the Wargame of the Century](http://www.classicempire.com/). This post is [the first in a series](https://dlang.org/blog/the-d-and-c-series/#betterC) about [D’s BetterC mode](https://dlang.org/blog/2017/08/23/d-as-a-better-c/)_



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)D was designed from the ground up to interface directly and easily to C, and to a lesser extent C++. This provides access to endless C libraries, the Standard C runtime library, and of course the operating system APIs, which are usually C APIs.

But there's much more to C than that. There are large and immensely useful programs written in C, such as the Linux operating system and a very large chunk of the programs written for it. While D programs can interface with C libraries, the reverse isn't true. C programs cannot interface with D ones. It's not possible (at least not without considerable effort) to compile a couple of D files and link them in to a C program. The trouble is that compiled D files refer to things that only exist in the D runtime library, and linking that in (it's a bit large) tends to be impractical.

D code also can't exist in a program unless D controls the `main()` function, which is how the startup code in the D runtime library is managed. Hence D libraries remain inaccessible to C programs, and chimera programs (a mix of C and D) are not practical. One cannot pragmatically "try out" D by add D modules to an existing C program.

That is, until Better C came along.

It's been done before, it's an old idea. Bjarne Stroustrup wrote a paper in 1988 entitled "[A Better C](http://www.drdobbs.com/open-source/a-better-c/223000087)". His early C++ compiler was able to compile C code pretty much unchanged, and then one could start using C++ features here and there as they made sense, all without disturbing the existing investment in C. This was a brilliant strategy, and drove the early success of C++.

A more modern example is Kotlin, which uses a different method. Kotlin syntax is not compatible with Java, but it is fully interoperable with Java, relies on the existing Java libraries, and allows a gradual migration of Java code to [Kotlin](https://en.wikipedia.org/wiki/Kotlin_(programming_language)). Kotlin is indeed a "Better Java", and this shows in its success.


### D as Better C


D takes a radically different approach to making a better C. It is not an extension of C, it is not a superset of C, and does not bring along C's longstanding issues (such as the preprocessor, array overflows, etc.). D's solution is to subset the D language, removing or altering features that require the D startup code and runtime library. This is, simply, the charter of the `-betterC` compiler switch.

Doesn't removing things from D make it no longer D? That's a hard question to answer, and it's really a matter of individual preference. The vast bulk of the core language remains. Certainly the D characteristics that are analogous to C remain. The result is a language somewhere in between C and D, but that is fully upward compatible with D.


### Removed Things


Most obviously, the garbage collector is removed, along with the features that depend on the garbage collector. Memory can still be allocated the same way as in C - using `malloc()` or some custom allocator.

Although C++ classes and COM classes will still work, D polymorphic classes will not, as they rely on the garbage collector.

Exceptions, typeid, static construction/destruction, RAII, and unittests are removed. But it is possible we can find ways to add them back in.

Asserts are altered to call the C runtime library assert fail functions rather than the D runtime library ones.

(This isn't a complete list, for that see [http://dlang.org/dmd-windows.html#switch-betterC](http://dlang.org/dmd-windows.html#switch-betterC).)


### Retained Things


More importantly, what remains?

What may be initially most important to C programmers is memory safety in the form of array overflow checking, no more stray pointers into expired stack frames, and guaranteed initialization of locals. This is followed by what is expected in a modern language -- modules, function overloading, constructors, member functions, Unicode, nested functions, dynamic closures, Compile Time Function Execution, automated documentation generation, highly advanced metaprogramming, and Design by Introspection.


### Footprint


Consider a C program:

```c
#include <stdio.h>

int main(int argc, char** argv) {
    printf("hello world\n");
    return 0;
}
```
It compiles to:

    _main:
    push EAX
    mov [ESP],offset FLAT:_DATA
    call near ptr _printf
    xor EAX,EAX
    pop ECX
    ret
The executable size is 23,068 bytes.

Translate it to D:

```d
import core.stdc.stdio;

extern (C) int main(int argc, char** argv) {
    printf("hello world\n");
    return 0;
}
```
The executable size is the same, 23,068 bytes. This is unsurprising because the C compiler and D compiler generate the same code, as they share the same code generator. (The equivalent full D program would clock in at 194Kb.) In other words, nothing extra is paid for using D rather than C for the same code.

The `Hello World` program is a little too trivial. Let's step up in complexity to the infamous sieve benchmark program:

```c
#include <stdio.h>

/* Eratosthenes Sieve prime number calculation. */

#define true    1
#define false   0
#define size    8190
#define sizepl  8191

char flags[sizepl];

int main() {
    int i, prime, k, count, iter;

    printf ("10 iterations\n");
    for (iter = 1; iter <= 10; iter++) {
        count = 0;
        for (i = 0; i <= size; i++)
            flags[i] = true;
        for (i = 0; i <= size; i++) {
            if (flags[i]) {
                prime = i + i + 3;
                k = i + prime;
                while (k <= size) {
                    flags[k] = false;
                    k += prime;
                }
                count += 1;
            }
        }
    }
    printf ("\n%d primes", count);
    return 0;
}
```
Rewriting it in Better C:

```d
import core.stdc.stdio;

extern (C):

__gshared bool[8191] flags;

int main() {
    int count;

    printf("10 iterations\n");
    foreach (iter; 1 .. 11) {
        count = 0;
        flags[] = true;
        foreach (i; 0 .. flags.length) {
            if (flags[i]) {
                const prime = i + i + 3;
                auto k = i + prime;
                while (k < flags.length) {
                    flags[k] = false;
                    k += prime;
                }
                count += 1;
            }
        }
    }
    printf("%d primes\n", count);
    return 0;
}
```
It looks much the same, but some things are worthy of note:



 	
  * `extern (C):` means use the C calling convention.

 	
  * D normally puts static data into thread local storage. C sticks them in global storage. `__gshared` accomplishes that.

 	
  * `foreach` is a simpler way of doing for loops over known endpoints.

 	
  * `flags[] = true;` sets all the elements in `flags` to `true` in one go.

 	
  * Using `const` tells the reader that `prime` never changes once it is initialized.

 	
  * The types of `iter`, `i`, `prime` and `k` are inferred, preventing inadvertent type coercion errors.

 	
  * The number of elements in `flags` is given by `flags.length`, not some independent variable.


And the last item leads to a very important hidden advantage: accesses to the `flags` array are bounds checked. No more overflow errors! We didn't have to do anything
in particular to get that, either.

This is only the beginning of how D as Better C can improve the expressivity, readability, and safety of your existing C programs. For example, D has nested functions, which in my experience work very well at prying goto's from my cold, dead fingers.

On a more personal note, ever since `-betterC` started working, I've been converting many of my old C programs still in use into D, one function at a time. Doing it one function at a time, and running the test suite after each change, keeps the program in a correctly working state at all times. If the program doesn't work, I only have one function to look at to see where it went wrong. I don't particularly care to maintain C programs anymore, and with `-betterC` there's no longer any reason to.

The Better C ability of D is available in the 2.076.0 beta: [download it](http://dlang.org/download.html#dmd_beta) and [read the changelog](http://dlang.org/changelog/2.076.0.html).
