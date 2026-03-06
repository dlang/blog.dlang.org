---
author: DBlogAdmin
comments: false
date: 2017-10-25 15:54:36+00:00
excerpt: Before diving into a series about C and D, a bit of a primer is called for.
  That’s where this post comes in. The primary goal is to help ensure a C environment
  is installed and working on Windows. It’s also useful to understand why things are
  different on that platform than on the others. Before we get to the why, we'll dig
  into the how.
layout: post
link: https://dlang.org/blog/2017/10/25/dmd-windows-and-c/
slug: dmd-windows-and-c
title: DMD, Windows, and C
wordpress_id: 1148
categories:
- Code
- Compilers &amp; Tools
permalink: /dmd-windows-and-c/
redirect_from: /2017/10/25/dmd-windows-and-c/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)The ability to interface with C was baked into D from the beginning. Most of the time, it’s something that requires little thought – as long as the declarations on the D side match what exists on the C side, things will usually just work. However, there are a few corner-case gotchas that arise from the simple fact that D, though compatible, is not C.

An upcoming series of posts here on the D blog will delve into some of these dark corners and shine a light on the traps lying in wait. In these posts, readers will be asked to follow along by compiling and executing the examples themselves so they may more thoroughly understand the issues discussed. This means that, in addition to a D compiler, readers will need access to a C compiler.

