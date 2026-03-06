---
author: KaiNacke
comments: false
date: 2019-06-25 15:50:19+00:00
layout: post
link: https://dlang.org/blog/2019/06/25/fuzzing-your-d-application-with-ldc-and-afl/
slug: fuzzing-your-d-application-with-ldc-and-afl
title: Fuzzing Your D Application with LDC and AFL
wordpress_id: 2092
categories:
- Code
- Guest Posts
- Tutorials
permalink: /fuzzing-your-d-application-with-ldc-and-afl/
redirect_from: /2019/06/25/fuzzing-your-d-application-with-ldc-and-afl/
---

![](http://dlang.org/blog/wp-content/uploads/2017/07/ldc.png)Fuzzing, or fuzz testing, is a powerful method to find hidden bugs in your application. The basic idea is to present random input to your application and monitor how it behaves. If it crashes or shows some other unusual behavior then you have found a bug.

The use of true random input is not very effective, as most applications reject such input. Therefore many fuzz testing tools mutate valid input, e.g. flipping one or two bits, and present this mutated input to the application. This approach is easy to automate. A fuzz test can run for hours or days until an input is found which crashes your application.

Fuzz testing is very popular. A lot of security bugs have been found with this method. So it’s better to fuzz test your application by yourself instead of waiting for your users to report serious bugs!

Johan Engelen showed at [DConf 2018](http://dconf.org/2018/talks/engelen.html) and in more detail in a [blog post](https://johanengelen.github.io/ldc/2018/01/14/Fuzzing-with-LDC.html) how you can use LLVM [libFuzzer](https://llvm.org/docs/LibFuzzer.html) to fuzz test your application. For libFuzzer, you need to write a test driver. This is powerful because you can make decisions about the function to test. The downside is that you have to code the test driver.

[AFL](http://lcamtuf.coredump.cx/afl/) (short for [American Fuzzy Lop](https://en.wikipedia.org/wiki/American_Fuzzy_Lop), a rabbit breed) is another tool to fuzz test an application. AFL has a different approach than libFuzzer and does not require coding. The application under test has to read its data from `stdin` or from a file. The binary must be instrumented, which requires a recompile of the application. In case you have no source code for the application you can use AFL together with QUEMU. No instrumentation is required but the tests run much slower.

Because random input is not a good choice, you give AFL one or more valid input files, preferably of a small size. AFL mutates this input file, e.g. by flipping a single bit. This new input file is presented to your application and the reaction on it is observed. With the instrumentation in place, AFL discovers the path the data takes through your application. The relationship between bit flips and different code paths that run because of the bit flips is recorded and used to discover new paths and to trigger unexpected behavior. Input which causes crashes is saved in a directory. The main UI gives a lot of information, including how many unique crashes occured in the test session.

AFL works best if the input is a small binary, e.g. a PNG or a ZIP file. If your application has a more verbose and structured input (e.g. a programming language) then you can provide a dictionary which helps AFL with the basic syntax.

The latest release of AFL has an interesting feature. For instrumenting code compiled with clang, a small LLVM plugin is used. This plugin can also be used with LDC, making it possible to fuzz test your D application!

I used AFL to fuzz test [LLtool](https://github.com/redstar/LLtool), my recursive-descent parser generator presented at [DConf 2019](https://youtu.be/PJpGySqFm2U). LLtool expects a grammar description as a file or on `stdin`. If no error is found, then a D fragment of a recursive-descent parser is produced. Here, I show my approach.

First of all, you need to install AFL. It is included in most Linux distributions, e.g. Ubuntu. A FreeBSD port is also available. One caveat here: please make sure that the AFL plugin is compiled with the same LLVM version as LDC. Otherwise you will see an error message like

    ld-elf.so.1: /usr/local/lib/afl/afl-llvm-pass.so: Undefined symbol "...."
during compilation. In this case, download AFL from the link above and compile it yourself.

Different distributions install AFL in different locations. You need to find out the path. E.g. Ubuntu uses `/usr/lib/afl`, FreeBSD uses `/usr/local/lib/afl`. I use an environment variable to record this value for later use (bash syntax):

    export AFL_PATH=`/ust/lib/afl`
To instrument your code you have to specify the AFL plugin on the LDC command line:

```bash
ldc2 -plugin=$AFL_PATH/afl-llvm-pass.so *.d
```
You will see a short statistic emitted by the new pass:

    afl-llvm-pass 2.52b by <lszekeres@google.com>
    [+] Instrumented 16118 locations (non-hardened mode, ratio 100%).
For LLVM instrumentation, AFL requires a small runtime library. You need to link the object file `$AFL_PATH/afl-llvm-rt.o` into your application.

In my `dub.sdl` file I created a special build type for AFL. This puts all the steps above into a single place. Plus, you can copy and paste this build type directly to your own `dub.sdl` file because the only dependencies are AFL and LDC!

```d
buildType "afl" {
    toolchainRequirements dmd="no" gdc="no" ldc=">=1.0.0"
    dflags "-plugin=$AFL_PATH/afl-llvm-pass.so"
    sourceFiles "$AFL_PATH/afl-llvm-rt.o"
    versions "AFL"
    buildOptions "debugMode" "debugInfo" "unittests"
}
```
Now you can type `dub build -b=afl` on the command line to instrument your application for use with afl. Do not forget to set the `AFL_PATH` environment variable, otherwise `dub` will complain.

Now create two new directories called `testcases` and `findings`. Put a small, valid input file into the `testcases` directory. For example save this

    %token number
    %%
    expr: term "+" term;
    term: factor "*" factor;
    factor: number;
as file `t1.g` in the `testcases` folder. Inputs which crash the application will be saved in the `findings` directory.

To call AFL, you type on the command line:

    afl-fuzz -i testcases -o findings ./LLtool --DRT-trapExceptions=0 @@
Two parts of the command line require further explanation. If the application requires a file for input, you specify the file path as `@@`. Otherwise AFL assumes that the application reads the input from `stdin`.

If the application crashes, then AFL saves the input causing the crash in the `findings/crashes` directory. But the D runtime is very friendly. Exceptions uncaught by the application are caught by the D runtime, a stack trace is printed, and the application terminates. This does not count as a crash for AFL. To produce a crash you have to specify the D runtime option `--DRT-trapExceptions=0`. For more information, read the relevant edition of [This week in D](http://arsdnet.net/this-week-in-d/2016-aug-07.html).

It is worth reading the AFL documentation because there it provides a lot of tips and background information. Enjoy watching AFL crashing your application and producing test cases for you!



* * *



_A long-time contributor to the D community, Kai Nacke is the author of '[D Web Development](https://www.packtpub.com/web-development/d-web-development)' and a maintainer of LDC, [the LLVM D Compiler](https://wiki.dlang.org/LDC)._
