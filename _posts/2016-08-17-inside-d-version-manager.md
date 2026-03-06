---
author: JacobCarlborg
comments: false
date: 2016-08-17 12:15:52+00:00
layout: post
link: https://dlang.org/blog/2016/08/17/inside-d-version-manager/
slug: inside-d-version-manager
title: Inside D Version Manager
wordpress_id: 183
categories:
- Compilers &amp; Tools
- Guest Posts
permalink: /inside-d-version-manager/
redirect_from: /2016/08/17/inside-d-version-manager/
---

_In his day job, Jacob Carlborg is a Ruby backend developer for [Derivco Sweden](http://www.derivco.se), but he's been using D on his own time since 2006. He is the maintainer of numerous open source projects, including [DStep](https://github.com/jacob-carlborg/dstep), a utility that generates D bindings from C and Objective-C headers, [DWT](https://github.com/d-widget-toolkit), a port of the Java GUI library [SWT](https://www.eclipse.org/swt/), and the topic of this post, [DVM](https://github.com/jacob-carlborg/dvm). He implemented native Thread Local Storage support for DMD on OS X and contributed, along with Michel Fortin, to [the integration of Objective-C](https://dlang.org/spec/objc_interface.html) in D._



* * *



[D Version Manager (DVM)](https://github.com/jacob-carlborg/dvm), is a cross-platform tool that allows you to easily download, install and manage multiple D compiler versions. With DVM, you can select a specific version of the compiler to use without having to manually modify the **PATH** environment variable. A selected compiler is unique in each shell session, and it's possible to configure a default compiler.

The main advantage of DVM is the easy downloading and installation of different compiler versions. Specify the version of the compiler you would like to install, e.g. `dvm install 2.071.1`, and it will automatically download and install that version. Then you can tell DVM to use that version by executing `dvm use 2.071.1`. After that, you can invoke the compiler as usual with `dmd`. The selected compiler version will persist until the end of the shell session.

DVM makes it possible for the user to select a specific compiler version without having to modify any makefiles or build scripts. It's enough for any build script to refer to the compiler by name, i.e. `dmd`, as long as the user selects the compiler version with DVM before invoking the script.


## History


DVM was created in the beginning of 2011. That was a different time for D. No proper installers existed, D1 was still a viable option, and each new release of DMD brought with it a number of regressions. Because of all the regressions, it was basically impossible to always use the latest compiler, and often even older compilers, for all of your projects. Taking into consideration projects from other developers, some were written in D1 and some in D2, making it inconvenient to have only one compiler version installed.

It was for these reasons I created DVM. Being able to have different versions of the compiler active in different shell sessions makes it easy to work on different projects requiring different versions of the compiler. For example, it was possible to open one tab for a D1 compiler and another for a D2 compiler.

The concept of DVM comes directly from the Ruby tool [RVM](https://rvm.io). Where DVM installs D compilers, RVM installs Ruby interpreters. RVM can do everything DVM can do and a lot more. One of the major things I did _not_ want to copy from RVM is that it's completely written in shell script (bash). I wanted DVM to be written in D. Because it's written in shell script, RVM enables some really useful features that DVM does not support, but some of them are questionable (some might call them hacks). For example, when navigating to an RVM-enabled project, RVM will automatically select the correct Ruby interpreter. However, it accomplishes this by overriding the built-in `cd` command. When the command is invoked, RVM will look in the target directory for one of the files .**rvmrc** or **.ruby-version**. If either is present, it will read that file to determine which Ruby interpreter to select.


## Implementation and Usage


One of the goals of DVM was that it should be implemented in D. In the end, it was mostly written in D with a few bits of shell script. Note that the following implementation details are specific to the platforms that fall under D's **Posix** umbrella, i.e. `version(Posix)`, but DVM is certainly available for Windows with the same functionality.


### Structure of the DVM Installation


Before DVM can be used, it needs to install itself. This is accomplished with the command, `dvm install dvm`. This will create the **~/.dvm** directory. It contains the following subdirectories: **archives**, **bin**, **compilers**, **env** and **scripts**.

**archives** contains a cache of downloaded zip archives of D compilers.

**bin** contains shell scripts, acting as symbolic links, to all installed D compilers. The name of each contains the version of the compiler, e.g. **dmd-2.071.1**, making it possible to invoke a specific compiler without first having to invoke the `use` command. This directory also contains one shell script, **dvm-current-dc**, pointing to the currently active D compiler. This allows the currently active D compiler to be invoked without knowing which version has been set. This can be useful for executing the compiler from within an editor or IDE, for example. A shell script for the default compiler exists as well. Finally, this directory also contains the binary **dvm** itself.

The **compilers** directory contains all installed compilers. All of the downloaded compilers are unpacked here. Due to the varying quality of the D compiler archives throughout the years, the `install` command will also make a few adjustments if necessary. In the old days, there was only one archive for all platforms. This command will only include binaries and libraries for the current platform. Another adjustment is to make sure all executables have the executable permission set.

The **env** directory contains helper shell scripts for the `use` command. There's one script for each installed compiler and one for the default selected compiler.

The **scripts** directory currently only contains one file, **dvm**. It's a shell script which wraps the **dvm** binary in the **bin** directory. The purpose of this wrapper is to aid the `use` command.


### The use Command


The most interesting part of the implementation is the `use` command, which selects a specific compiler, e.g. `dvm use 2.071.1`. The selection of a compiler will persist for the duration of the shell session (window, tab, script file).

The command works by prepending the path of the specified compiler to the **PATH** environment variable. This can be **~/.dvm/compilers/dmd-2.071.1/{platform}/bin** for example, where **{platform}** is the currently running platform. By prepending the path to the environment variable, it guarantees the selected compiler takes precedence over any other possible compilers in the **PATH**. The reason the **{platform}** section of the path exists is related to the structure of the downloaded archive. Keeping this structure avoids having to modify the compiler's configuration file, **dmd.conf**.

The interesting part here is that it's not possible to modify the environment variables of the parent process, which in this case is the shell. The magic behind the `use` command is that the `dvm` command that you're actually invoking is not the D binary; it's the shell script in the **~/.dvm/scripts** path. This shell script contains a function called `dvm`. This can be verified by invoking `type dvm | head -n 1`, which should print _dvm is a function_ if everything is installed correctly.

The installation of DVM adds a line to the shell initialization file, **.bashrc**, **.bash_profile** or similar. This line will load/source the DVM shell script in the **~/.dvm/scripts** path which will make the `dvm` command available. When the `dvm` function is invoked, it will forward the call to the **dvm** binary located in **~/.dvm/bin/dvm**. The **dvm** binary contains all of the command logic. When the `use` command is invoked, the **dvm** binary will write a new file to **~/.dvm/tmp/result** and exit. This file contains a command for loading/sourcing the environment file available in **~/.dvm/env** that corresponds to the version that was specified when the `use` command was invoked. After the **dvm** binary has exited, the shell script function takes over again and loads/sources the result file if it exists. Since the shell script is loaded/sourced instead of executed, the code will be evaluated in the current shell instead of a sub-shell. This is what makes it possible to modify the **PATH** environment variable. After the result file is loaded/sourced, it's removed.

If you find yourself with the need to build your D project(s) with multiple compiler versions, such as the current release of DMD, one or more previous releases, and/or the latest beta, then DVM will allow you to do so in a hassle-free manner. Pull up a shell, execute `use` on the version you want, and away you go.
