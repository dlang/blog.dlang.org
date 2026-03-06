---
author: DBlogAdmin
comments: false
date: 2018-09-15 07:44:27+00:00
excerpt: A brief introduction to the projects selected for the Symmetry Autumn of
  Code.
layout: post
link: https://dlang.org/blog/2018/09/15/symmetry-autumn-of-code-is-underway/
slug: symmetry-autumn-of-code-is-underway
title: Symmetry Autumn of Code is Underway
wordpress_id: 1710
categories:
- Community
- Companies
- D Foundation
- SAoC
permalink: /symmetry-autumn-of-code-is-underway/
redirect_from: /2018/09/15/symmetry-autumn-of-code-is-underway/
---

![](http://dlang.org/blog/wp-content/uploads/2018/09/logo-1.png)Earlier this year, Laeeth Isharc brought an idea to the D Foundation for [Symmetry Investments](http://symmetryinvestments.com/) to sponsor a summer of code. He was eager to provide a few motivated individuals the incentive to get some great work done for D and the D community. Sporadic email discussions preceded some chatting [at DConf 2018 Munich](http://dconf.org/2018/index.html) and the idea subsequently began to pick up steam. By the time the details were sketched out, it had transformed into the Symmetry Autumn of Code.


## The SAoC projects


Eight applicants submitted their resumes and project proposals for three slots. Nearly all of the proposals were taken from [the SAoC suggestions page at the D Wiki](https://wiki.dlang.org/SAOC_2018_ideas#HTTP.2F2_for_Vibe.d). Given the limited window, the selection process went fairly quick. Of the three selected, two had mentors attached. It took a little while to find a mentor for the third selection, so we extended the milestone deadline for that participant. Now I can happily say that all three are well underway.


### A fork-based concurrent GC for DRuntime


Francesco Mecca proposed this project. The goal is to take [Leandro Lucarella’s original D1 fork-based GC](http://www.dsource.org/projects/druntime/browser/branches/CDGC/src/gc/gc.d), port it to D2, adapt it to DRuntime’s GC interface, and culminate with a pull request for DRuntime to present the work and open discussion. Leandro agreed to mentor this project and is working with Francesco to develop a test suite that the port must pass as part of Milestone 2. They have also included documentation in their milestone list, which is good news.


### vibe.d HTTP/2 implementation


This one came from Francesco Galla, who is currently pursuing a MSc in Network and Security. The goal here is to enhance [the vibe-http library](https://github.com/vibe-d/vibe-http) to support HTTP/2. Fittingly, Sönke Ludwig, [the maintainer of vibe.d](http://vibed.org/), agreed to mentor the project, but requested someone to share the load due to his schedule. Sebastian Wilzbach stepped up as co-mentor.

This project involves rewriting the current HTTP/1 API, ensuring it works as expected, then incrementally adding support for HTTP/2. Portions of the rewrite were already completed before SAoC came along, but had not yet been tested. As such, testing and bug fixing will be a significant portion of the first milestone.


### Porting the Mago debugger to D


This project was proposed by László Szerémi. He’s a heavy user of debuggers in D and wants to see the situation improved. He believes that [porting the Mago debugger to D](https://github.com/rainers/mago) is a major step in that direction. His first two milestones are concerned with translating, testing, and bug fixing. To cap off the SAoC event, he intends to get a GUI frontend up and running with some basic features.

László had no mentor when he applied, and no one had specifically volunteered to mentor any debugger projects, so we put out a call in the forums and I reached out directly to a few people. In the end, Stefan Koch agreed to take it on.


## SAoC is not the end


I know that one of the goals Laeeth had behind his initial suggestion for this event was to enhance the D ecosystem. None of the selected projects are simple, one-shot tasks. These are projects which will all require attention, care, and effort beyond SAoC. The participants are getting them started and we hope they’ll continue to maintain them for some time to come, but in the end, these are projects for the community. When SAoC 2018 is behind us, it will ultimately be up to the community to determine if the projects live long and prosper or die young.

I’ll post more about SAoC and the participants as the event goes on. We wish them the best in meeting their milestones!
