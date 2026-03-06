---
author: DBlogAdmin
comments: false
date: 2017-03-31 13:35:58+00:00
layout: post
link: https://dlang.org/blog/2017/03/31/project-highlight-workspace-d/
slug: project-highlight-workspace-d
title: 'Project Highlight: workspace-d'
wordpress_id: 715
categories:
- Compilers &amp; Tools
- IDEs
- Project Highlights
permalink: /project-highlight-workspace-d/
redirect_from: /2017/03/31/project-highlight-workspace-d/
---

Not so long ago, Jan Jurzitza sat down at his keyboard intent on writing a D plugin for [Atom](https://atom.io/), his text editor of choice at the time. Then came disappointment.

“I was pretty unhappy with their API,” he says.

Visual Studio Code was released a short time after. He decided to give it a go and “instantly fell in love with it”. His Atom plugin was pushed aside and he started work on a new plugin for VS Code called [code-d](https://github.com/Pure-D/code-d).


However, I did not want to maintain the same functionality in two plugins for two different text editors, so I thought that making a program which contains most of the plugin logic, like starting and calling dcd, dscanner, dfmt, etc., would be beneficial and would also help with including D support in more editors and IDEs in the future.


For the uninitiated, [DCD (the D Completion Daemon)](https://github.com/Hackerpilot/DCD), [DScanner](https://github.com/Hackerpilot/Dscanner), and [Dfmt](https://github.com/Hackerpilot/dfmt) are D-oriented tools for plugin developers, all maintained by Brian Schott. They are, respectively, a client-server based auto-completion program, a source code analyzer, and a code formatter. A number of [IDE](http://wiki.dlang.org/IDEs) and [text editor](http://wiki.dlang.org/Editors) plugins employ them directly.

So Jan started work on his new tool and named it [workspace-d](https://github.com/Pure-D/workspace-d).


With workspace-d I want to make it simple for plugin developers to integrate D functionality into their editor of choice. workspace-d is designed to work both as a standalone program through stdio and as a D library. Once I ported most of the code from my Atom extension to workspace-d, I could simply spawn it as a subprocess in code-d, which I got working with it quite quickly.


In addition to porting his [Atom plugin](https://github.com/Pure-D/atomize-d) to use workspace-d, he also created [one for Sublime Text](https://github.com/Pure-D/sublime-d). Currently, he’s not devoting any time to either and is looking for someone else to take over maintenance of one or both. Anyone interested might start by submitting pull requests. Aside from workspace-d itself, Jan’s focus is on code-d.

He’s recently been working on version 2.0 of workspace-d, with a focus on streamlining the way it handles requests.


Using [traits](http://dlang.org/spec/traits.html), [templates](http://dlang.org/spec/template.html), and [CTFE](http://dlang.org/spec/function.html#interpretation) (Compile-Time Function Execution), basically all D compile time magic, I was able to make an automatic wrapper for the functions for version 2.0. Basically, when a request like `{"cmd":"hello"}` comes in, it runs the D function `hello` with its default arguments. If the arguments don’t match, it responds with an error. This system automatically binds function arguments to JSON values and generates a response from the return value.


To deserialize the JSON requests, he’s using [painlessjson](https://github.com/BlackEdder/painlessjson), a third-party library available in [the DUB package registry](https://code.dlang.org/).


It works really great and I can really recommend it for some simple and easy conversions between D types and JSON. This change really cleaned up all the code and made it possible to use workspace-d as a library.


He's also working on a new project, [serve-d](https://github.com/Pure-D/serve-d), that works with Microsoft’s [Language Server Protocol](https://github.com/Microsoft/language-server-protocol).


serve-d is an alternative for the workspace-d command line I/O for those who prefer JSON RPC over my custom binary/JSON mix. It’s fiber based and uses workspace-d as a library, which results in really clean code. There’s an alpha version of the implementation on github already, both the server and a new branch on code-d. With the Language Server Protocol, I’m hoping for easier integration in other editors. The concept is basically the same as workspace-d’s command line interface, but, because Microsoft is such a big company, I’m hoping that more editors by big developers are going to implement this protocol.


Building and installing workspace-d should go pretty smoothly on Linux or OS X, but it’s currently a little bumpy on Windows. Because of an issue Jan has yet to resolve, it can only be built on Windows with [LDC](https://github.com/ldc-developers/ldc/releases).


The auto completion didn’t work for some people on Windows because it got stuck in the `std.process.execute` function when creating a pipe to write to. I couldn’t find any way to reproduce it in a standalone program so I couldn’t file a bug either. So what we did to avoid this issue in the short term was to simply disallow compilation on Windows using DMD. It works just fine when compiled with LDC.


Jan’s primarily a Linux user (he doesn’t own a Mac and only runs Windows in a VM). He credits GitHub user [@Andrepuel](https://github.com/Andrepuel) for getting it operational on OS X, and [@aka-demik](https://github.com/aka-demik) for finding the issue on Windows and verifying that it compiles with LDC. He’ll be grateful to anyone who can help fully resolve the Windows/DMD issue once and for all.

If you're looking to develop a D plugin for your favorite editor, consider taking advantage of the work Jan as already done with workspace-d to save yourself some effort. And VS Code users can put it to use via code-d to get code completion and more. Visit its [VS Code marketplace page](https://marketplace.visualstudio.com/items?itemName=webfreak.code-d) to read reviews and installation instructions.
