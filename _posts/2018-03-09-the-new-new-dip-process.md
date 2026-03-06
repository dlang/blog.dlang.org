---
author: DBlogAdmin
comments: false
date: 2018-03-09 12:49:07+00:00
excerpt: When I took on the role of DIP Manager last year, my number one goal was
  to clear out the queue. I made a few revisions to the process and got busy. Over
  the next few months, things went along fairly well, not so much from anything I
  did as from the quality of the submissions. But at some point, things broke down
  and the process stalled.
layout: post
link: https://dlang.org/blog/2018/03/09/the-new-new-dip-process/
slug: the-new-new-dip-process
title: The New New DIP Process
wordpress_id: 1437
categories:
- Community
- Core Team
permalink: /the-new-new-dip-process/
redirect_from: /2018/03/09/the-new-new-dip-process/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)When I took on the role of DIP Manager last year, my number one goal was to clear out the queue. I made a few revisions to the process and got busy. Over the next few months, things went along fairly well, not so much from anything I did as from the quality of the submissions. But at some point, things broke down and the process stalled.

Near the end of the year, Andrei asked me to make two specific changes to the process. One of them was to come up with a different approach for handling the final review. To that end, he suggested I look at how other languages handle the review of their enhancement proposals.

Before I started, I identified some other areas of the process that were problematic so that I could keep an eye out for ideas on how to shore them up. Might as well overhaul the whole process rather than one small part of it. So I put the entire DIP process on hold until the new process was ready to go.

The main thing I learned from looking at other language processes was that about the only thing they have in common is that they use GitHub repositories to store their proposals and that new submissions are made through PRs. Beyond that, there’s a quite a bit of variety in how they handle review and evaluation.

Ultimately, I decided the basic framework we already had was well-suited for our circumstances. It just needed some serious tweaking to iron out the problem spots. So I set out to write up a new procedure document to address the things that needed addressing. When it was done, it took a while to get the seal of approval, but it finally came through and the process is no longer stalled.

So first, before describing the major changes, it will help to understand their motivation.


### What went wrong


One of the earliest stumbles I had as DIP Manager was a mix up over DIP 1006. I had gotten it in my head that the author intended to rewrite it before moving forward. The reality was that I had informed him before DConf that I would get it going at the end of May. The result was that it sat there for several months before I realized my error.

Another problem came with DIP 1009. There was an issue with the way it was written – the style didn’t meet the standard laid out in [the Guidelines document](https://github.com/dlang/DIPs/blob/master/GUIDELINES.md). This led to multiple email exchanges, with massive delays and more misunderstandings, that resulted in the DIP being stuck in limbo for quite a while.

The communication problem over DIP 1009 was what prompted the process revision. For the Final Review of each DIP, I was acting as the middle-man between Walter and Andrei on one side and the DIP Author on the other. It worked well enough as long as little further effort on the DIP was needed, but as soon as there were questions that needed answering or more work to do, it became inefficient, cumbersome, and prone to misunderstanding.

Perhaps the biggest issue of all was time. DIP 1006 sat in Draft Review with nothing from me for several months. DIPs 1006, 1009, and 1011 have been in the Final Review stage for ages. There’s no reason any DIP author should have to wait for months on end with no feedback, or vague promises, no matter which stage of the process a DIP is in. It’s discouraging and demotivating. The process should require some motivation and effort from DIP authors, but it should also require a commitment from the other side to keep the authors informed and to get each DIP through from beginning to end as efficiently as possible.

Some of these problems could have been avoided if I had taken a different view of my role. I saw the role of DIP Manager more as that of a Shepherd than a Gatekeeper. The ultimate fate of a DIP rested on the Author’s shoulders, not mine. The Guidelines were [“more what you call guidelines than actual rules”](https://www.youtube.com/watch?v=b6kgS_AwuH0). After I made my revisions to the Guidelines document early on, they fell right out of my head and I never looked at the document again.


### Righting the wrongs


The [new Procedure document](https://github.com/dlang/DIPs/blob/master/PROCEDURE.md) outlines the new process. Following is a summary of the big ones.

A minor issue is that there was some confusion about the existing review-stage names. There are now four review stages rather than three: Draft Review, Community Review, Final Review, and Formal Assessment. The Draft Review is the same as before. The Community Review is the new name for the old Preliminary Review. The old Final Review, which had two parts, has been split out into the Final Review and the Formal Assessment – the former is the last chance for the community to leave feedback, and the latter is Walter and Andrei’s decision round.

For all but the Draft Review, each stage specifies a maximum amount of time that a DIP can go without progress. For example, a DIP may remain in the `Post-Community Round N` state for 180 days, and a DIP in `Formal Assessment` should receive a final disposition within 30 days. The document defines the steps that must be taken when these deadlines are not met.

Related, though not in the document, is what I will do to keep to the deadlines. I’ll be making use of the calendar in the D Foundation’s Google account to post the start and finish date of each stage for each DIP. When a DIP is between stages, I’ll set milestone dates so that the DIP Author and I can have a clear target to aim for. If we’re on the same page, there will be less opportunity for uncertainty and misunderstanding.

The document provides for a new process for handling the Formal Assessment. No longer will I be a middleman between two email chains. Now, Walter and Andrei will provide their feedback on a private gist, with direct participation by the DIP Author. This should help things move more quickly and will eliminate (or greatly reduce) the chance of anyone (me in particular) causing more delay by getting things mixed up.

Another change is the requirement for a Point of Contact (POC). From here on out, every DIP must have a POC. For a single-author DIP, the DIP Author is the POC. If there are multiple authors, they must select one from among themselves. The need for this came to light after a misunderstanding that arose from the communication problem. The POC must commit to seeing the DIP through to the end of the process. The document outlines what happens to a DIP when the POC becomes unavailable.

Another change that’s not outlined in the document is in how I view my role as DIP Manager. From here on out, I will consider the guidelines as actual rules. I’ll do my best to make sure a DIP meets the standards expected in terms of language and style before it leaves the Draft Review stage. We can tweak it as we go, of course, but never again should a DIP be sent back for revision because it’s too informal.


## Open to refinement


The new Procedure document and the undocumented tweaks to my process are the result of lessons learned over several months. That doesn’t mean they’re perfect. We’ll always be open to suggestions on how to patch up any holes that are identified. Not every change was mentioned above, so please read [the document](https://github.com/dlang/DIPs/blob/master/PROCEDURE.md) for the details.

Hopefully, the three DIPs currently awaiting a final disposition will be resolved before too much longer. After that, [DIP 1012](https://github.com/dlang/DIPs/blob/7c0539b337ad588c68f2f8667f4de2fb23d53295/DIPs/DIP1012.md) will be moved forward for the Final Review and Formal Assessment to become the first DIP to go through the new gist-based review. DIP 1013 (which will likely be the one introducing [binary assignment operators for properties](https://github.com/dlang/DIPs/pull/97)), will be the first test-case for the new process in its entirety. Let’s all keep an eye open for what works and what needs work.

And to everyone, thanks for your patience while I went through my growing pains and we got the mess sorted out. Now that the train is back on the tracks, I’ll do my best to keep it moving.
