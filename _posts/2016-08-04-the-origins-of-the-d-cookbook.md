---
author: AdamRuppe
comments: false
date: 2016-08-04 14:14:21+00:00
layout: post
link: https://dlang.org/blog/2016/08/04/the-origins-of-the-d-cookbook/
slug: the-origins-of-the-d-cookbook
title: The Origins of the D Cookbook
wordpress_id: 140
categories:
- Books
- Guest Posts
permalink: /the-origins-of-the-d-cookbook/
redirect_from: /2016/08/04/the-origins-of-the-d-cookbook/
---

_Adam Ruppe is the author of [D Cookbook](http://amzn.to/1ZTE47m) and maintainer of [This Week in D](http://arsdnet.net/this-week-in-d). Modules from his freely available [arsd package](https://github.com/adamdruppe/arsd) are used throughout the D community. He is also known for his [legendary DConf 2014 presentation](http://dconf.org/2014/talks/ruppe.html)._



* * *



![dcookbook](http://dlang.org/blog/wp-content/uploads/2016/08/dcookbook.jpg)In 2013, Packt Publishing approached me to write [D Cookbook](https://www.packtpub.com/application-development/d-cookbook). Over the next several months, I was tasked with collecting over 100 D programming language "recipes" and writing a couple of pages about each one.

I wanted it to be a mix of practical and fun examples of what D can do over the wide variety of fields in which I have used it, each one illustrative of some language concept that the reader could use in different contexts. As such, for the majority of the book, I avoided the use of libraries, even keeping the presence of [Phobos ](https://dlang.org/phobos/index.html)or my own modules to a minimum, allowing me to focus on the implementations of the ideas.

That said, I didn't come up with a hundred recipes out of thin air! They came from three main sources: [my arsd libraries](https://github.com/adamdruppe/arsd), my support efforts on [the D forums](http://forum.dlang.org/), [the #D IRC channel](irc://irc.freenode.net/d), and [Stack Overflow](http://stackoverflow.com/questions/tagged/d) (it was Stack Overflow that got Packt's attention in the first place), and a few other side projects, like my minimal D runtime ([zip file](http://arsdnet.net/dcode/minimal.zip)).

I first saw D way back in 2002. At the time, I was still fairly new to programming and was going to download another copy of [Digital Mars C++](http://digitalmars.com/download/dmcpp.html) (which I used to compile 16-bit DOS programs!) when I saw a link on the Digital Mars website about this D programming language. My first impression - upon seeing the `import` keyword - was that it was too Java-like and I wasn't interested. Oh, how I regret that naive snap decision! I didn't take my next look at D until 2006, and that time, with far more real world experience and theoretical knowledge under my belt, I quickly fell in love.

Over the next year, I wrote a game library based on [SDL](http://libsdl.org/) and [OpenGL](https://www.opengl.org/) and made a few little games for myself. My D code at this point wasn't terribly unique. Much of it was based on my existing C++ codebases. That fairly uninteresting fact would later become one of my most recurring D tips: a great deal of your C, C++, and Java knowledge can carry over directly to D! Thanks to the similarities between the languages, it's easy to learn. While your code is unlikely to work perfectly if simply copy/pasted into a **.d** file, porting it probably isn't hard. I found the D language very comfortable right off the bat. Even today, I tell people with D questions to simply consider how they would do it in C++ and apply that existing knowledge to D. This can also work with C and C++ solutions gleaned from Internet resources like Stack Overflow.

I started working as a full-time web developer in 2008, which put a time constraint on my hobby game development efforts, but didn't ice my love of D. Indeed, it wasn't long at all before I worked D into my professional life by writing web libraries and eventually transitioning my work projects away from PHP and onto D!

At the same time, the D language was going through a series of rapid changes. Templates and compile time function evaluation got overhauled, `immutable` was introduced... Most interestingly to me, compile time reflection got massively expanded and a few features like [opDispatch](https://dlang.org/spec/operatoroverloading.html#dispatch) got added. Compile time reflection in older D was limited to the `is` expression and template tricks, but new D had `__traits`, making it competitive even with dynamic languages, without sacrificing D's other strengths.

I was a very early adopter of these features and set out to discover how to combine them in ways to make my work easier, to compete with the other web languages, and to just show off a little :) If someone came on the forum and said _D can't do this_, then I'd reply _Challenge accepted_.

In the following years, I wrote: a DOM and JSON library, taking the most interesting ideas from Javascript into D; a web framework, realizing some dreams I had in rapidly churning out prototypes that could also survive the change process of real world development; OAuth and database libraries to support the needs of the projects; and more. One of the most interesting modules was my [web.d](https://github.com/adamdruppe/arsd/blob/master/web.d), which takes a D class definition and builds web infrastructure around it: an automatically generated web site with CRUD forms from the static type information as well as Javascript and PHP code for API access to that functionality. This stretched D's reflection capabilities and hit quite a few bugs on the bleeding edge, necessitating creative workarounds or alternative approaches. If I was really desperate, I'd fix the bug in DMD myself!

At the same time, I was regularly seen in the D community, helping other people with their problems, and, of course, reading insights from other members of the community. Every few days, I had another tip, and was also building up a mental picture of people's common difficulties with D.

By 2013, I had years of experience with almost every corner and combination of D. Now, you can get a good slice of that in just a few days by reading the book!
