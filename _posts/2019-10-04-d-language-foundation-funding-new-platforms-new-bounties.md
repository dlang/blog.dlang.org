---
author: DBlogAdmin
comments: false
date: 2019-10-04 10:07:00+00:00
layout: post
link: https://dlang.org/blog/2019/10/04/d-language-foundation-funding-new-platforms-new-bounties/
slug: d-language-foundation-funding-new-platforms-new-bounties
title: 'D Language Foundation Funding: New Platforms, New Bounties'
wordpress_id: 2213
categories:
- '#dbugfix'
- D Foundation
- Donations
- News
permalink: /d-language-foundation-funding-new-platforms-new-bounties/
redirect_from: /2019/10/04/d-language-foundation-funding-new-platforms-new-bounties/
---

![Digital Mars logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)When I first announced [the HR Fund](https://www.flipcause.com/secure/cause_pdetails/NTUxOTc=) here [on the blog back in April](https://dlang.org/blog/2019/04/15/manpower-in-the-d-ecosystem-or-resources-resources-resources/), there was talk among the D Language Foundation team of hiring one or more people to flesh out the specification and implementation of `shared`. That sort of work requires a very specific skillset that only a few people in the orbit of D possess. So far, we've been unable to find any of them with the time to spare. Meanwhile, the HR Fund is sitting there, waiting to be used.


### Mobile Support for LDC


A few weeks ago, Ethan Watson wrote a post in the D forums titled, [DMD or LDC on mobile](https://forum.dlang.org/post/rzgqgowgxxxhkttndrcc@forum.dlang.org). That thread, followed up by emails with Ethan and a few other people, presented a great opportunity to start putting the HR fund to use. Given that LDC already has support for ARM and DMD does not, it's more practical to fund efforts on LDC than on DMD

As Adam Ruppe has suggested in the forums, he is currently working under contract to complete the existing work on Android support for LDC. By the time he's finished, it should be possible for anyone to build a D application for Android and distribute it through the Play Store.

The iOS story, unfortunately, hasn't yet moved forward. We had the ideal candidate on board and eager to get started, but he was sadly unable to get the time off from work that he would need to get the job done. We've asked around, looking for someone else with the same skillset to take on the task, but have come up empty. So now we're reaching to the community at large. But with a twist...


### New Task Bounties


Back in August, I announced that [we had launched a new Bug Bounty system](https://dlang.org/blog/2019/08/17/bug-bounties-have-arrived/). The term "Bug" was perhaps too restrictive, so [I've renamed the menu to Task Bounties](https://www.flipcause.com/secure/cause_pdetails/NjI2Njg=). And as of today the D Language Foundation has seeded three new bounties: two for Bugzilla issues and one for the aforementioned LDC project.

[![Donate to the campaign for adding iOS/iPadOS support to LDC.](http://dlang.org/blog/wp-content/uploads/2019/10/Screenshot_2019-10-04-D-Task-Bounties.png)](https://www.flipcause.com/secure/cause_pdetails/NjU3MTY=)

The D Language Foundation has put forward $3000 to seed the bounty to add iOS and iPadOS support to LDC. We encourage anyone interested in seeing this task complete to donate to increase the bounty.

This isn't a typical bounty, as the money will only be paid as the result of contract work. As such, the money to seed it comes from the HR fund. So if you're interested in taking the bounty home, click the image above and read the bounty description. We want to get this completed as soon as possible, else we'd wait for our original candidate to become available. So if you have the requisite knowledge, skills, and abilities to get the job done, please don't hesitate to reach out.

The Foundation seeded two Bugzilla bounties (from the General Fund) at $50 each: one for [issue #18472 (betterC: cannot use format at compile time)](https://www.flipcause.com/secure/cause_pdetails/NjU3MTQ=) and the other for [issue #18062 (](https://www.flipcause.com/secure/cause_pdetails/NjU3MTU=)ddoc[: Generated .html files should retain the package hierarchy](https://www.flipcause.com/secure/cause_pdetails/NjU3MTU=)). Click through those links to increase the bounties, or visit [Bugzilla #18472](https://issues.dlang.org/show_bug.cgi?id=18472) or [Bugzilla #18062](https://issues.dlang.org/show_bug.cgi?id=18062) for the bug details to get started on fixing one.

I'd like to thank the members of the dlang-jp community for bringing these bugs to our attention. I recently met three of them in Tokyo along with Átila Neves. Aside from having a great time hanging out and [touring part of Asakusa](https://www.japan-guide.com/e/e3004.html), we had a good chat about D and the Japanese D community. I look forward to the next opportunity to see them.

We'll be seeding more Bugzilla bounties in the coming weeks. I'll be digging into some of the old `#dbugfix` issues that are still open. If you have a bug that's particularly troubling you, please [consider seeding a bounty for it yourself](https://www.flipcause.com/secure/cause_pdetails/NjI1ODA=). Alternatively, post a link to it on Twitter with the `#dbugfix` hashtag and we'll consider the possibility of seeding a bounty with Foundation money.

Please [visit the Task Bounties page](https://www.flipcause.com/secure/cause_pdetails/NjI2Njg=) to see if anything else there strikes your fancy!


### The HR Fund Status


The HR Fund currently sits at $16,345. We're about to lose some of it to Adam for his work on Android and (hopefully) more to someone who takes on the iOS task. Currently, we're looking into other opportunities to put some of it to use. We still have dreams of funding major work, so we need to continue to make the HR Fund grow.

You can help us by donating to [the HR Fund campaign directly](https://www.flipcause.com/secure/cause_pdetails/NTUxOTc=), or by using our special $60 campaign: [donate $60 to the HR Fund and get a DConf 2019 t-shirt](https://www.flipcause.com/secure/cause_pdetails/NTg4NzE=). We still have several shirts available, scattered throughout the world, so please take one off of someone's hands!


### AmazonSmile


In a previous post, I mentioned the AmazonSmile plugins [Smile Always for Chrome](https://chrome.google.com/webstore/detail/smile-always/jgpmhnmjbhgkhpbgelalfpplebgfjmbf?hl=en) and [Smart Amazon Smile for Firefox](https://addons.mozilla.org/en-US/firefox/addon/smart-amazon-smile/?src=search) as easy ways to support the D Language Foundation. These plugins ensure that every time you visit amazon.com you will be sent to [smile.amazon.com](https://smile.amazon.com/) instead to support your selected charity. If its the D Language Foundation, we get 0.5% of every eligible purchase you make (and sorry to the international folks, but the D Language Foundation is only available as a charity through the .com domain).

Now, you can also support the D Language Foundation through the Amazon Shopping App for Android. [Visit the AmazonSmile Mobile page](https://smile.amazon.com/b?node=15576745011&ref_=pe_732550_434054570) to see how.
