---
author: WalterBright
comments: false
date: 2018-06-11 13:51:16+00:00
excerpt: D as BetterC (a.k.a. DasBetterC) is a way to upgrade existing C projects
  to D in an incremental manner. This article shows a step-by-step process of converting
  a non-trivial C project to D and deals with common issues that crop up.
layout: post
link: https://dlang.org/blog/2018/06/11/dasbetterc-converting-make-c-to-d/
slug: dasbetterc-converting-make-c-to-d
title: 'DasBetterC: Converting make.c to D'
wordpress_id: 1579
categories:
- BetterC
- Code
- Core Team
permalink: /dasbetterc-converting-make-c-to-d/
redirect_from: /2018/06/11/dasbetterc-converting-make-c-to-d/
---

_[Walter Bright](http://walterbright.com/) is the BDFL of the D Programming Language and founder of [Digital Mars](http://digitalmars.com/). He has decades of experience implementing compilers and interpreters for multiple languages, including Zortech C++, the first native C++ compiler. He also created [Empire, the Wargame of the Century](http://www.classicempire.com/). This post is [the third in a series](https://dlang.org/blog/the-d-and-c-series/#betterC) about [D’s BetterC mode](https://dlang.org/blog/2017/08/23/d-as-a-better-c/)_



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)D as BetterC (a.k.a. DasBetterC) is a way to upgrade existing C projects to D in an incremental manner. This article shows a step-by-step process of converting a non-trivial C project to D and deals with common issues that crop up.

