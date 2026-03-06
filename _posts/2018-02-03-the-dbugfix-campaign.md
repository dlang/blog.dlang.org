---
author: DBlogAdmin
comments: false
date: 2018-02-03 15:25:45+00:00
excerpt: 'Every major release of DMD comes with a list of closed issues from Bugzilla.
  For example, looking at the changelog for DMD 2.078.0 shows the following counts
  for closed regressions, bugs, and enhancements: 51 for the compiler, 37 for the
  standard library, 6 for the runtime, 17 for the website, and 1 for the linker. That’s
  112 total issues, the majority related to the compiler. The total number of closed
  issues fluctuates between releases, but the compiler and standard library normally
  get the lion’s share.'
layout: post
link: https://dlang.org/blog/2018/02/03/the-dbugfix-campaign/
slug: the-dbugfix-campaign
title: 'The #dbugfix Campaign'
wordpress_id: 1343
categories:
- Community
- Core Team
- News
permalink: /the-dbugfix-campaign/
redirect_from: /2018/02/03/the-dbugfix-campaign/
---

### Why so many bugs?


![](http://dlang.org/blog/wp-content/uploads/2018/02/bug.jpg)Every major release of DMD comes with a list of closed issues [from Bugzilla](https://issues.dlang.org/). For example, looking at [the changelog for DMD 2.078.0](https://dlang.org/changelog/2.078.0.html#bugfix-list) shows the following counts for closed regressions, bugs, and enhancements: 51 for the compiler, 37 for the standard library, 6 for the runtime, 17 for the website, and 1 for the linker. That’s 112 total issues, the majority related to the compiler. The total number of closed issues fluctuates between releases, but the compiler and standard library normally get the lion’s share.

This isn’t news to anyone who regularly follows DMD releases. But spend enough time on the forums and you’ll eventually see someone under the impression that bugs aren’t getting fixed. They cite the number of open issues in the database, or the age of some of the open issues, or the fact that they can’t find any formal process for fixing bugs. In reaction, it’s easy to point to the changelogs, or cite the number of closed issues in the database, or bring up the number of open issues in other language compilers. And, of course, to explain once again that this is a volunteer community where people work on the things that matter to them, and organizing groups to complete specific tasks is like herding cats.

That’s all quite reasonable, but really isn’t going to matter to someone who found the motivation to check D out, but is still looking for the motivation to stay. For me personally, I really don’t care how many issues are in the database, or the age of the oldest. All I care about is that it works for me. But I’m already invested in D. I don’t need to be motivated to stick around. And while I wouldn’t use a bug database as criteria to judge a new language, I can see that others do. It’s akin to looking at a stable repository on GitHub and dismissing it as abandoned because of its lack of recent activity. If you don’t see the whole picture, you can’t make an informed judgement.

If perception were the only issue, then it would simply be a matter of web design and PR. However, there have been, and are, people invested in D who have become frustrated because issues they reported, or that directly affect them, have languished in Bugzilla for months or even years. This can’t simply be dismissed as not seeing the whole picture. This is a matter of manpower and process. A number of issues are still open because there isn't a simple fix, or perhaps because no one has taken an interest. The set of people who can solve complex issues is small. The set of people willing to work on issues that aren't interesting to them is smaller. But again, how do you get a disparate group of volunteers of varying skill levels to devote their free time to fixing other peoples’ problems?

This is something the D community has struggled with as it has grown. There are no easy, comprehensive solutions without a full-time team of dedicated personnel, something we simply don’t have. However, it’s possible that there are opportunities to take baby steps and chip away at some of these issues without the complications inherent in herding cats.


### The #dbugfix campaign


To recap, there are two primary complaints about the D bug-fixing process (such as it is):



 	
  * Too many (old) bugs in the database

 	
  * Bugs you care about aren’t getting fixed


In an effort to alleviate these problems, one baby step to chip away at it, I’m announcing the **#dbugfix** campaign.

It works like this. If there is an issue in [D’s Bugzilla](https://issues.dlang.org/) that you want to see fixed, whatever the reason (maybe it’s blocking you, or it’s annoying you, or it’s an enhancement you want, or you think it’s too old – it doesn’t matter), then either tweet out the issue number with **#dbugfix** in the tweet, or create a topic in [the general forum](https://forum.dlang.org/group/general) with the issue number and **#dbugfix** in the subject line. I’ll monitor both Twitter and the forums and keep a running tally of issue numbers.

A week before a major version of DMD is released (starting with 2.080.0, which is slated for May 1), I’ll look at the tally and find the top five issues. I’ve already gotten people to commit to fixing _at least_ two of the top five. That doesn’t mean _only_ two. It could well be more. It depends on the complexity of the issues and how many other volunteers we can scrounge up. Hopefully, the two (or more) fixed bugs will be ready to be merged in the subsequent major release.

In the blog post announcing each major release, I’ll report on which bugs in the current release were fixed as a result of the campaign and announce the two selected for the subsequent release. If any of the top five from the previous release were not fixed, I’ll call for volunteers to help so that they can be squashed as soon as possible.

Yes, I know. We enabled voting on Bugzilla issues and that didn’t change anything. That’s because there was no real commitment to fixing any of the highest-voted issues. The votes simply served a guideline for the people browsing the database, looking for issues to fix. For this campaign, there really are people _committed_ to fixing at least two of the issues that float to the top for every major release.

But two is not a lot! No, it isn’t. But it also isn’t zero. As I mentioned at the top of this post, dozens of issues are already fixed with each major DMD release. The problem (for those who see it as such) is that there’s currently next to zero community involvement in deciding which issues get fixed. This campaign gives the community more input into the selection process and it provides public updates on the status of that process. It is my hope that, in addition to changing perception and chipping away at the bug count, it encourages more people to help fix bugs.

If you would like to volunteer your time and knowledge to helping out with this campaign and increase the number of **#dbugfix** bugs fixed in each release, please email me at [aldacron@gmail.com](mailto:aldacron@gmail.com). For everyone else, I’ve got a search for **#dbugfix** set up in my Twitter client, so start tweeting!