That raises a potential snafu. On the systems that the DMD frontend groups under the [`version(Posix)`](https://dlang.org/spec/version.html#predefined-versions) umbrella, it’s a reliable assumption that a C compiler is easily available (if a D compiler is installed and functioning properly, the C compiler will already be installed). The concept of a _system compiler_ is a long established tradition on those systems. On Windows… not so much.

So before diving into a series about C and D, a bit of a primer is called for. That’s where this post comes in. The primary goal is to help ensure a C environment is installed and working on Windows. It’s also useful to understand why things are different on that platform than on the others. Before we get to the why, we'll dig into the how.

First, assume we have the following two source files in the same directory.

**cfoo.c**

```c
#include <stdio.h>

void say_hello(void) 
{
    puts("Hello!");
}
```
**dfoo.d**

```d
extern(C) void say_hello();

void main() 
{
    say_hello();
}
```
Now let’s see how to get the two working together.


### DMD and C


The [DMD packages for Windows](https://dlang.org/download.html) ship with everything the compiler needs: a linker and other tools, plus a handful of critical system libraries. So on the one hand, Windows is the only platform where DMD has no external dependencies out of the box. On the other hand, it’s the only platform where a working DMD installation does not imply a C compiler is also installed. And these days, the out-of-the-box experience often isn't the one you want.

On all the other platforms, the C compiler option is the system compiler, which in practice means GCC or Clang. The system linker to which DMD sends its generated object files might be [`ld`](ftp://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html), [`lld`](https://lld.llvm.org/), [`ld.gold`](https://en.wikipedia.org/wiki/Gold_(linker)), or any `ld`-compatible linker. On Windows, there are currently two compiler choices, and neither can be assumed to be installed by default: the Digital Mars C and C++ compiler, `dmc`, or the Microsoft compiler, `cl`. We’ll look at each in turn.


#### DMD and DMC


The linker ([Optlink](http://www.digitalmars.com/ctg/optlink.html)) and other tools that ship with DMD are also part of the DMC distribution. DMD uses these tools by default (or when the `-m32` switch is passed on the command line). To link any C objects or static libraries, they should be in the [OMF format](https://en.wikipedia.org/wiki/Relocatable_Object_Module_Format). Compilers that can generate OMF are a rare breed these days, and while something like [Open Watcom](https://en.wikipedia.org/wiki/Watcom_C/C%2B%2B) may work, using DMC will guarantee 100% compatibility.

The DMC package (version 8.57 as I write) can be [downloaded from digitalmars.com](http://digitalmars.com/download/freecompiler.html). It’s a 3 MB zip file that can be unzipped anywhere. Personally, since I only ever use it in conjunction with DMD, I keep it in `C:\D` so that the `dm` directory is a sibling of the `dmd2` directory. Once it’s unzipped, it can be added to the path if desired. Be aware that some of the tools DMD and DMC ship with may conflict with tools in other packages if they are on the global path.

For example, both come with Digital Mars `make` and Optlink, which is named `link.exe`. The former might conflict with Cygwin, or a MinGW distribution that’s independent of MSYS2 (if `mingw32-make` has been renamed), and the latter with Microsoft’s linker (which generally shouldn’t be on the global path anyway). Some may prefer just to keep it all off the global path. In that case, it’s simple to configure a command prompt shortcut that sets the PATH when it launches. For example, create a batch file, that looks like this:

    echo Welcome to your Digital Mars environment.
    @set PATH=C:\D\dmd2\windows\bin;C:\D\dm\bin;%PATH%
Save it as `C:\D\dmenv.bat`. Right click an empty spot on the desktop and, from the popup menu, select `New->Shortcut`. In the location field, enter the following:

    C:\System\Win32\cmd.exe /k C:\d\dmenv.bat
Now you have a shortcut that, when double clicked, will launch a command prompt that has both `dmd` and `dmc` on the path.

Once installed, documentation on the command-line switches for the tools is available at the Digital Mars site. The most relevant are the docs for [DMC](http://www.digitalmars.com/ctg/sc.html), [Optlink](http://www.digitalmars.com/ctg/optlink.html), and [Librarian](http://www.digitalmars.com/ctg/lib.html) (`lib.exe`). The latter two will come in handy even when doing pure D development with vanilla DMD, as those are the tools needed to when manually manipulating its object file output.

That’s all there is to it. As long as both `dmc.exe` and `dmd.exe` are on the path in any given command prompt, both compilers will find the tools they need via the default settings in their configuration files. For knocking together quick tests with both C and D on Windows, it’s a quick thing to launch a command prompt, compile & link, and execute:

```bash
dmc -c cfoo.c
dmd dfoo.d cfoo.obj
dfoo
```
Easy peasy. Now let’s look at the other option.


#### DMD and Microsoft’s CL


Getting DMD to work with the Microsoft toolchain requires installing the Microsoft build tools and the Windows SDK. The easiest way to get everything is to use one of the Community editions of Visual Studio. The installer will download and install all the tools and the SDK. The latest is always available from [https://www/visualstudio.com](https://www.visualstudio.com/). With VS 2017, the installer has been overhauled such that it’s possible to minimize the size of the install more than was possible with past editions. An alternative is to install the [Microsoft Build Tools](http://landinghub.visualstudio.com/visual-cpp-build-tools) and the [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk) separately. However, this is still a large install that isn’t much of a win in light of the new VS 2017 installer options (for those on Windows 8.1 or 10).

Once the tooling is installed, DMD’s configuration file needs to be modified to point its environment variables to the proper locations. Rather than repeat all of that here, I’ll direct you to the [DMD installation page](https://wiki.dlang.org/Installing_DMD) at the D Wiki. One of the reasons to prefer the DMD installer over the zip archive is that it will detect any installation of Visual Studio or the Microsoft Build Tools and automatically modify the configuration as needed. This is more convenient than needing to remember to update the configuration every time a new version of DMD is installed. It also offers to install VS 2013 Community if the tooling isn’t found, can install [Visual D](http://rainers.github.io/visuald/visuald/StartPage.html) (the D plugin for Visual Studio), and will add DMD to the system path if you want it to.

It’s a bit of an annoyance to launch Visual Studio for simple tests between C and D. Since it’s not recommended to put the MS tools on the system path, each VS and Microsoft Build Tools installation ships with a number of batch files that will set the path for you (like the one we created for DMC above). The installer sets up shortcuts in the Windows Start menu. There are several different options to choose from. To launch a 64-bit environment with VS 2017 (or the 2017 build tools), find `Visual Studio 2017` in the Start menu and select `x64 Native Tools Command Prompt for VS 2017`. For VS 2015 (or the 2015 build tools), go to `Visual Studio 2015` and click on `VS 2015 x64 Native Build Tools Command Prompt`. Similar options exist for 32-bit (where `x86` replaces `x64`) and cross compiling.

From the VS-enabled 64-bit environment, we can run the following commands to compile our two files.

```bash
cl /c cfoo.c
dmd -m64 dfoo.d cfoo.obj
dfoo
```
In a 32-bit VS environment, replace `-m64` with `-m32mscoff`.


### The consequences of history


When a new programming language is born these days, it’s not uncommon for its tooling to be built on top of an existing toolchain rather than completely from scratch. Whether we’re talking about languages like Kotlin built on the JRE, or those like Rust using LLVM, reusing existing tools saves time and allows the developers to focus their precious man-hours on the language itself and any language-specific tooling they require.

When Walter Bright first started putting D together in 1999, that trend had not yet come around. However, he already had an existing toolchain in the form of the Digital Mars C and C++ compiler tools. So it was a no-brainer to make use of his existing tools and compiler backend and just focus on making a new frontend for DMD. There were four major side-effects of this decision, all of which had varying consequences in D's future development.

First, the DMC tools were Windows-only, so the early versions of DMD would be as well. Second, the linker, Optlink, only supports the OMF format. That meant that DMD’s output would be incompatible with the more common [COFF](https://en.wikipedia.org/wiki/COFF) output of most modern C and C++ compilers on Windows. Third, the DMC tools do not support 64-bit, so DMD would be restricted to 32-bit output. Finally, Symantec had the legal rights to the existing backend, which meant their license would apply to DMD. While the frontend was open source, the backend license required one to get permission from Walter to distribute DMD (on a side note, this prevented DMD from being included in official Linux package repositories once Linux support was added, but [Symantec granted permission](http://forum.dlang.org/thread/oc8acc$1ei9$1@digitalmars.com) to relicense the backend earlier this year and it is now freely distributable under the Boost license).

[DMD 0.00](http://www.digitalmars.com/d/1.0/changelog1.html#new000) was released in December of 2001. The [0.63 release](http://www.digitalmars.com/d/1.0/changelog1.html#new063) brought Linux support in May of 2003. Walter could have based the Linux version on the GCC backend, but as a business owner, and through a caution born from past experience, he was concerned about any legal issues that could arise from his working with GPL code on one platform and maintaining a proprietary backend on another. Instead, he modified the DMD backend to generate ELF objects and hand them off to the GCC tools. This decision to enhance the backend became the approach for all new formats going forward. He did the same when adding support for Mac OS X: he modified the backend to work with the [Mach-O object format](https://en.wikipedia.org/wiki/Mach-O).

Along with the new formats, the compiler gained the ability to generate 64-bit binaries everywhere except Windows. In order to interface with C on Windows, it was usually necessary to convert COFF object files and static libraries to OMF, to use a tool like [coffimplib](http://www.digitalmars.com/ctg/coffimplib.html) to generate DLL import libraries in the OMF format, or to create dynamic bindings and load DLLs manually via `LoadLibrary` and `GetProcAddress`. Then Remedy Games decided to use D.

[Quantum Break](https://www.remedygames.com/games/quantumbreak/) was the first AAA game title [to ship with D](https://dconf.org/2016/talks/watson.html) as part of its development process. Remedy used it for their gameplay code, creating their own [open source tool](https://github.com/Remedy-Entertainment/binderoo) to [bind with](https://dconf.org/2017/talks/watson.html) their C++ game engine. Before they could get that far, however, they needed 64-bit support in DMD on Windows. That was the motivator to get it implemented. It took a while (apparently, there are some undocumented quirks in [Microsoft's variant of COFF](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680547(v=vs.85).aspx), a.k.a PECOFF, a.k.a. MS-COFF), but Walter eventually got it done, and support for 32-bit COFF along with it. Again, as he had on other platforms, he modified the backend to generate object files in the new format.

This is why it’s necessary to have the Microsoft toolchain installed in order to produce 64-bit binaries with DMD on Windows. Microsoft’s `cl` is as close to a _system compiler_ as one is going to get on Windows. There is, however, an option that has not yet been fully explored. It's [a toolchain](http://mingw-w64.org/doku.php) that can be freely distributed, packaged with [a reasonable download size](https://nuwen.net/mingw.html), supports 32-bit and 64-bit output, and is mostly compatible with PECOFF. There is a possibility that it may be investigated as an option for future DMD releases to be built upon.


### Going from here


Now that this primer is out of the way, the short series on C is just about ready to go. It will kick off with a brief summary of existing material, showing how easy it is to get D and C to work together in the general case. That will be followed up by two posts on arrays and strings. This is where most of the gotchas come into play, and anyone using D and C in the same program should understand what they are and how to avoid them.