While [the dmd D compiler front end](https://github.com/dlang/dmd) has already been converted to D, it’s such a large project that it can be hard to see just what was involved. I needed to find a smaller, more modest project that can be easily understood in its entirety, yet is not a contrived example.

The old make program I wrote for the [Datalight C compiler](https://en.wikipedia.org/wiki/Datalight) in the early 1980’s came to mind. It’s a real implementation of the classic make program that’s been in constant use since the early 80’s. It’s written in pre-Standard C, has been ported from system to system, and is a remarkably compact 1961 lines of code, including comments. It is still in regular use today.

Here’s the [make manual](https://digitalmars.com/ctg/make.html), and the [source code](https://github.com/DigitalMars/Compiler/commit/473bf5bef99a1748d5154b343b986534271cd841#diff-14c44b49b9f46b1aeab33a6684214e55). The executable size for make.exe is 49,692 bytes and the last modification date was Aug 19, 2012.

The Evil Plan is:



 	
  1. Minimize diffs between the C and D versions. This is so that if the programs behave differently, it is far easier to figure out the source of the difference.

 	
  2. No attempt will be made to fix or improve the C code during translation. This is also in the service of (1).

 	
  3. No attempt will be made to refactor the code. Again, see (1).

 	
  4. Duplicate the behavior of the C program as exactly and as much as possible,
bugs and all.

 	
  5. Do whatever is necessary as needed in the service of (4).


Once that is completed, only then is it time to fix, refactor, clean up, etc.


## Spoiler Alert!


The [completed conversion](https://github.com/DigitalMars/Compiler/blob/1dbd3e3381c8f7f7ab2e35214dcd63455ae38c29/dm/src/make/dmake.d). The resulting executable is 52,252 bytes (quite comparable to the original 49,692). I haven’t analyzed the increment in size, but it is likely due to instantiations of the `NEWOBJ` template (a macro in the C version), and changes in the DMC runtime library since 2012.


## Step By Step


Here are the differences between the [C and D versions](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34). It’s 664 out of 1961 lines, about a third, which looks like a lot, but I hope to convince you that nearly all of it is trivial.

The `#include files are replaced` by corresponding D imports, such as [replacing `#include <stdio.h>` with `import core.stdc.stdio;`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R11). Unfortunately, some of the `#include` files are specific to Digital Mars C, and D versions do not exist (I need to fix that). To not let that stop the project, I simply included the [relevant declarations in lines 29 to 64](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R29). (See the documentation [for the `import` declaration](https://dlang.org/spec/module.html#import-declaration).)

[`#if _WIN32` is replaced](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L23) with [`version (Windows)`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R20). (See the documentation [for the version condition](https://dlang.org/spec/version.html#version) and [predefined versions](https://dlang.org/spec/version.html#predefined-versions).)

[`extern (C):`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R27) marks the remainder of the declarations in the file as compatible with C. (See the documentation [for the linkage attribute](https://dlang.org/spec/attribute.html#linkage).)

A global search/replace changes uses of the [debug1, debug2 and debug3](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L27) macros to [debug printf](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R499). In general, [`#ifdef DEBUG` preprocessor directives](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff–14c44b49b9f46b1aeab33a6684214e55L290) are replaced with [`debug` conditional compilation](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R276). (See the documentation [for the `debug` statement](https://dlang.org/spec/version.html#DebugStatement).)

```c
/* Delete these old C macro definitions...
#ifdef DEBUG
-#define debug1(a)       printf(a)
-#define debug2(a,b)     printf(a,b)
-#define debug3(a,b,c)   printf(a,b,c)
-#else
-#define debug1(a)
-#define debug2(a,b)
-#define debug3(a,b,c)
-#endif
*/

// And replace their usage with the debug statement
// debug2("Returning x%lx\n",datetime);
debug printf("Returning x%lx\n",datetime);
```
The [`TRUE`, `FALSE`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L37) and `NULL` macros are search/replaced with `true`, `false`, and `null`.

The [`ESC macro is replaced`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L40) by a [manifest constant](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R66). (See the documentation [for manifest constants](https://dlang.org/spec/enum.html#manifest_constants).)

```c
// #define ESC     '!'
enum ESC =      '!';
```
The [`NEWOBJ macro is replaced`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L42) with a [template function](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R68).

```c
// #define NEWOBJ(type)    ((type *) mem_calloc(sizeof(type)))
type* NEWOBJ(type)() { return cast(type*) mem_calloc(type.sizeof); }
```
The [`filenamecmp` macro is replaced](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L56) with [a function](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R74).

Support for [obsolete platforms is removed](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L45).

Global variables in D are placed by default into thread-local storage (TLS). But since `make` is a single-threaded program, they can be inserted into global storage with the [`__gshared` storage class](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R82). (See the documentation [for the `__gshared` attribute](https://dlang.org/spec/attribute.html#gshared).)

    // int CMDLINELEN;
    __gshared int CMDLINELEN
D doesn’t have a separate struct tag name space, so the typedefs are not necessary. An
[`alias`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R113) can be used instead. (See the documentation [for `alias` declarations](https://dlang.org/spec/declaration.html#alias).) Also, [`struct`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R131) is omitted from variable declarations.

```python
/*
typedef struct FILENODE
        {       char            *name,genext[EXTMAX+1];
                char            dblcln;
                char            expanding;
                time_t          time;
                filelist        *dep;
                struct RULE     *frule;
                struct FILENODE *next;
        } filenode;
*/
struct FILENODE
{
        char            *name;
        char[EXTMAX1]  genext;
        char            dblcln;
        char            expanding;
        time_t          time;
        filelist        *dep;
        RULE            *frule;
        FILENODE        *next;
}

alias filenode = FILENODE;
```
[`macro` is a keyword in D](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L86), so we’ll just use `MACRO` instead.

Grouping together [multiple pointer declarations](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L83) is not allowed in D, [use this instead](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R98):

    // char *name,*text;
    // In D, the * is part of the type and 
    // applies to each symbol in the declaration.
    char* name, text;
[C array declarations](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L111) are transformed to [D array declarations](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R131). (See the documentation [for D’s declaration syntax](https://dlang.org/spec/declaration.html#declaration_syntax).)

    // char            *name,genext[EXTMAX+1];
    char            *name;
    char[EXTMAX+1]  genext;
[`static`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L159) has no meaning at module scope in D. `static` globals in C are equivalent to `private` module-scope variables in D, but that doesn’t really matter when the module is never imported anywhere. They still need to be `__gshared` and that can be [applied to an entire block of declarations](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R191). (See the documentation [for the `static` attribute](https://dlang.org/spec/attribute.html#static))

```d
/*
static ignore_errors = FALSE;
static execute = TRUE;
static gag = FALSE;
static touchem = FALSE;
static debug = FALSE;
static list_lines = FALSE;
static usebuiltin = TRUE;
static print = FALSE;
...
*/

__gshared
{
    bool ignore_errors = false;
    bool execute = true;
    bool gag = false;
    bool touchem = false;
    bool xdebug = false;
    bool list_lines = false;
    bool usebuiltin = true;
    bool print = false;
    ...
}
```
[Forward reference declarations](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L189) for functions are not necessary in D. Functions defined in a module can be called at any point in the same module, before or after their definition.

[Wildcard expansion](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L240) doesn’t have much meaning to a `make` program.

[Function parameters declared with array syntax](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L242) are pointers in reality, and are declared as pointers in D.

    // int cdecl main(int argc,char *argv[])
    int main(int argc,char** argv)
[`mem_init()` expands to nothing](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L248) and we previously removed the macro.

C code can play fast and loose with [arguments to functions](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L275), D demands that function prototypes be respected.

```d
void cmderr(const char* format, const char* arg) {...}

// cmderr("can't expand response file\n");
cmderr("can't expand response file\n", null);
```
[Global search/replace C's arrow operator](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L297) (`->`) with the dot operator (`.`), as member access in D is uniform.

Replace [conditional compilation directives](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L303) with D’s `version`.

```d
/*
 #if TERMCODE
    ...
 #endif
*/
    version (TERMCODE)
    {
        ...
    }
```
The [lack of function prototypes](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R312) shows the age of this code. D requires proper prototypes.

    // doswitch(p)
    // char *p;
    void doswitch(char* p)
[`debug` is a D keyword](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L335). Rename it to `xdebug`.

The [`\n\` line endings](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L396) for C multiline string literals are not necessary in D.

[Comment out unused code](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R463) using D’s `/+ +/` nesting block comments. (See the documentation [for line, block and nesting block comments](https://dlang.org/spec/lex.html#comment).)

[`static if` can replace](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R503) many uses of `#if`. (See the documentation [for the `static if` condition](https://dlang.org/spec/version.html#staticif).)

[Decay of arrays to pointers](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L585) is not automatic in D, use `.ptr`.

    // utime(name,timep);
    utime(name,timep.ptr);
[Use `const` for C-style strings](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L658) derived from string literals in D, because D won’t allow taking mutable pointers to string literals. (See the documentation [for `const` and `immutable`](https://dlang.org/spec/const3.html#const_and_immutable).)

    // linelist **readmakefile(char *makefile,linelist **rl)
    linelist **readmakefile(const char *makefile,linelist **rl)
[`void*` cannot be implicitly cast to `char*`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R920). Make it explicit.

```python
// buf = mem_realloc(buf,bufmax);
buf = cast(char*)mem_realloc(buf,bufmax);
```
[Replace `unsigned` with `uint`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L1099).

[`inout` can be used to transfer](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R1160) the “const-ness” of a function from its argument to its return value. If the parameter is `const`, so will be the return value. If the parameter is not `const`, neither will be the return value. (See the documentation [for `inout` functions](https://dlang.org/spec/function.html#inout-functions).)

```d
// char *skipspace(p) {...}
inout(char) *skipspace(inout(char)* p) {...}
```
[`arraysize` can be replaced](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L1628) with the `.length` property of arrays. (See the documentation [for array properties](https://dlang.org/spec/arrays.html#array-properties).)

    // useCOMMAND  |= inarray(p,builtin,arraysize(builtin));
    useCOMMAND  |= inarray(p,builtin.ptr,builtin.length)
String literals are immutable, so it is necessary to [replace mutable ones with a stack allocated array](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L1683). (See the documentation [for string literals](https://dlang.org/spec/lex.html#string_literals).)

    // static char envname[] = "@_CMDLINE";
    char[10] envname = "@_CMDLINE";
[`.sizeof` replaces C’s `sizeof()`](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L1688). (See the documentation [for the `.sizeof` property](https://dlang.org/spec/property.html#sizeof)).

```python
// q = (char *) mem_calloc(sizeof(envname) + len);
q = cast(char *) mem_calloc(envname.sizeof + len);
```
Don’t care about [old versions of Windows](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55R1692).

Replace ancient C usage of [`char *` with `void*`.](https://github.com/DigitalMars/Compiler/commit/a25bee1ba3a3b5231fd80aa1b24aa18ce3eb3a34#diff-14c44b49b9f46b1aeab33a6684214e55L1773)

And that wraps up the changes! See, not so bad. I didn’t set a timer, but I doubt this took more than an hour, including debugging a couple errors I made in the process.

This leaves the file [man.c](https://github.com/DigitalMars/Compiler/commit/473bf5bef99a1748d5154b343b986534271cd841#diff-69c1b24ec5df3f3b5a5bb99bc60345f7), which is used to open the browser on the [make manual page](https://digitalmars.com/ctg/make.html) when the `-man` switch is given. Fortunately, this was already ported to D, so we can just copy [that code](https://github.com/DigitalMars/Compiler/commit/7765879be1be6fcb3eb959d1598d8ff227c2b0af#diff-2bac52ea58e12548536bb246cced3af4).

Building `make` is so easy it doesn’t even need a makefile:

    \dmd2.079\windows\bin\dmd make.d dman.d -O -release -betterC -I. -I\dmd2.079\src\druntime\import\ shell32.lib
## Summary


We’ve stuck to the Evil Plan of translating a non-trivial old school C program to D, and thereby were able to do it quickly and get it working correctly. An equivalent executable was generated.

The issues encountered are typical and easily dealt with:



 	
  * Replacement of `#include` with `import`

 	
  * Lack of D versions of `#include` files

 	
  * Global search/replace of things like `->`

 	
  * Replacement of preprocessor macros with:

 	
```d
* manifest constants

 	
* simple templates

 	
* functions

 	
* version declarations

 	
* debug declarations
```
  * Handling identifiers that are D keywords

 	
  * Replacement of C style declarations of pointers and arrays

 	
  * Unnecessary forward references

 	
  * More stringent typing enforcement

 	
  * Array handling

 	
  * Replacing C basic types with D types


None of the following was necessary:

 	
  * Reorganizing the code

 	
  * Changing data or control structures

 	
  * Changing the flow of the program

 	
  * Changing how the program works

 	
  * Changing memory management




## Future


Now that it is in DasBetterC, there are lots of modern programming features available to improve the code:



 	
  * [modules!](https://dlang.org/spec/module.html)

 	
  * memory safety (including [buffer overflow checking](https://dlang.org/spec/arrays.html#bounds))

 	
  * metaprogramming

 	
  * [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)

 	
  * Unicode

 	
  * [nested functions](https://dlang.org/spec/function.html#nested)

 	
  * [member functions](https://dlang.org/spec/class.html#member-functions)

 	
  * [operator overloading](https://dlang.org/spec/operatoroverloading.html)

 	
  * [documentation generation](https://dlang.org/spec/ddoc.html)

 	
  * [functional programming support](https://dlang.org/spec/function.html#pure-functions)

 	
  * [Compile Time Function Execution](https://dlang.org/spec/function.html#interpretation)

 	
  * [etc.](https://dlang.org/spec/spec.html)




## Action


Let us know over at the [D Forum](https://forum.dlang.org/group/general) how your DasBetterC project is coming along!
