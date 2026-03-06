---
author: DBlogAdmin
comments: false
date: 2019-03-22 13:51:53+00:00
layout: post
link: https://dlang.org/blog/2019/03/22/dconf-2019-london-programme/
slug: dconf-2019-london-programme
title: DConf 2019 London Programme
wordpress_id: 1994
categories:
- DConf
permalink: /dconf-2019-london-programme/
redirect_from: /2019/03/22/dconf-2019-london-programme/
---

The [DConf 2019 schedule](https://dconf.org/2019/schedule/index.html) was published on March 17th. This year we've got a solid mix of first-time DConf speakers and veterans. If you haven't visited the site in a while, you'll surely notice that it's been redesigned. The old version was not responsive and was quite annoying to manipulate on small screens. That has been rectified. And we've also got a new logo appropriate for this year's venue.

![](http://dlang.org/blog/wp-content/uploads/2019/03/dconf-2019-logo2.png)


## Day One - May 8th


It wouldn't be DConf without keynotes from Walter and Andrei. Walter's focus this year is on [memory allocation strategies in D](https://dconf.org/2019/talks/bright.html), the talk which will kick off Day 1. The video of his talk from last year was one of those we lost to a technical error, but the slides [are available for download](https://dconf.org/2018/talks/bright.html). Both slides and video are available for his talks from every previous edition: [2017 in Berlin](https://dconf.org/2017/schedule/), [2016 in Berlin](https://dconf.org/2016/talks/bright.html), [2015 in Utah](https://dconf.org/2015/talks/bright.html), [2014 in Menlo Park](https://dconf.org/2014/talks/bright.html), and [2013 in Menlo Park](https://dconf.org/2013/talks/bright.html).

First-time DConf speaker Jens Mueller follows Walter's keynote with a talk on the approach Dunhumby (formerly Sociomantic Labs) takes for [implementing machine learning](https://dconf.org/2019/talks/mueller.html). Specifically, he'll be covering how they integrated a C library, MXNet, into their D applications.

D bug fixing machine Razvan Nitu is back this year, speaking just before lunch on Day 1. He presented half-hour talks at [DConf 2017 in Berlin](https://dconf.org/2017/talks/nitu.html) and [2018 in Munich](https://dconf.org/2018/talks/nitu.html) related to his D Language Foundation scholarship at Politehnica University of Bucharest. This year, he's doing a full hour on the new [D Copy Constructor](https://dconf.org/2019/talks/nitu.html) from [the recently approved DIP 1018](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1018.md).

Robert Schadek has presented at [DConf 2013 in Menlo Park](https://dconf.org/2013/talks/schadek.html) and [DConf 2016 in Berlin](https://dconf.org/2016/talks/schadek.html). This year he's got the much-coveted post-lunch slot, where he'll tell us all why [spreadsheets must die](https://dconf.org/2019/talks/schadek.html). What's the D connection?


After a brief and humorous look into the capabilities and idiosyncrasies of spreadsheet programming, using D as an alternative will be explored.


The next two hours are filled by three first-time DConf speakers: Guillaume Piolat will show off [his intel-intrinsics library](https://dconf.org/2019/talks/piolat.html), Lionello Lunesu will give a 30-minute demonstration on [packaging D applications](https://dconf.org/2019/talks/lunesu.html), and prolific D contributor Sebastian Wilzbach will use a half-hour slot to explain [how to become a D contributor](https://dconf.org/2019/talks/wilzbach.html).

Walter and Andrei will close out Day 1 with our now traditional [Ask Us Anything session](https://dconf.org/2019/talks/panel.html). They'll take questions from the conference attendees and the livestream viewers. We'll try to take as many as we can, but time is limited. Since it's the last slot of the day, we don't need to worry about fitting in a 10-minute break, so we can go the full hour, but we can't go too far over time.


## Day Two - May 9th


Our invited keynote speaker this year is Laeeth Isharc of Symmetry Investments. Laeeth has contributed to the D ecosystem in multiple ways, including setting up the [2018 Symmetry Autumn of Code](https://dlang.org/blog/symmetry-autumn-of-code/) and in bringing DConf to London. His abstract has not yet been published, but it will be [available on his talk page](https://dconf.org/2019/talks/isharc.html) once it has been.

Mathis Beer will follow Laeeth with his first DConf talk. His employer, Funkwerk, was profiled here on the D Blog in the past as part of our [D in Production Series](https://dlang.org/blog/d-in-production/). He'll be showing us how Funkwerk combines functional programming and OOP in their application design.

Luís Marques has occupied the pre-lunch slot on Day 1 twice: [in 2016](https://dconf.org/2016/talks/marques.html) and [in 2017](https://dconf.org/2017/talks/marques.html). Last year, he was in [the post-lunch slot on Day 2](https://dconf.org/2018/talks/marques.html). This year, he's sending us into lunch again, this time on Day 2, with his ideas on [how to make D's compile-time features even easier to use](https://dconf.org/2019/talks/marques.html).

Átila Neves has given some DConf lightning talks in recent years, but his last full talk was [at DConf 2015](https://dconf.org/2015/talks/neves.html), which followed his first one [at DConf 2014](https://dconf.org/2014/talks/neves.html). He'll be [giving his perspective](https://dconf.org/2019/talks/neves.html) on what motivates programming language adoption, with a look at how C++ succeeded and how D can emulate its success.

Next up are half-hour presentations from two more Politehnica University of Bucharest students. PhD candidate Eduard Staniloiu, who spoke about his work on D collections [in 2017](https://dconf.org/2017/talks/staniloiu.html) and [in 2018](https://dconf.org/2018/talks/staniloiu.html), will this year be [telling us about the details](https://dconf.org/2019/talks/staniloiu.html) of the upcoming `ProtoObject`, which is intended to become the new root of the D class hierarchy to bridge the gap between the old `Object` class and features that have been added to the language over time.

He'll be followed by undergraduate Alexandru Militaru who will [present the results](https://dconf.org/2019/talks/militaru.html) of porting a Linux kernel driver to D to put to the test D's suitability for systems programming and the D motto of _fast code, fast_.

The last full talk of the day will come from Francesco Galla, the winner of the 2018 Symmetry Autumn of Code. For the SAoC, he worked on adding HTTP/2 support to [the vibe-](https://github.com/vibe-d/vibe-http)http[ package](https://github.com/vibe-d/vibe-http). His abstract [is not online](https://github.com/vibe-d/vibe-http) at the moment, but the subject of his talk will be his SAoC project.

We end Day 2 with our traditional round of Lightning Talks. Anyone in attendance is welcome to come to the lectern and regale the crowd with a 5-minute presentation on your D-related topic of interest (sometimes, we may allow a non-D related topic if the potential presenter has a strong elevator pitch). Sign up beforehand or let us know on site. Either way, it's first-come, first-serve. We have 10 slots available. And I have a strong suspicion that the emcee this year will ruthlessly hold speakers to their 5-minute limit!


## Day Three - May 10th


The bulk of our veteran speakers are lined up on Day 3. Andrei will open this last day of talks with the final keynote of the conference. As tradition dictates, he's keeping us in suspense as to the topic. Keep an eye [on his talk page](https://dconf.org/2019/talks/alexandrescu.html) for the abstract.

At DConf 2017, Bastiaan Veelo gave a talk on [using the Pegged parser generator](https://dconf.org/2017/talks/veelo.html), which we learned his company had employed to develop a transcompiler that would help them convert their Extended Pascal codebase to D. This year, he'll be [presenting a followup to that talk](https://dconf.org/2019/talks/veelo.html), detailing the challenges they've faced and the benefits they've seen from some of D's features.

Steven Schveighoffer has become a familiar face at the DConf lectern, having presented [in 2016](https://dconf.org/2016/talks/schveighoffer.html), [in 2017](https://dconf.org/2017/talks/schveighoffer.html), and [in 2018](https://dconf.org/2018/talks/schveighoffer.html). He's back again, this time sending us to lunch with [a talk focused on generative programming](https://dconf.org/2019/talks/schveighoffer.html), using serialization as an example of putting D's powerful compile-time features to work in generating boilerplate code so we programmers don't have to.

Ethan Watson, also a three-time DConf speaker ([in 2016](https://dconf.org/2016/talks/watson.html), [in 2017](https://dconf.org/2017/talks/watson.html), and [in 2018](https://dconf.org/2018/talks/watson.html)),  brings us back from lunch with more compile-time desiderata. He'll be showing [how to push D's compile-time capabilities to the limits](https://dconf.org/2019/talks/watson.html) while still keeping the code easy to maintain and highly efficient.

John Colvin has presented at [DConf 2015](https://dconf.org/2015/talks/colvin.html) and [DConf 2016](https://dconf.org/2016/talks/colvin.html). His talk [is all about ranges](https://dconf.org/2019/talks/colvin.html); not just D ranges, but the concept of ranges, particularly as they were applied in production code at Symmetry and the lessons learned from a heavily range-based DSL.

[LDC veteran](https://github.com/ldc-developers/ldc) and [D book author](https://www.amazon.com/gp/product/178528889X/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=178528889X&linkCode=as2&tag=dlang-20&linkId=DR64TRTOQS2GVORZ) Kai Nacke is another DConf three-timer: [in 2016](https://dconf.org/2016/talks/nacke.html), [in 2017](https://dconf.org/2017/talks/nacke.html), and [in 2018](https://dconf.org/2018/talks/nacke.html). He'll explain how his parser generator, LLTool, [evolved from its conception to its current state](https://dconf.org/2019/talks/nacke.html).

Ali Çehreli is, along with Walter and Andrei, an officer of the D Language Foundation and the author of the excellent 'Programming in D' book, available [for purchase in print](https://www.amazon.com/gp/product/1515074609/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1515074609&linkCode=as2&tag=dlang-20&linkId=PHFTCYVQWVUB6GDB) and freely[ online](http://ddili.org/ders/d.en/index.html). He's also a two-time DConf speaker, in [2013](https://dconf.org/2013/talks/cehreli.html) and [in 2016](http://ddili.org/ders/d.en/index.html). This year he's bringing us an experience report on how he employed D to develop a tool in his job with Mercedes-Benz Research and Development North America.


## Day 4 - May 11th - The Hackathon


The DConf Hackathon returns for its third consecutive iteration. It's not the sort of hackathon most programmers are familiar with, where multiple teams compete to develop a program from scratch, usually over several days. Instead, it's a single day of collaboration on D and its ecosystem.

Participants are free to work on any project they'd like or none at all. Typically, a few people sit down and catch up on the personal projects they've not had time for, some recruiting others to help or provide feedback. Others come together to plan new projects or to work on issues affecting the D ecosystem. Some provide impromptu tutorial sessions on D or specific programming topics. Others take the time to chat.

Each of the two previous Hackathons saw significant work accomplished, including numerous pull requests for outstanding Bugzilla issues in the core D projects and even the implementation of a new D feature (`static foreach`, [as described in DIP 1010](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1010.md), was implemented at the first Hackathon at DConf 2017).

Last year, we started the Hackathon with an extra talk about [WekaIO's open source run-time support library, Mecca](https://dconf.org/2018/talks/shemesh.html). This year we're opening with [an Annual General Meeting (AGM)](https://dconf.org/2019/talks/agm.html). The goal is to discuss and, hopefully, resolve some of the outstanding issues in the D Community and potentially set the stage for some work during the Hackathon.

The plan is to keep the AGM to under two hours so that we can have time for [the morning Hackathon session](https://dconf.org/2019/talks/hackathon1.html). This will give folks a chance to get their game plans set so they can jump right into the code at [the afternoon session](https://dconf.org/2019/talks/hackathon2.html) after lunch.


## The Bennies


As a reminder, we've got the 2nd-floor room [at the Prince Arthur pub](https://www.theprincearthurpub.co.uk/), not far from the venue, each of the first three evenings of the conference, courtesy of Mercedes-Benz Research and Development North America. We'll have a couple of free rounds each night for all who show up.

We've also tentatively scheduled tours with Julian McDonnel [of JoolzGuides.com](https://joolzguides.com/). On May 6th and 7th, he'll be taking those who have signed up on a walking tour from Temple Station to London Bridge. Details on how to participate are provided to all who register. Slots are limited and it's first-come, first-serve.

The 15% Early-Bird registration discount has been extended until March 24th. [Register before it goes away](https://dconf.org/2019/registration.html).

See you in London!
