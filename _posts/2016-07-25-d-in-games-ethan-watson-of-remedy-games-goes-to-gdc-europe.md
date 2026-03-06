---
author: DBlogAdmin
comments: false
date: 2016-07-25 13:47:59+00:00
layout: post
link: https://dlang.org/blog/2016/07/25/d-in-games-ethan-watson-of-remedy-games-goes-to-gdc-europe/
slug: d-in-games-ethan-watson-of-remedy-games-goes-to-gdc-europe
title: 'D in Games: Ethan Watson of Remedy Games Goes to GDC Europe'
wordpress_id: 120
categories:
- Game Development
- News
permalink: /d-in-games-ethan-watson-of-remedy-games-goes-to-gdc-europe/
redirect_from: /2016/07/25/d-in-games-ethan-watson-of-remedy-games-goes-to-gdc-europe/
---

At DConf 2013 at the Facebook HQ, Manu Evans, then of Remedy Games, gave a talk titled, "[Using D Alongside a Game Engine](http://dconf.org/2013/talks/evans_1.html)" (follow the link for the slides and video). He talked about Remedy's experience getting D to work with their existing C++ game engine. The D landscape was a bit different when they got underway than it is today. For starters, DMD did not support 64-bit targets on Windows and the language had not yet gained support for directly binding to C++. Manu's talk discusses the steps they took to work around such issues and achieve a working system.

Fast forward to 2016. Remedy releases [Quantum Break](http://www.quantumbreak.com/), the first commercial AAA game to ship with bits implemented in D. Ethan Watson comes to DConf 2016 with a talk titled, "[Quantum Break: AAA Gaming with Some D Code](http://dconf.org/2016/talks/watson.html)" (follow the link for the slides, go to YouTube [for the video](https://www.youtube.com/watch?v=OLFBal4Qo_k&list=PL3jwVPmk_PRyTWWtTAZyvmjDF4pm6EX6z&index=4)). In it, he expanded on what Manu had talked about before. Now, he's getting ready to take that message to [Game Developers Conference Europe](http://www.gdceurope.com/).


On the 16th of August this year in Cologne, Germany, I will be presenting a talk titled, "[D: Using an Emerging Language in Quantum Break](http://schedule.gdceurope.com/session/d-using-an-emerging-language-in-quantum-break)". For those who have tuned in to DConf previously, it will ostensibly be a combination of both my talk from 2016 and Manu Evans's talk from 2013, but necessarily cut down due to the fact that it will need to fit two hours of content into one.


He has more in mind than just discussing the technical aspects of Remedy's solution. During informal discussions at DConf, Ethan spoke of how he would like to bring D to the attention of his colleagues at other studios in the game industry. His upcoming talk presents just such an opportunity.


The approach I'll be taking is something of a sales pitch to other game developers. My talk at DConf this year was a bit apologetic on my part as I felt we did not use D anywhere near as much as we should, and that we still had unsolved problems at the end of it. GDC Europe will be a platform for me to talk about the journey toward shipping a game with an emerging language in use; a way to show what benefits the language has over the industry stalwart C++ and other modern languages like C# and Rust; and as a bit of a way to drum up support from other developers and get them to at least look at the language and evaluate it for their own usage.


Additionally, a future event Ethan hinted at in his DConf talk will become reality in time for GDC Europe.


We will also be open sourcing our binding solution for GDC. I've been cleaning it up over the last couple of weeks, to the point where it's effectively a complete reimplementation designed for ease of use, ease of reading, and to support some of the additional ideas I've had for the system since I took over from Manu. It'll basically be our way of saying, "D is great! Here's some code we prepared earlier to help you get up to speed."


That's awesome news for the D community as a whole. D has had [built-in support for binding to C++](https://dlang.org/spec/cpp_interface.html) for a little while and it has improved over several releases, though it's not as fully functional as the support for binding to C (a far simpler task) and likely never will be. There is also an experimental third-party solution called [Calypso](https://wiki.dlang.org/Calypso), which is a fork of the LDC compiler that ultimately aims to provide full and direct interfacing with C++. The availability of an open-source alternative that has been battle-tested in production code can only be a good thing.

Unfortunately, there is one downside to Ethan's presentation.


They've scheduled my talk at the same time as John Romero's talk. Never thought I'd be competing with him for attention.


Who would want to compete for an audience with one of the co-founders of [id software](http://www.idsoftware.com/) and co-creator of franchises like [Doom ](https://en.wikipedia.org/wiki/Doom_(series))and [Quake](https://en.wikipedia.org/wiki/Quake_(series)), who [is giving a talk](http://schedule.gdceurope.com/session/the-early-days-of-id-software-programming-principles) about the programming principles the id team developed in their early years? Well, if you ask Ethan, the downside is that he _can't skip out of his own talk for Romero's_. Ah, well. As they say, them's the breaks.
