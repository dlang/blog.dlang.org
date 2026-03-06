---
author: RonTarrant
comments: false
date: 2019-01-18 14:20:19+00:00
layout: post
link: https://dlang.org/blog/2019/01/18/d-lighted-im-sure/
slug: d-lighted-im-sure
title: D-lighted, I'm Sure
wordpress_id: 1911
categories:
- Community
- Guest Posts
- User Stories
permalink: /d-lighted-im-sure/
redirect_from: /2019/01/18/d-lighted-im-sure/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)For me, finding D is the most recent step along a road winding back at least a dozen years. I’d been searching for a cross-platform development language/environment (POSIX and Windows, but not so much mobile since my search began before mobile was really a thing) and at this point, D fits better than anything else I’ve tried. I won’t go out on a limb and say it’s the Holy Grail of X-Plat, but at the very least, it’s brought some fun back into coding for me. And whenever I massage a hunk of code until it finally works… well, I’m addicted to those little victories.


## The Road to D


The first language I tackled back in the mid-oughts to meet this end was PHP. I’d been a web developer for a few years when I found out that PHP had a standalone desktop version. When I then stumbled across [Andrei Zmievski’s PHP-GTK](gtk.php.net), lightbulbs went on and fireworks went off. For a while. The big drawback I found with PHP-GTK was that no PHP compiler could handle the GTK end. So after a few years of patiently hoping someone would tackle this while I wrote nearly 40 blog posts on its use, I started looking elsewhere.

