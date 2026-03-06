---
author: DBlogAdmin
comments: false
date: 2019-04-08 10:16:24+00:00
layout: post
link: https://dlang.org/blog/2019/04/08/project-highlight-dpp/
slug: project-highlight-dpp
title: 'Project Highlight: DPP'
wordpress_id: 2029
categories:
- Code
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-dpp/
redirect_from: /2019/04/08/project-highlight-dpp/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)D was designed from the beginning to be ABI compatible with C. Translate the declarations from a C header file into a D module and you can link directly with the corresponding C library or object files. The same is true in the other direction as long as the functions in the D code are annotated with [the appropriate linkage attribute](https://dlang.org/spec/attribute.html#linkage). These days, it’s possible to bind with C++ and even Objective-C.

[Binding with C](https://dlang.org/blog/the-d-and-c-series/) is easy, but can sometimes be a bit tedious, particularly when done by hand. I can speak to this personally as I originally implemented [the Derelict collection of bindings](https://github.com/DerelictOrg) by hand and, though I slapped together some automation when I ported it all over to [its successor project, BindBC](https://github.com/BindBC), everything there is maintained by hand. [Tools like dstep exist](https://github.com/jacob-carlborg/dstep) and can work well enough, though they come with limitations which require careful attention to and massaging of the output.

Tediousness is an enemy of productivity. That’s why several pages of discussion were generated from [Átila Neves’s casual announcement](https://forum.dlang.org/post/ywgookituhxbrzfxtfvl@forum.dlang.org) a few weeks before DConf 2018 that it was now possible to `#include` C headers in D code.

dpp[ is a compiler wrapper](https://github.com/atilaneves/dpp) that will parse a D source file with the `.dpp` extension and expand in place any `#include` directives it encounters, translating all of the C or C++ symbols to D, and then pass the result to a D compiler (DMD by default). Says Átila:


What motivated the project was a day at Cisco when I wanted to use D but ended up choosing C++ for the task at hand. Why? Because with C++ I could include the relevant header and be on my way, whereas with D (or any other language really) I’d have to somehow translate the header and all its transitive dependencies somehow. I tried dstep and it failed miserably. Then there’s the fact that the preprocessor is nearly always needed to properly use a C API. I wanted to remove one advantage C++ has over D, so I wrote dpp.


Here’s the example he presented [in the blog post](https://atilaoncode.blog/2018/04/09/include-c-headers-in-d-code/) accompanying the initial announcement:

```d
// stdlib.dpp
#include <stdio.h>
#include <stdlib.h>

void main() {
    printf("Hello world\n".ptr);

    enum numInts = 4;
    auto ints = cast(int*) malloc(int.sizeof * numInts);
    scope(exit) free(ints);

    foreach(int i; 0 .. numInts) {
        ints[i] = i;
        printf("ints[%d]: %d ".ptr, i, ints[i]);
    }

    printf("\n".ptr);
}
```
Three months later, dpp was [successfully compiling the `julia.h` header](https://forum.dlang.org/post/fplmjoiggnxyuvuxpafa@forum.dlang.org) allowing the Julia language to be embedded in a D program. The following month, it was [enabled by default on run.dlang.io](https://run.dlang.io/is/egqYGY).

C support is fairly solid, though not perfect.


Although preprocessor macro support is one of dpp’s key features, some macros just can’t be translated because they expand to C/C++ code fragments. I can’t parse them because they’re not actual code yet (they only make sense in context), and I can’t guess what the macro parameters are. Strings? Ints? Quoted strings? How does one programmatically determine that `#define FOO(S) (S)` is meant to be a C cast? Did you know that in C macros can have the same name as functions and it all works? Me neither until I got a bug report!


Push the `stdlib.dpp` code block from above [through run.dlang.io](https://run.dlang.io/is/LvzAbr) and read the output to see an example of translation difficulties.

The C++ story is more challenging. [ Átila recently wrote about](https://atilaoncode.blog/2019/03/07/the-joys-of-translating-cs-stdfunction-to-d/) one of the problems he faced. That one he managed to solve, but others remain.


dpp can’t translate C++ template specialisations on reference types because reference types don’t exist in D. I don’t know how to translate anything that depends on SFINAE because it also doesn’t exist in D.


For those not in the know, classes in D are reference types in the same way that Java classes are reference types, and function parameters annotated with `ref` accept arguments by reference, but when it comes to variable declarations, D has no equivalent for the C++ lvalue reference declarator, e.g. `int& someRef = i;`.

Despite the difficulties, Átila persists.


The holy grail is to be able to #include a C++ standard library header, but that’s so difficult that I’m currently concentrating on a much easier problem first: being able to successfully translate C++ headers from a much simpler library that happens to use standard library types (`std::string`, `std::vector`, `std::map`, the usual suspects). The idea there is to treat all types that dpp can’t currently handle as opaque binary blobs, focusing instead on the production library types and member functions. This _sounds_ simple, but in practice I’ve run into issues with LLVM IR with ldc, ABI issues with dmd, mangling issues, and the most fun of all: how to create instances of C++ stdlib types in D to pass back into C++? If a function takes a reference to `std::string`, how do I give it one? I did find a hacky way to pass D slices to C++ functions though, so that was cool!


On the plus side, he’s found some of D’s features particularly helpful in implementing dpp, though he did say that “this is harder for me to recall since at this point I mostly take D’s advantages for granted.” The first thing that came to mind was a combination of built-in unit tests and token strings:

```d
unittest {
    shouldCompile(
        C(q{ struct Foo { int i; } }),
        D(q{ static assert(is(typeof(Foo.i) == int)); })
    );
}
```
It’s almost self-explanatory: the first parameter to `shouldCompile` is C code (a header), and the second D code to be compiled after translating the C header. D’s token strings allow the editor to highlight the code inside, and the fact that C syntax is so similar to D lets me use them on C code as well!


He also found help from D’s contracts and the garbage collector.


libclang is a C library and as such has hardly any abstractions or invariant enforcement. All nodes in the AST are represented by a libclang “cursor”, which can have several “kinds”. D’s contracts allowed me to document and enforce at runtime which kind(s) of cursors a function expects, preventing bugs. Also, libclang in certain places requires the client code to manually manage memory. D’s GC makes for a wrapper API in which that is never a concern.


During development, he exposed some bugs in the DMD frontend.


I tried using `sumtype` in a separate branch of dpp to first convert libclang AST entities into types that are actually enforced at compile time instead of run time. Unfortunately that caused me to have to switch to compiling all code at once since `sumtype` behaves differently in separate compilation, triggering previously unseen frontend bugs.


For unit testing, he uses [unit-threaded, a library he created](https://github.com/atilaneves/unit-threaded) to augment D’s built-in unit testing feature with advanced functionality. To achieve this, the library makes use of D’s compile-time reflection features. But dpp has _a lot_ of unit tests.


Given the number of tests I wrote for dpp, compiling takes a very long time. This is exacerbated by `-unittest`, which is a known issue. Not using unit-threaded’s runner would speed up compilation, but then I’d lose all the features. It’d be better if the compile-time reflection required were made faster.


Perhaps he’ll see some joy there when [Stefan Koch’s ongoing work on NewCTFE](https://dlang.org/blog/2017/04/10/the-new-ctfe-engine/) is completed.

Átila will be speaking about dpp on May 9 as part of [his presentation at DConf 2019 in London](https://dconf.org/2019/talks/neves.html). The conference runs from May 8–11, so as I write there’s still [plenty of time to register](https://dconf.org/2019/registration.html). For those who can’t make it, you can watch the livestream (a link to which will be provided in the D forums each day of the conference) or see the videos of all the talks on [the D Language Foundation’s YouTube channel](https://www.youtube.com/channel/UC5DNdmeE-_lS6VhCVydkVvQ) after DConf is complete.
