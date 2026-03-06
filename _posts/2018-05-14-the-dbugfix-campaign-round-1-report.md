---
author: DBlogAdmin
comments: false
date: 2018-05-14 15:04:57+00:00
layout: post
link: https://dlang.org/blog/2018/05/14/the-dbugfix-campaign-round-1-report/
slug: the-dbugfix-campaign-round-1-report
title: 'The #dbugfix Campaign: Round 1 Report'
wordpress_id: 1530
categories:
- '#dbugfix'
- Compilers &amp; Tools
- Core Team
permalink: /the-dbugfix-campaign-round-1-report/
redirect_from: /2018/05/14/the-dbugfix-campaign-round-1-report/
---

![](http://dlang.org/blog/wp-content/uploads/2018/02/bug.jpg)

The D Foundation [released version 2.080.0 of DMD](https://forum.dlang.org/thread/pcav4o$1kln$1@digitalmars.com) on May 2nd. That normally would have been accompanied by a blog post highlighting [some of the items from the changelog](https://dlang.org/changelog/2.080.0.html). This time, it should also have included the first update on [the #dbugfix campaign](https://dlang.org/blog/2018/02/03/the-dbugfix-campaign/). I was on vacation with my wife in the days before and after DConf and never got around to writing the post. It’s a bit late to announce the DMD release now, so this post is instead all about the first round of #dbugfix.

I admit, my initial expectations of participation were low. I suppose I didn’t want to be disappointed. Happily, in the end I had no reason to be pessimistic. There were several issues “nominated” on Twitter and in the forums. I was also pleased that some of the forum posts led to substantial discussion. In fact, [at least one of the forum threads](https://forum.dlang.org/post/mailman.1249.1521512907.3374.digitalmars-d@puremagic.com) led to one of the nominated bugs being fixed well before the period ended, and the fix was included in the 2.080 release.


## The results


The following issues were nominated, but were fixed before the nomination period ended. #5227 is the only one I’m certain was fixed as a result of discussion from the #dbugfix campaign. The others might have been, or they may have been fixed in the normal course of activity.



 	
  * [5227 X ^^ FP at compile-time](https://issues.dlang.org/show_bug.cgi?id=5227)

 	
  * [16189 Optimizer bug, with simple test case](https://issues.dlang.org/show_bug.cgi?id=16189)

 	
  * [18562 expression is not evaluated when accessing manifest constant](https://issues.dlang.org/show_bug.cgi?id=18562) – this one was marked as a duplicate of [12486](https://issues.dlang.org/show_bug.cgi?id=12486), but that was closed as `RESOLVED FIXED` on April 8.

 	
  * [18574 Unclear error message when trying to inherit from multiple classes](https://issues.dlang.org/show_bug.cgi?id=18574)


The following issues were open at the time the bug fixers selected which two issues to focus on, along with the “vote” count (+1’s, retweets, or anything which indicated support for fixing the issue) as best as I could determine:

 	
  * [15984 [REG2.071] Interface contracts retrieve garbage instead of parameters](https://issues.dlang.org/show_bug.cgi?id=15984) – 5 votes

 	
  * [15692 Allow struct member initializer everywhere](https://issues.dlang.org/show_bug.cgi?id=15692) – 4 votes

 	
  * [5710 cannot use delegates as parameters to non-global template](https://issues.dlang.org/show_bug.cgi?id=5710) – 3 votes

 	
  * [18068 No file names and line numbers in stack trace](https://issues.dlang.org/show_bug.cgi?id=18068) – 3 votes

 	
  * [1983 Delegates violate const](https://issues.dlang.org/show_bug.cgi?id=1983) – 1 vote

 	
  * [2043 Closure outer variables in nested blocks are not allocated/instantiated correctly: should have multiple instances but only have one.](https://issues.dlang.org/show_bug.cgi?id=2043)

 	
  * [16486 Compiler see template alias like a separate type in template function definition](https://issues.dlang.org/show_bug.cgi?id=16486) – 1 vote (duplicate of [10884](https://issues.dlang.org/show_bug.cgi?id=10884) and [1807](https://issues.dlang.org/show_bug.cgi?id=1807))

 	
  * [17592 destroy isn’t marked @nogc when the class destructor is marked as @nogc](https://issues.dlang.org/show_bug.cgi?id=17592) – 1 vote (duplicate of [15246](https://issues.dlang.org/show_bug.cgi?id=15246))

 	
  * [17957 D shared library throws asserts when called from C detached pthread but not terminated with dlclose](https://issues.dlang.org/show_bug.cgi?id=17957) – 1 vote

 	
  * [18147 Debug information limited in size](https://issues.dlang.org/show_bug.cgi?id=18147) – 1 vote

 	
  * [18353 Unexpected OPTLINK Termination at EIP = 0040F60A](https://issues.dlang.org/show_bug.cgi?id=18353) – 1 vote (duplicate of [17508](https://issues.dlang.org/show_bug.cgi?id=17508))

 	
  * [18493 [betterC] Can’t use aggregated type with postblit](https://issues.dlang.org/show_bug.cgi?id=18493) – 1 vote

 	
  * [18572 AliasSeq default arguments are broken](https://issues.dlang.org/show_bug.cgi?id=18572) – 1 vote


The mandate for the bug fixers is to select at least two of the top five. Of course there has to be some leeway here, given that some issues are extremely difficult to resolve, so they can look beyond the top five if they need to. In this case, given that all of the issues share only four distinct vote counts, the entirety of the list is in play anyway.

I got the bug fixers together before I headed off to my (spectacular) German vacation and asked them to come to a consensus by May 1st on two bugs to fix (I must note that they met their deadline, even though I did not). The two bugs selected were:

 	
  * [15984 [REG2.071] Interface contracts retrieve garbage instead of parameters](https://issues.dlang.org/show_bug.cgi?id=15984)

 	
  * [18068 No file names and line numbers in stack trace](https://issues.dlang.org/show_bug.cgi?id=18068)


There are no time constraints on when these will be fixed. It is hoped that they will be fixed and merged in time for 2.081.0, but they could come in a later release. The point is, each of these is now a subject of attention.

Some details regarding a few of the issues not selected:

 	
  * re: 15692 – a while back, Sebastian Wilzbach [revived an old DIP](https://github.com/dlang/DIPs/pull/71) for in-place struct initialization and is part way through an implementation.

 	
  * re: 1983 – there’s [an old PR waiting for a champion to revive it](https://github.com/dlang/dmd/pull/2130).

 	
  * re: 18493 – Mike Franklin has a PR that’s waiting on clarification about whether or not `nothrow` should be implicit with `-betterC`.




## Keep it going


Now that it’s done, I’m happy to declare the inaugural round of the #dbugfix campaign a success. I had hoped that asking for nominations in the forums would as an alternative to Twitter would spark discussions and am pleased to see that happened. So let’s build on this and keep it going.

The slate was wiped clean and the votes reset from the day I declared the first round finished on April 24th. I haven’t noticed any nominations since then (though I haven’t searched for any yet), but please keep them coming. If there’s an issue on the above list that’s not yet fixed but that you feel strongly about, nominate it again. Or nominate something not on the list that’s stuck in your craw. If you see a nomination on Twitter that you support, retweet it to add your vote. In the forums, give a +1 reply. Keep it going!

And while you’re nominating your issues, keep in mind that fixing bugs is only part of the story. If you really care about seeing issues fixed, please try your hand at reviewing pull requests. Contributors whose fixes languish in the PR queue are demotivated from contributing more, but without more eyes reviewing the PRs, the bandwidth to speed up the reviews isn’t there. More PR reviews beget more merges beget more closed issues beget more motivated contributors and satisfied users.

Barring a change in release schedule, DMD 2.081.0 should be released July 1. This round of nominations will close at the beginning of the last week of June. So tweet out or open a new forum thread for your #dbugfix nominee today. Please include both the #dbugfix hashtag and the bugzilla issue number in the tweet or post title, along with a link to the issue report. That will make my job considerably easier.

And review some PRs!

_Thanks to Sebasitan Wilzbach and Mike Franklin for their help with this, particularly in keeping me updated with info about PR and DIPs!_
