---
author: DBlogAdmin
comments: false
date: 2016-09-16 12:44:24+00:00
layout: post
link: https://dlang.org/blog/2016/09/16/project-highlight-timur-gafarov/
slug: project-highlight-timur-gafarov
title: 'Project Highlight: Timur Gafarov'
wordpress_id: 217
categories:
- Game Development
- Project Highlights
permalink: /project-highlight-timur-gafarov/
redirect_from: /2016/09/16/project-highlight-timur-gafarov/
---

![dlib-logo](http://dlang.org/blog/wp-content/uploads/2016/09/dlib-logo.png)To begin with, let's be clear that Timur Gafarov is a person and not a project. The impetus for this post was an open source [first-person shooter](https://en.wikipedia.org/wiki/First-person_shooter), called [Atrium](https://github.com/gecko0307/atrium), that he develops and maintains. In the course of making the game, he has created a few other D projects, each of which could be the focus of its own post. So this time around, we're going to do a plural Project Highlight and introduce you to [the GitHub repository](https://github.com/gecko0307) of Timur Gafarov.

When Timur first came to D six years ago, you might say it was love at first sight.


As an indie game developer with a strong bias toward graphics engines and rendering tech, I always try to keep track of modern compiled languages effective enough for writing real-time stuff. The most obvious choice in this field is C++, and I actually used it for several years until I found D in 2010. I immediately fell in love with the language's clean, beautiful syntax, its powerful template system, the numerous built-in features absent in C++, and the rich and easy to use standard library.


Interestingly, he was actually attracted by one of the things often cited as a turn-off about D back then: the lack of libraries. It was a situation in which he saw opportunities to create things from scratch, without worrying about reinventing the wheel. At the time, [DUB ](https://code.dlang.org/getting_started)was not yet a thing, so the first task he set himself was coding up a build system called [Cook](https://github.com/gecko0307/cook2), which he still uses sometimes instead of DUB.

After that, he was ready to start making a game. He wanted to use [OpenGL](https://www.opengl.org/), and found an [existing binding](https://github.com/DerelictOrg/DerelictGL3) in [Derelict](https://github.com/DerelictOrg) (a collection of dynamic bindings maintained by this post's author -- there also exists a collection of static bindings called [Deimos](https://github.com/D-Programming-Deimos)) that allowed him to do so in D. With that off his plate, he next sat down to write a game engine. The first few steps in that direction resulted in a set of utility packages that coalesced into [dlib](https://github.com/gecko0307/dlib).


At first, there were no clear plans or goals. After a previous period of using third-party engines, I had some experience with low-level graphics coding in C++ and just wanted to port my stuff to D for a start. I began with vector/matrix algebra and image I/O. These efforts resulted in dlib, a general purpose library.


He next turned his attention to 3D physics. Even though there were existing libraries with bindable C interfaces, like [ODE ](http://www.ode.org/)(with a dynamic binding that existed then in the form of [DerelictODE](https://github.com/DerelictOrg/DerelictODE) and [a static binding](https://github.com/D-Programming-Deimos/ODE) in Deimos) and [Newton](https://github.com/MADEAPPS/newton-dynamics) (for which [Deimos-like](https://github.com/resurtm/DeimosGameDynamics) and[ Derelict-like](https://github.com/Sycam0inc/derelictnewton) bindings have since been created by third parties), he just couldn't help himself. Enter [dmech](https://github.com/gecko0307/dmech).


Atrium was born as part of my experiment with writing a 3D physics engine, called dmech. OK, this wasn't strictly necessary, but the chance of being the first one to write this kind of thing in D was so attractive that I couldn't stand it :) Of course, I didn't dare to compete with such industry standards as [Bullet](http://bulletphysics.org/wordpress/), but nevertheless it was an amazing experience. I've learned a lot.


[caption id="attachment_229" align="aligncenter" width="660"]![A screenshot from Atrium.](http://dlang.org/blog/wp-content/uploads/2016/09/013-1024x542.jpg) A screenshot from Atrium.[/caption]

A physics engine and utility library weren't all he needed. That's where [DGL](https://github.com/gecko0307/dgl) comes in.


The next milestone was writing a graphics engine that I call DGL. I can't consider this step fully completed, because I'm never happy with my abstractions and design solutions. Finally, I ended up with some kind of simplified [Physically-Based Rendering](https://www.marmoset.co/toolbag/learn/pbr-theory) pipeline, [Percentage Closer Filtering](http://ogldev.atspace.co.uk/www/tutorial42/tutorial42.html) shadows and multipass rendering, with which I'm sort of satisfied for now.

There was a period when I had to use outdated hardware, so DGL uses OpenGL 1.x, relying on extensions to utilize modern GPU technologies. Yeah, in 2016 this sounds funny :) But recently I started experimenting with OpenGL 4, so the engine is likely to be rewritten. Again!


But for Timur, the graphics system isn't the most important part of Atrium.


It's the collision detection and character kinematics. I think that believable character behavior and interactions with the virtual world are crucial for any realistic game. In Atrium, these are fully physics-based, with as few hacks and workarounds as possible. Rigid body dynamics natively 'talk' with player-controllable kinematics. That means, for example, that a character can be pushed with moving objects and can push other objects himself. A stack of dynamic boxes can be transported via a kinematic moving platform. A gravity gun can be used to move things. And so on. I'm deeply inspired by Valve's masterpieces, [Half-Life 2](https://en.wikipedia.org/wiki/Half-Life_2) and [Portal](https://en.wikipedia.org/wiki/Portal_(video_game)), so I want to make my own 'physical' first person puzzle.


Working on this project has certainly been a labor of love.


It has already taken me about six years of spare-time work and it still exists only in the form of an early gameplay demo. Of course, if I'd used an existing mature engine, like [Unity](https://unity3d.com/) or [UE](https://www.unrealengine.com/what-is-unreal-engine-4), things would be much simpler. Constraining myself to use D for such a complex task may look strange, but it's fun. And that's that.


Through those years, he has learned a good deal about the ups and downs of game development with D. Particularly that ever popular bugbear known as the GC.


Modern D is a very attractive choice as a language for game development. Even the garbage collector is not a problem, because you can use object pools, custom allocators, or simply `malloc` and `free`. The key point is to know when the GC is invoked and try to avoid those cases in performance critical code. Personally, I prefer using `malloc` so that I can free the memory when I want, since `delete` has been deprecated and `destroy`  just releases all the references to an object instance without actually deleting it. Using manual memory management imposes some restrictions on the code--for example, you can't use closures or D's built-in containers--but that, again, is not a big problem. A large effort is currently underway to lessen GC usage in dlib, so that you can use it to write fully unmanaged applications with ease. It has GC-free containers, file I/O streams, image decoders, and so on.


If you are interested in game development with D, [Timur's GitHub repository](https://github.com/gecko0307) should be an early point of call. Even if you aren't making a game or a game engine, you may well find something useful in [dlib](https://github.com/gecko0307/dlib). With its growing list of contributors, it's getting a good deal of care and attention.

Thanks to Timur for taking the time to contribute to this post. We wish him luck with all of his projects!
