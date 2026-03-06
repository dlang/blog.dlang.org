---
author: DBlogAdmin
comments: false
date: 2017-12-05 15:52:46+00:00
excerpt: One of the early design goals behind the D programming language was the ability
  to interface with C. To that end, it provides ABI compatibility, allows access to
  the C standard library, and makes use of the same object file formats and system
  linkers that C and C++ compilers use. Most built-in D types, even structs, are directly
  compatible with their C counterparts and can be passed freely to C functions, provided
  the functions have been declared in D with the appropriate linkage attribute. In
  many cases, one can copy a chunk of C code, paste it into a D module, and compile
  it with minimal adjustment. Conversely, appropriately declared D functions can be
  called from C.
layout: post
link: https://dlang.org/blog/2017/12/05/interfacing-d-with-c-getting-started/
slug: interfacing-d-with-c-getting-started
title: 'Interfacing D with C: Getting Started'
wordpress_id: 1257
categories:
- Code
- D and C
- Tutorials
permalink: /interfacing-d-with-c-getting-started/
redirect_from: /2017/12/05/interfacing-d-with-c-getting-started/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)One of the early design goals behind the D programming language was the ability to [interface with C](https://dlang.org/spec/interfaceToC.html). To that end, it provides ABI compatibility, allows access to the C standard library, and makes use of the same object file formats and system linkers that C and C++ compilers use. Most built-in D types, even structs, are directly compatible with their C counterparts and can be passed freely to C functions, provided the functions have been declared in D with the appropriate [linkage attribute](https://dlang.org/spec/attribute.html#linkage). In many cases, one can copy a chunk of C code, paste it into a D module, and compile it with minimal adjustment. Conversely, appropriately declared D functions can be called from C.

That’s not to say that D carries with it all of C’s warts. It includes features intended to eliminate, or more easily avoid, some of the errors that are all too easy to make in C. For example, bounds checking of arrays is enabled by default, and a safe subset of the language provides compile-time enforcement of memory safety. D also changes or avoids some things that C got wrong, such as what Walter Bright sees as [C’s biggest mistake](http://www.drdobbs.com/architecture-and-design/cs-biggest-mistake/228701625): conflating pointers with arrays. It’s in these differences of implementation that surprises lurk for the uninformed.

This post is the first in a series exploring the interaction of D and C in an effort to inform the uninformed. I’ve previously written about the basics of this topic in [an article at GameDev.net](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/binding-d-to-c-r3122/), and in my book, ‘[Learning D](http://amzn.to/1IlQkZX)’, where the entirety of Chapter 9 covers it in depth.

This blog series will focus on those aforementioned corner cases so that it’s not necessary to buy the book or to employ trial and error in order to learn them. As such, I’ll leave the basics to the GameDev.net article and recommend that anyone interfacing D with C (or C++) give it a read along with [the official documentation](https://dlang.org/spec/interfaceToC.html).

The C and D code that I provide to highlight certain behavior is intended to be compiled and linked by the reader. The code demonstrates both error and success conditions. Recognizing and understanding compiler errors is just as important as knowing how to fix them, and seeing them in action can help toward that end. That implies some prerequisite knowledge of compiling and linking C and D source files. Happily, that’s the focus of the next section of this post.

For the C code, we’ll be using the Digital Mars C/C++ and Microsoft C/C++ compilers on Windows, and GCC and Clang elsewhere. On the D side, we’ll be working exclusively with the D reference compiler, DMD. Windows users unfamiliar with setting up DMD to work with the Microsoft tools will be well served by the post on this blog titled, ‘[DMD, Windows, and C](https://dlang.org/blog/2017/10/25/dmd-windows-and-c/)’.

We’ll finish the post with a look at one of the corner cases, one that is likely to rear its head early on in any exploration of interfacing D with C, particularly when creating bindings to existing C libraries.


### Compiling and linking


The articles in this series will present example C source code that is intended to be saved and compiled into object files for linking with D programs. The command lines for generating the object files look pretty much the same on every platform, with a couple of caveats. We’ll look first at Windows, then lump all the other supported systems together in a single section.

In the next two sections, we’ll be working with the following C and D source files. Save them in the same directory (for convenience) and make sure to keep the names distinct. If both files have the same name in the same directory, then the object files created by the C compiler and DMD will also have the same name, causing the latter to overwrite the former. There are compiler switches to get around this, but for a tutorial we’re better off keeping the command lines simple.

**chello.c**

```c
#include <stdio.h>
void say_hello(void) 
{
    puts("Hello from C!");
}
```
**hello.d**

```d
extern(C) void say_hello();
void main() 
{
    say_hello();
}
```
The `extern(C)` bit in the declaration of the C function in the D code is [a linkage attribute](https://dlang.org/spec/attribute.html#linkage). That's covered by the other material I referenced above, but it's a potential gotcha we'll look at later in this series.


#### Windows


The official DMD packages for Windows, [available at dlang.org](https://dlang.org/download.html) as a zip archive and an installer, are the only released versions of DMD that do not require any additional tooling to be installed as a prerequisite to compile D files. These packages ship with everything they need to compile 32-bit executables in the OMF format (again, I refer you to ‘[DMD, Windows, and C](https://dlang.org/blog/2017/10/25/dmd-windows-and-c/)’ for the details).

When linking any foreign object files with a D program, it’s important that the object file format and architecture match the D compiler output. The former is an issue primarily on Windows, while attention must be paid to the latter on all platforms.

Compiling C source to a format compatible with vanilla DMD on Windows requires [the Digital Mars C/C++ compiler](http://digitalmars.com/download/freecompiler.html). It’s a free download and ships with some of the same tools as DMD. It outputs object files in the OMF format. With both it and DMD installed and on the system path, the above source files can be compiled, linked, and executed like so:

```bash
dmc -c chello.c
dmd hello.d chello.obj
hello
```
The `-c` option tells DMC to forego linking, causing it to only compile the C source and write out the object file `chello.obj`.

To get 64-bit output on Windows, DMC is not an option. In that case, DMD requires the Microsoft build tools on Windows. Once the MS build tools are installed and set up, open the preconfigured x64 Native Tools Command Prompt from the Start menu and execute the following commands (again, see ‘[D, Windows, and C](https://dlang.org/blog/2017/10/25/dmd-windows-and-c)’ on this blog for information on how to get the Microsoft build tools and open the preconfigured command prompt, which may have a slightly different name depending on the version of Visual Studio or the MS Build Tools installed):

```bash
cl /c chello.c
dmd -m64 hello.d chello.obj
hello
```
Again, the `/c` option tells the compiler not to link. To produce 32-bit output with the MS compiler, open a preconfigured x86 Native Tools Command Prompt and execute these commands:

```bash
cl /c hello.c
dmd -m32mscoff hello.c chello.obj
hello
```
DMD recognizes the `-m32` switch on Windows, but that tells it to produce 32-bit OMF output (the default), which is not compatible with Microsoft’s linker, so we must use `-m32mscoff` here instead.


#### Other platforms


On the other platforms D supports, the system C compiler is likely going to be GCC or Clang, one of which you will already have installed if you have a functioning `dmd` command. On Mac OS, `clang` can be installed via `XCode` in the App Store. Most Linux and BSD systems have a GCC package available, such as via the often recommended command line, `apt-get install build-essential`, on Debian and Debian-based systems. Please see the documentation for your system for details.

On these systems, the environment variable `CC` is often set to the system compiler command. Feel free to substitute either `gcc` or `clang` for `CC` in the lines below as appropriate for your system.

```bash
CC -c chello.c
dmd hello.d chello.o
./hello
```
This will produce either 32-bit or 64-bit output, depending on your system configuration. If you are on a 64-bit system and have 32-bit developer tools installed, you can pass `-m32` to both `CC` and `dmd` to generate 32-bit binaries.


### The `long` way


Now that we’re configured to compile and link C and D source in the same binary, let’s take a look at a rather common gotcha. To fully appreciate this one, it helps to compile it on both Windows and another platform.

One of the features of D is that all of the integral types have a fixed size. A `short` is always 2 bytes and an `int` is always 4. This never changes, no matter the underlying system architecture. This is quite different from C, where the spec only imposes relative requirements on the size of each integral type and leaves the specifics to the implementation. Even so, there are wide areas of agreement across modern compilers such that on every platform D currently supports the sizes for almost all the integral types match those in D. The exceptions are `long` and `ulong`.

In D, `long` and `ulong` are always 8 bytes across all platforms. This never changes. It lines up with the corresponding C types just fine on most 64-bit systems under the `version(Posix)` umbrella, where the C `long` and `unsigned long` are also 8 bytes. However, they are 4 bytes on 32-bit architectures. Moreover, they’re _always_ 4 bytes on Windows, even on a 64-bit architecture.

Most C code these days will account for these differences either by using the preprocessor to define custom integral types or by making use of the C99 `stdint.h` where types such as `int32_t` and `int64_t` are unambiguously defined. Yet, it’s still possible to encounter C libraries using `long` in the wild.

Consider the following C function:

**maxval.c**

```c
#include <limits.h>
unsigned long max_val(void)
{
    return ULONG_MAX;
}
```
The naive D implementation looks like this:

**showmax1.d**

```d
extern(C) ulong max_val();
void main()
{
    import std.stdio : writeln;
    writeln(max_val());
}
```
What this does depends on the C compiler and architecture. For example, on Windows with `dmc` I get `7316910580432895`, with x86 `cl` I get `59663353508790271`, and `4294967295` with x64 `cl`. The last one is actually the correct value, even though the size of the `unsigned long` on the C side is still 4 bytes as it is in the other two scenarios. I assume this is because the x64 ABI stores return values in the 8-byte `RAX` register, so it can be read into the 8-byte `ulong` on the D side with no corruption. The important point here is that the two values in the x86 code are garbage because the D side is expecting a 64-bit return value from 32-bit registers, so it's reading more than it's being given.

Thankfully, DRuntime provides a way around this in `core.c.config`, where you’ll find `c_long` and `c_ulong`. Both of these are conditionally configured to match the compile-time C runtime implementation and architecture configuration. With this, all that’s needed is to change the declaration of `max_val` in the D module, like so:

**showmax2.d**

```d
import core.stdc.config : c_ulong;
extern(C) c_ulong max_val();

void main()
{
    import std.stdio : writeln;
    writeln(max_val());
}
```
Compile and run with this and you’ll find it does the right thing everywhere. On Windows, it's `4294967295` across the board.

Though less commonly encountered, `core.stdc.config` also declares a portable `c_long_double` type to match any `long double` that might pop up in a C library to which a D module must bind.


### Looking ahead


In this post, we’ve gotten set up to compile and link C and D in the same executable and have looked at the first of several potential problem spots. We used DMD here, but it should be possible to substitute one of the other D compilers (`ldc` or `gdc`) without changing the command line (with the exception of `-m32mscoff`, which is specific to DMD). The next post in this series will focus entirely on getting D arrays and C arrays to cooperate. See you there!