Back in the 1990’s, I’d been steeped in Javascript and HTML, writing simple online apps for banking clients out of Vancouver, BC and later, Bancroft, Ontario. With such a background, when [I ran across Electron](https://electronjs.org/) a few years ago, it seemed like a good fit. I assumed learning it would be easy-peasy-lemon-squeezy, but a lot about the nature of web development languages had changed. To boot, several more languages, standards, and paradigms had been thrown into the mix, so to say the least, I found it all confusing and more than a little intimidating. What I really wanted was one language, an easy distribution system, and a GUI toolkit that didn’t necessitate balancing style sheets with front- and backend code as well as JSON files. And Electron, unfortunately, needs to drag Chromium along for the ride in a little metaphorical red wagon. It’s a solution, but not the one I was looking for.

Then last year, I stumbled onto D. I’d been hearing about it for years, but I hadn’t read much about it. I didn’t realize it was available across so many POSIX platforms. I also didn’t realize it embraced the OOP paradigm and so hadn’t given it much thought. I liked the subtle humor of D being next in line after B, C and A (which oddly enough came along later than B and C), but after a brief smile, I paid no more attention until last October.

When I finally took a good look, I realized that with its curly braces and OOP propensity, D runs right up my street. But before rolling up my sleeves, I made sure there was a GUI toolkit I could use, something that didn’t necessitate balancing three differing paradigms at once. I found [a list of GUI toolkits](https://wiki.dlang.org/Libraries_and_Frameworks#GUI_Libraries) on the D Wiki and was gratified [to see GtkD among them](https://gtkd.org/). So for the last two plus months, I’ve been putting most of my effort into learning how D and GtkD work together.


## Perspiration and Refreshments


It may or may not be important to know this, but I don’t have a CS degree. I’m completely self-taught, a process that started while stuck in the middle of a frozen nowhere for three weeks more than 30 years ago. But that’s a whole other story. My point is, there are holes in my education. That’s what happens when you follow your nose instead of a syllabus.

Because of that, some of the intricacies of D elude me and may always do so. Although I’ve read Ali Çehreli’s chapter on the subject ([from Programming in D](https://ddili.org/ders/d.en/index.html)) I have no idea what mixins are or why I’d want to use them. And templates seem like a good idea, but I don’t know why. I blame my lack of formal CS education for this, but I’m quite comfortable with classes and objects, so I’m not sweating it.

I was first introduced to OOP and the Gang of Four when I was learning PHP, so D covers familiar territory in that regard. Curly braces are another thing I feel quite at home with, having used them for most of my programming life.

But I’m finding the familiarity of D to be a bit of a stumbling block as well. It’s just different enough from C and PHP to mean I have to work hard at pounding those differences into my brain. I deal with it through rote typing of example code. I figure if I copy out enough D code instead of lazing along with copy-n-paste, eventually I’ll push C and PHP far enough into the background that I can see past them. And I [keep Mr. Çehreli’s book handy](https://gumroad.com/l/PinD) so I don’t go completely off the rails. It’s been quite helpful. And speaking of helpful…


## The Ecology of D




### Forums


So far, I’ve signed up for two D-oriented forums, [one at dlang.org](https://forum.dlang.org/) and [the other at gtkd.org](https://forum.gtkd.org/). I have yet to find an unfriendly avatar. Everyone I’ve encountered seems willing to jump in and help. To contrast these forums with one I’ve been active on for a few years (not mentioning names, but this other forum is related to art and graphics, not programming) on the D forums I haven’t been insulted, nor have I been questioned for looking at things from a (warning: film term) Dutch angle---which is one of the things I do best. That comes from my art background, I suspect. I was quite the rabble rouser in art college. (Just ask Alan Wood the installation artist about our knock-down-drag-out shouting match in the cafeteria of Emily Carr College if you don’t believe me. But again, that’s a whole other story.)

On dlang.org, I mostly hang in [the Learn sub-forum](https://forum.dlang.org/group/learn), which I suppose is only natural at this stage. It’s probably the most polite bunch I’ve run across on a forum ever, and I’ve been frequenting forums since the hoary days of the BBS when 1200 BAUD was lauded as the fastest thing since the 427 hemi.

Mike Wey seems quite patient for someone who answers more-or-less the same question over and over again on the GtkD forum. The only negative I found with that forum was technical. I signed up, made some posts, and when I went back to sign in a second time, I had to reregister. But I was still identified as the same Ron Tarrant who signed up the first time (I think) so perhaps that’s how things are supposed to be. It’s unusual, but it works.

I will also mention that the GtkD documentation pages are a mind-bender, but because this is where I’m planning to spend a lot of time over the next while, I’ve decided to pitch in and help make things more accessible.

I’m drawing on experiences writing my PHP-GTK blog back in 2006 and porting a bunch of code examples and tutorials into GtkD. I went so far as to buy a domain name ([gtkDcoding.com](http://gtkdcoding.com/)) in preparation for launching a site and blog covering all this. That’s how I deal with learning curves and have done since I wrote that series of BASIC tutorials for my sister while freezing in a Newfoundland outport back in 1985.


### Tools


I got up and running with dmd quite quickly. Installation on Windows 10 and FreeBSD was straightforward. A few quick questions on the D forum, and I had everything I needed to do single- or multi-file projects. A few more questions answered on the GtkD forum and I was comfortable enough to start porting my PHP-GTK code.

But I have to say, [DUB eludes me](https://dub.pm/getting_started). I don’t know if it’s because Electron left me feeling like JSON files are a disgruntled engineer's revenge plot or if it’s just the way my brain works. Perhaps if I put my mind to it, I could learn it, but since I’m getting the results I want from [plain ole dmd](https://dlang.org/download.html), I’ve done no more than skim DUB’s docs up until now. Just for the record, back in my C days, make files were a caution for me, too. I eventually licked them, and if it ever becomes important enough to me to figure out DUB, I guess I will. But for now, my heart’s not in it.

Finding a text editor that supports D (and especially GtkD) syntax highlighting rather than—as a few people on the forum stated—supporting C and getting ‘good enough’ support for D, led me to abandon the search and roll my own. So far, I’ve done more-or-less complete highlighters for PSPad and CodeBlocks. To be fair, they both support D out of the box, but not GtkD which is important to me, mostly as an aid to remembering module names.

I haven’t even looked at other tools. I’m dimly aware of some other sub-forums for what look like other tools, but to be honest I haven’t read them. As I said, dmd does the job, so I’m satisfied for now.


### Conclusion


And that’s a quick summary of my first two months with D. On the one hand, I don’t really understand some of D’s paradigms, but on the other, the ones I do relate to are meeting my expectations. The only two things I haven’t found yet are:



 	
  * information about how I would go about packaging a D app (with GtkD) for distribution, and

 	
  * how to build on one platform for distribution on another (if that’s even possible).


I’m willing to put in some time on this and eventually get gtkDcoding.com off the ground as my way of giving back. It’s been a long time since those frozen weeks [with a Coleco Adam](https://en.wikipedia.org/wiki/Coleco_Adam) when I tried to explain BASIC to my thirteen-year-old sister (who is now a paramedic and doesn’t care any more). And I must say, I’m as excited now about D and GtkD as I was about BASIC and general computing way back then. Being retired, I have the time to pursue it and I’m looking forward to becoming a regular member of the D community.

And in case you don’t know what a Dutch angle is, I urge you to watch some episodes of Batman from the 1960’s. See how the camera tilts when things go wrong for our heroes? [That’s a Dutch angle](https://en.wikipedia.org/wiki/Dutch_angle).


## Bio


I was inspired to learn programming while vacationing in a frozen Newfoundland outport in April of 1985. It started as a desperate attempt to keep from stripping down to my shorts and disappearing into a blizzard, but became a lifelong passion along with acting, writing, and music. In keeping with this non-typical start, it was in art college where I learned my first serious programming language (6502 assembly) and later it was on a job as a technical writer and artist that I wrote my first serious code for a client, an online mortgage calculator for a credit union in British Columbia. The culmination of my programming career was finishing the PHP-GTK app, Corkboard, a writer’s tool for story planning. (Don’t bother looking. I couldn’t come up with a distribution system, so it languishes here on a back-up drive.)

Since dropping out of high school in 1972, I’ve made a living as a taxi driver, musician, screenwriter, technical writer, artist, sound reinforcement equipment salesman, and biology lab technician among other things. I’ve also made money acting, programming, and promoting concerts. I retired from Statistics Canada in 2010 and have since divided my time between acting, writing lame novels, pursuing the elusive X-Plat beast, and keeping house for my wife of 33 years.
