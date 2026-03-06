---
author: DBlogAdmin
comments: false
date: 2017-05-31 12:43:08+00:00
layout: post
link: https://dlang.org/blog/2017/05/31/project-highlight-excel-d/
slug: project-highlight-excel-d
title: 'Project Highlight: excel-d'
wordpress_id: 826
categories:
- Code
- Project Highlights
permalink: /project-highlight-excel-d/
redirect_from: /2017/05/31/project-highlight-excel-d/
---

Ever had the need to write an Excel plugin? Check this out.

![](http://dlang.org/blog/wp-content/uploads/2017/05/excel-300x158.png)Atila Neves opened his [lightning talk](https://youtu.be/xJy6ifCekCE?list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB) at [DConf 2017](http://dconf.org/2017/index.html) like this:


I'm going to talk about how you can write Excel add-ins in D. Don't ask me why. It's just because people need it.


From there he goes into a quick intro on how to write plugins for Excel and gives a taste of what it looks like to register a single function in a C++ implementation:

    Excel12f(
        xlfRegister, NULL, 11,          // 11: Number of args
        &xDLL,                          // name of the DLL
        TempStr12(L"Fibonacci"),        // procedure name
        TempStr12(L"UU"),               // type signature
        TempStr12(L"Compute to..."),    // argument text
        TempStr12(L"1"),                // macro type
        TempStr12(L"Generic Add-In"),   // category
        TempStr12(L""),                 // shortcut text
        TempStr12(L""),                 // help topic
        TempStr12(L"Number to compute to"), // function help
        TempStr12(L"Computes the nth Fibonacci number") // arg help 
    );
Two things to note about this. First, `Excel12f` is a C++ function (wrapping a C API) that must be called in an add-in DLL's entry point (`xlAutoOpen`) for each function that needs to be registered when the add-in is loaded by Excel. For a small plugin, the implementations of any registered functions might be in the same source file, but in a larger one they might be a bit of a maintenance headache by being located somewhere else. Also take note of all the comments used to document the function arguments, a common sight in C and C++ code bases.

The example D code Atila showed using [excel-d](https://github.com/kaleidicassociates/excel-d) is a world of difference:

```d
@Register(
    ArgumentText("Array"),
    HelpTopic("Length of Array"),
    FunctionHelp("Length of an Array"),
    ArgumentHelp(["array"])
)
double DoublesLength(double[] arg) {
    return arg.length;
}
```
Here, the boilerplate for the registration is being generated at compile-time via a [User Defined Attribute](http://dlang.org/spec/attribute.html#UserDefinedAttribute), which is used to annotate the function. Implementation and registration are happening in the same place. Another key difference is that the UDA has fields with descriptive names, eliminating the need to comment each argument. Finally, the UDA only requires four arguments, nine less than the C++ function. This is because it makes use of D's compile-time introspection features to figure out as much as it possibly can and, at the same time, optional arguments (like the shortcut text) can just be omitted.

Since this is a Project Highlight on the D Blog, we're going to ignore Atila's opening request and ask, "Why?" There are actually two parts to that. First, why Excel?


Our customers are traders, and they work with Excel as one of their main tools. They need/want to, amongst other things, receive live stock updates in a cell and have their formulae automatically update. There's other functionality they'd like to have and that means adding this to Excel somehow.


Of all the possible languages that could be used for this purpose, the business chose D. That brings us to the second part of the question: why D?


This is possible in Visual Basic, Python or C#, and possibly other languages. But none of them match D's performance. C++ does, but it's tedious and requires a lot of boilerplate to get going. D combines the speed and power of C++ with the reflection capabilities of those other languages. No boilerplate, just code, runs fast.


There's more to the story, of course. The company is heavily invested in D.


We use D for nearly everything, even some "scripts". The bulk of it is calculations for market indicators. Lots of data in -> munge -> new data out that needs to look pretty for traders. So integrating with existing code was an important factor.


Even though excel-d is targeting Windows, much of it was actually developed on Linux.


We use a Linux container as our reference development machine, but people use what they want. I do nearly all of my work on Linux and only boot into Windows when I have to. For the Excel work, that's a necessity. But, as usual for me, I wrote all the tests to be platform agnostic, so I do the Excel development on Linux and test there. Every now and again that means a particular quirk of Excel wasn't captured well in my mocking code, but it's usually a quick fix after that.


He says they use both DMD and LDC for development, and both are running in continuous integration.

Although DMD doesn't technically require [Visual Studio](https://www.visualstudio.com/) to be installed (out of the box, it generates 32-bit [OMF objects](https://en.wikipedia.org/wiki/Relocatable_Object_Module_Format), and uses the [OPTLINK linker](http://www.digitalmars.com/ctg/optlink.html), rather than the VS-compatible [COFF](https://en.wikipedia.org/wiki/COFF)), anyone doing serious work on Windows is going to need VS (or the [Visual Studio Build Tools](http://landinghub.visualstudio.com/visual-cpp-build-tools) and the [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk)) for 64-bit and 32-bit COFF support. The latest [LDC binary releases](https://github.com/ldc-developers/ldc) currently require the MS tools (support for [MinGW](https://mingw-w64.org/doku.php) was dropped, but according to the [D Wiki](https://wiki.dlang.org/LDC#MinGW), could be picked up again if there's a champion for it).

Atila already had VS on his Windows partition. For this project, he got a bit of help from the VS plugin for D, [Visual D](https://dlang.org/blog/2016/08/12/project-highlight-visual-d/).


I had to install VisualD because our reference project for Excel was in a Visual Studio solution, but afterwards I reverse engineered the build and didn't open Visual Studio ever again.


Currently, excel-d has no support for custom dialogs or menus. Both items are on his TODO list.

If you're working with D and need to write an Excel add-in, or want to try something cleaner than C++ to do so, excel-d is available in the [DUB package registry](https://code.dlang.org/packages/excel-d). If not, the sponsors of the project, Kaleidic Associates and [Symmetry Investments](http://symmetryinvestments.com/about-us/), have made several other [open source projects](https://github.com/kaleidicassociates) available. They are interested in hiring talented hackers with a moral compass who aspire towards excellence and would like to work in D.

_excel-d was developed by Stefan Koch and Atila Neves._
