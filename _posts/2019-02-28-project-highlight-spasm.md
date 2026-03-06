---
author: DBlogAdmin
comments: false
date: 2019-02-28 12:16:02+00:00
excerpt: Spasm allows Single Page Apps to be written in D and compiled into WebAssembly.
  In this post, Spasm creator and maintainer Sebastiaan Koppe explains how the project
  came about, what it does, and where it's going.
layout: post
link: https://dlang.org/blog/2019/02/28/project-highlight-spasm/
slug: project-highlight-spasm
title: 'Project Highlight: Spasm'
wordpress_id: 1961
categories:
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-spasm/
redirect_from: /2019/02/28/project-highlight-spasm/
---

![](http://dlang.org/blog/wp-content/uploads/2019/02/hackathon.png)In 2014, Sebastiaan Koppe was working on a React project. The app’s target market included three-year-old mobile phones. He encountered some performance issues that, after investigating, he discovered weren’t attributable solely to the mobile platform.


It all became clear to me once I saw the flame graph. There were so many function calls! We managed to fix the performance issues by being a little smarter about updates and redraws, but the sight of that flame graph never quite left me. It made me wonder if things couldn’t be done more efficiently.


As time went on, Sebastiaan gained the insight that a UI is like a state machine.


Interacting with the UI moves you from one state to the next, but the total set of states and the rules governing the UI are all static. In fact, we require them to be static. Each time a user clicks a button, we want it to respond the same way. We want the same input validations to run each time a user submits a form.

So, if everything is static and defined up-front, why do all the javascript UI frameworks work so hard during runtime to figure out exactly what you want rendered? Would it not be more efficient to figure that out before you send the code to the browsers?


This led him to the idea of analyzing UI definitions to create an optimal UI renderer, but he was unable to act on it at the time. Then in 2018, native-language DOM frameworks targeting WebAsm, like [asm-dom for C++](https://mbasso.github.io/asm-dom/) and [Percy for Rust](https://chinedufn.github.io/percy/), came to his attention. Around the same time, the announcement of [Vladimir Panteleev’s dscripten-tools](https://github.com/CyberShadow/dscripten-tools) introduced him to [Sebastien Alaiwan’s older dscripten project](https://github.com/Ace17/dscripten). The former is an alternative build toolchain for the latter, which is an example of compiling D to [asm.js via Emscripten](https://github.com/emscripten-core/emscripten). Here he saw an opportunity to revisit his old idea using D.


D’s static introspection gave me the tools to create render code at compile time, bypassing the need for a virtual DOM. The biggest challenge was to map existing UI declarations and patterns to plain D code in such a way that static introspection can be used to extract all of the information necessary for generating the rendering code.


One thing he really wanted to avoid was the need to embed [React’s Javascript extension, JSX,](https://reactjs.org/docs/introducing-jsx.html) in the D code, as that would require the creation of a compile-time parser. Instead, he decided to leverage the D compiler.


For the UI declarations, I ended up at a design where every HTML node is represented by a D struct and all the node’s attributes and properties are members of that struct. Combined with annotations, it gives enough information to generate optimal render code. With that, I implemented [the famous todo-mvc application](https://github.com/skoppe/d-wasm-todomvc-poc). The end result was quite satisfying. The actual source code was on par or shorter than most javascript implementations, the compiled code was only 60kB after gzip, and rendering various stages in the todo app took less than 2ms.


He announced his work on the D forums in September of 2018 ([the live demo is still active](https://skoppe.github.io/d-wasm-todomvc-poc/) as I write).

Unfortunately, he wasn’t satisfied with the amount of effort involved to get the end result. It required using [the LLVM-based D compiler, LDC,](https://wiki.dlang.org/LDC) to compile D to LLVM IR, then using Emscripten to produce asm.js, and [finally using binaryen](https://github.com/WebAssembly/binaryen) to compile that into WebAssembly. On top of that…


…you needed a patched version of LLVM to generate the asm.js, which required a patched LDC. Compiling all those dependencies takes a long time. Not anything I expect end users to willfully subject themselves to. They just want a compiler that’s easy to install and that just works.


As it happened, [the 1.11.0 release of LDC](https://github.com/ldc-developers/ldc/releases/tag/v1.11.0) in August 2018 actually had rudimentary [support for WebAssembly baked in](https://wiki.dlang.org/Generating_WebAssembly_with_LDC). Sebastiaan started rewriting the todo app and his Javascript glue code to use LDC’s new WebAssembly target. In doing so, he lost Emsripten's bundled musl libc, so he switched his D code to use `-betterC` mode, [which eliminates D’s dependency on DRuntime](https://dlang.org/blog/2017/08/23/d-as-a-better-c/) and, in turn, the C standard library.

With that, he had the easy-to-install-and-use package he wanted and was able to get the todo-mvc binary down to 5kb after gzip. When he announced this news in the D forums, he was led down a new path.


Someone asked about WebGL. That got me motivated to think about creating bindings to the APIs of the browser. That same night I did a search and found underrun, [an entry in the 2018 js13k competition](https://js13kgames.com/entries/underrun). I decided to port it to D and use it to figure out how to write bindings to WebGL and WebAudio.


He created the bindings by hand, but later [he discovered WebIDL](https://heycam.github.io/webidl/). After [the underrun port](https://skoppe.github.io/spasm/examples/underrun/) was complete, he started work on using WebIDL to generate bindings.


It formed very quickly over the course of 2–3 months. The hardest part was mapping every feature of WebIDL to D, and at the same time figuring out how to move data, objects and callbacks between D and Javascript. All kinds of hard choices came up. Do I support `undefined`? Or optional types? Or union types? What about the “any” type? The answer is yes, all are supported.


He’s been happy with the result. D bindings to web APIs are included in Spasm and they follow the Javascript API as much as possible. A post-compile step is used to generate Javascript glue code.


It runs fairly quickly and generates only the Javascript glue code you actually need. It does that by collecting imported functions from the generated WebAssembly binary, cross-referencing those to functions to WebIDL definitions and then generating Javascript code for those.


The result of all this is Spasm, [a library for developing](https://github.com/skoppe/spasm) single-page WebAssembly applications in D. The latest version is always available [in the DUB repository](https://code.dlang.org/packages/spasm).


I can’t wait to start working on hot module replacement with Spasm. Many Javascript frameworks provide it out-of-the box and it is really valuable when developing. Server-side rendering is also something that just wants to get written. While Spasm doesn’t need it so much to reduce page load times, it is necessary when you want to unittest HTML output or do SEO.


At the moment, his efforts are directed toward creating a set of basic material components. He’s had a hard time getting something together that works in plain D, and at one point considered abandoning the effort and working instead on a declarative UI language that compiles to D, but ultimately he persisted and will be announcing the project soon.


After the material project there are still plenty of challenges. The biggest thing I have been postponing is memory management. Right now the allocator in Spasm is a simple bump-the-pointer allocator. The memory for the WebAssembly instance in the browser is hardcoded to 16mb and when it's full, it's full. I could grow the memory of course, but I really need a way to reuse memory. Without help from the compiler - like Rust has - that either means manual memory management or a GC.


One way to solve the problem would be [to port DRuntime](https://github.com/dlang/druntime) to WebAssembly, something he says he’s considered “a few times” already.


At least the parts that I need. But so far the GC has always been an issue. In WebAssembly, memory is linear and starts at 0. When you combine that with an imprecise GC, suddenly everything looks like a pointer and, as a consequence, it won’t free any memory. Recently someone wrote a precise GC implementation. So that is definitely back on the table.


He’s also excited that he recently ran WebAssembly generated from D on Cloudflare workers.


The environment is different from a browser, but its the same in many ways. This is all very exciting and lots of possibilities will emerge for D. In part because you can generate WebAssembly binaries that are pretty lean and mean.


We’re pretty excited about the work Sebastiaan is doing and can’t wait to see where it goes. Keep an eye on the [Dlang Newsfeed (@dlang_ng)](https://twitter.com/dlang_ng) on Twitter, or [the official D Twitter feed (@D_Programming)](https://twitter.com/D_Programming) to learn about future Spasm announcements.
