---
author: AtilaNeves
comments: false
date: 2019-10-15 12:51:40+00:00
layout: post
link: https://dlang.org/blog/2019/10/15/my-vision-of-ds-future/
slug: my-vision-of-ds-future
title: My Vision of D's Future
wordpress_id: 2225
categories:
- Core Team
- D Foundation
- The Language
permalink: /my-vision-of-ds-future/
redirect_from: /2019/10/15/my-vision-of-ds-future/
---

![](http://dlang.org/blog/wp-content/uploads/2019/08/royal_d-200x200.png)When Andrei Alexandrescu stepped down as deputy leader of the D programming language, I was asked to take over the role going forward. It’s needless to say, but I’ll say it anyway, that those are some pretty big shoes to fill.

I’m still settling into my new role in the community and figuring out how I want to do things and what those things even are. None of this happens in a vacuum either, since Walter needs to be on board as well.

I was asked [on the D forums](https://forum.dlang.org/post/lcrsrevmvtnbqsqktsbe@forum.dlang.org) to write a blog post on my “dreams and way forward for D”, so here it is. What I’d like to happen with D in the near future:


### Memory safety


“But D has a GC!”, I hear you exclaim. Yes, but it’s also a systems programming language with value types and pointers, meaning that today, D isn’t memory safe. [DIP1000 was a step in the right direction](https://github.com/dlang/DIPs/blob/master/DIPs/other/DIP1000.md), but we have to be memory safe unless programmers opt-out via a “I know what I’m doing” `@trusted` block or function. This includes transitioning to `@safe` by default.


### Safe and easy concurrency


We’re mostly there—using the actor model eliminates a lot of problems that would otherwise naturally occur. We need to finalize `shared` and make everything `@safe` as well.


### Make D the default implementation language


D’s static reflection and code generation capabilities make it an ideal candidate to implement a codebase that needs to be called from several different languages and environments (e.g. Python, Excel, R, …). Traditionally this is done by specifying data structures and RPC calls in an Interface Definition Language (IDL) then translating that to the supported languages, with a wire protocol to go along with it.

With D, none of that is necessary. One can write the production code in D and have libraries automagically make that code callable from other languages. Add to all of this that it’s possible and easy to write D code that runs as fast or faster than the alternatives, and it’s a win on all fronts.


### Second to none reflection.


Instead of disparate ways of getting things done with fragmented APIs (`__traits`, `std.traits`, custom code), I’d like for there to be a library that centralizes all reflection needs with a great API. I’m currently working on it.


### Easier interop with C++.


As I mentioned in [my DConf 2019 talk](https://www.youtube.com/watch?v=79COPHF3TnE&t=1s), C++ has had the success it’s enjoyed so far by making the transition from C virtually seamless. I want current C++ programmers with legacy codebases to just as easily be able to start writing D code. That’s why I wrote dpp, but it’s not quite there yet and we might have to make language changes to accommodate this going forward.


### Fast development times.


I think we need a ridiculously fast interpreter so that we can skip machine code generation and linking. To me, this should be the default way of running `unittest` blocks for faster feedback, with programmers only compiling their code for runtime performance and/or to ship binaries to final users. This would also enable a REPL.


### String interpolation


I was initially against this, but the more I think about it the more it seems to make sense for D. Why? String mixins. Code generation is one of D’s greatest strengths, [and token strings enable](https://dlang.org/spec/lex.html#token_strings) visually pleasing blocks of code that are actually “just strings”. String interpolation would make them vastly easier to use. As it happens, [there’s a draft DIP for it](https://github.com/dlang/DIPs/pull/165) in the pipeline.

That’s what I came up with after a long walk by Lake Geneva. I’d love to know what the community thinks of this, what their pet peeves and pet features would be, and how they think this would help or hinder D going forward.
