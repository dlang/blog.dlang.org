---
author: DBlogAdmin
comments: false
date: 2017-01-30 13:03:24+00:00
layout: post
link: https://dlang.org/blog/2017/01/30/project-highlight-dpaste/
slug: project-highlight-dpaste
title: 'Project Highlight: DPaste'
wordpress_id: 603
categories:
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-dpaste/
redirect_from: /2017/01/30/project-highlight-dpaste/
---

[DPaste](https://dpaste.dzfl.pl/new) is an online compiler and collaboration tool for the D Programming Language. Type in some D code, click run, and see the results. Code can also be saved so that it can be shared with others. Since it was first announced in the forums [back in 2012](https://forum.dlang.org/post/xfvfjjdhmovvuebfyjjg@forum.dlang.org), it has become a frequently used tool in facilitating online discussions in the D community. But Damian Ziemba, the creator and maintainer of DPaste, didn't set out with that goal in mind.


Actually it was quite spontaneous and random. I was hanging out on the [#D IRC channel at freenode](irc://freenode.net/D). I was quite amazed at how active this channel was. People were dropping by asking questions, lots of code snippets were floating around. One of the members created an IRC bot that was able to compile code snippets, but it was for his own language that he created with D. Someone else followed and created the same kind of bot, but with the ability to compile code in D, though it didn't last long as it was run on his own PC. So I wrote my own, purely in D, that was compiling D snippets. It took me maybe 4-5 hours to write both an IRC support lib and the logic itself. Then some server hardening where the bot was running and voila, we had `nazbot @ #D`, which was able to evaluate statements like `^stmt import std.stdio; writeln("hello world");` and would respond with, `"hello world"`.


Nazbot became popular and people started floating new ideas. That ultimately led Damian to take a CMS he had already written in PHP and repurpose it to use as a frontend for what then became DPaste.


The frontend is written in PHP and uses MySQL for storage. It acts as a web interface (using a Bootstrap HTML template and jQuery) and API provider for 3rd Parties. The backend is responsible for actual compilation and execution. It's possible to use multiple backends. The frontend is a kind of load-balancer when it comes to choosing a backend. The frontend and the backend may live on different machines.


DPaste is primarily used through the web interface, but it's also used by dlang.org.


Once dpaste.dzfl.pl was well received, the idea popped up that maybe we could provide runnable examples on the main site. So it was implemented. The next idea, proposed by Andrei Alexandrescu, was to enable runnable examples on all of the Phobos documentation. I got swallowed by real life and couldn't contribute at the time, but eventually Sebastian Wilzbach took it up and [finished the implementation](http://forum.dlang.org/thread/o3969r$2qed$1@digitalmars.com). So today we have interactive examples in [the Phobos documentation](https://dlang.org/phobos/index.html).


When Damian first started work on DPaste in 2011, the D ecosystem looked a bit different than it does today.


There weren't as many 3rd party libraries as we have now; there was no [DUB](http://code.dlang.org/getting_started), there was no [vibe.d](http://vibed.org/), etc. I wish I'd had vibe.d back then. I would have implemented the frontend in D instead of PHP.

What I enjoy the most about D is just how "nice" to the eye the language is (compared to C and C++, which I work with on a daily basis) and how easy it is to express what's in your mind. I've never had to stop and think, "how the hell can I implement this", which is quite common with C++ in my case. In the current state, what is also amazing is how D is becoming a "batteries-included" package. Whatever you need, you just `dub fetch` it.


He's implemented DPaste such that it requires very little in terms of maintenance costs. It automatically updates itself to the latest compiler release and also knows how to restart itself if the backend hangs for some reason. He says the only real issue he's had to deal with over the past five years is spam, which has forced him to reimplement the captcha mechanism several times.

As for the future? He has a few things in mind.


I plan to rewrite the backend from scratch, open source it and use a docker image so anybody can easily pick up development or host his own backend (which is almost done). Functionally, I want to maintain different compiler versions like DMD 2.061.0, DMD 2.062.1, DMD 2.063.0, LDC 0.xx, GDC x.xx.xx, etc., and connect more architectures as backends (currently x86, arm and aarch64 are planned).

I also want to rewrite the frontend in D using vibe.d, websockets, and angular.js. In general, I would like to make the created applications more interactive. So, for example, you could use the output from your code snippet in realtime as it is produced. I would like also to split a middle end off from the frontend. The middle end would provide communication with backends and offer both a REST API and websockets. Then the frontend would be responsible purely for user interaction and nothing else.


He would also like to see DPaste become more _official_, perhaps through making it a part of dlang.org. And for a point further down the road, Damian has an even grander plan.


I hope to make a full blown online IDE for dlang.org with workspaces, compilers to chose, and so on.


That would be cool to see!
