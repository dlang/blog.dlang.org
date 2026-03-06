---
author: DBlogAdmin
comments: false
date: 2018-12-10 10:36:25+00:00
layout: post
link: https://dlang.org/blog/2018/12/10/updates-in-d-land/
slug: updates-in-d-land
title: Updates in D Land
wordpress_id: 1842
categories:
- D Foundation
- DConf
- DIPs
- Donations
- News
- SAoC
permalink: /updates-in-d-land/
redirect_from: /2018/12/10/updates-in-d-land/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)As we encroach upon the end of 2018, [a recent Reddit thread wishing D a happy 17th birthday](https://www.reddit.com/r/programming/comments/a47s2x/happy_17th_birthday_d/) reminded me how far the language has come since I first stumbled upon it in 2003. Many of the steps along the way were powered by the energy of users who had little incentive to contribute beyond personal interest. And here we are, all these years later, still moving forward.

There are a number of current and upcoming happenings that will play a role in keeping that progress going. In this post, I'd like to remind you, update you, or inform you about some of them.


## The Pull Request Manager Campaign


If you haven't heard, [the D Language Foundation has hired a pull request manager](https://dlang.org/blog/2018/11/10/the-new-fundraising-campaign/), to be paid out of a pool of donations. This is our first major fundraising campaign through Flipcause. I'm happy to report that it's going well. As I write, we've raised $1,864 of our $3,000 target in 66 days thanks to the kindness of 30 supporters. If you'd like to support us in this cause, click on the campaign card.



You can access our full campaign menu at any time via the "Donate Now" button in the sidebar here on the blog. A pull request has also been submitted to [integrate the menu into dlang.org's donation page](https://dlang.org/foundation/donate.html). Currently, we only have two campaigns (this one and the General Fund) but any future campaigns will be accessible through those menus.


## Symmetry Autumn of Code


Earlier this year, Symmetry Investments partnered with the D Language Foundation to sponsor the Symmetry Autumn of Code (SAoC). Three participants were selected to work on D-related projects over the course of four months, with milestones to mark their progress. If you haven't heard of it or had forgotten, you can [read the details on the SAoC page here at the blog](https://dlang.org/blog/symmetry-autumn-of-code/).

Unfortunately, one participant was unable to continue after the first milestone. The other two, whom we have come to refer to as the two Francescos, have each successfully completed three milestones and are in the home stretch, aiming for that final payment and free trip to DConf 2019.

Francesco Mecca is working on porting an old D1 GC to modern D, and Francesco Galla' is busy adding HTTP/2 support [the vibe-http library](https://github.com/vibe-d/vibe-http). Both have made significant progress and are on track to a successful SAoC. Read more about their projects [in my previous SAoC update](https://dlang.org/blog/2018/09/15/symmetry-autumn-of-code-is-underway/).


## DIP Updates


I've received partial feedback on a decision regarding [DIP 1013 (The Deprecation Process)](https://github.com/dlang/DIPs/blob/1ac8987906528d634312af71bb2b527b93396dd0/DIPs/DIP1013.md) and expect to hear the final verdict soon. As soon as I do, I'll move Manu's [DIP 1016 (ref T accepts r-values)](https://github.com/dlang/DIPs/blob/1ac8987906528d634312af71bb2b527b93396dd0/DIPs/DIP1016.md) into the Formal Assessment stage for Walter and Andrei to consider.

I had intended to move [Razvan's Copy Constructor DIP](https://github.com/dlang/DIPs/pull/129) into Community Review by now, as that is a high priority for Walter and Andrei. However, he's been working out some more details so it's not quite yet ready. So as not to hold up the process any longer, I'll be starting Community Review for [one of the other DIPs in the PR queue](https://github.com/dlang/DIPs/pulls) at the end of this week. When the Copy Constructor DIP is ready, I'll run its review in parallel.


## Google Summer of Code (GSoC) 2019


At the end of last month, [I announced in the forums](https://forum.dlang.org/post/yanpgfffimlauuesqzwy@forum.dlang.org) that we're ramping up for GSoC 2019. [I seeded our Wiki page](https://wiki.dlang.org/GSOC_2019_Ideas) with two potential project ideas to get us started. So far, only one additional idea has been added and no one has contacted me about participating as a student or a mentor.

It's been a while since we were last accepted into GSoC and we'd very much like to get into it this time. To do so, we need more project ideas, students to execute them, and mentors to provide guidance to the students. If you're looking for another way to contribute to the D community, this is a great way to do so. Adding project ideas costs little beyond the time it takes to add the details to the Wiki and, if you are lacking in ideas already dying to escape the confines of your neurocranium, the time it takes to brainstorm something. Student and mentor participation is a more significant commitment, but it's also a lot more rewarding. If you're interested, tell me at [aldacron@gmail.com](mailto:aldacron@gmail.com).


## DConf 2019


Finally, I'm happy to announce... Just kidding. I can't announce anything yet about DConf 2019, but I hope to be able to soon. What I can say with certainty is that in 2019, DConf will be where DConf has never gone before. We're currently working out some details with an eye toward making 2019 a big year for DConf.

I'm really excited about it and eager to let everyone know. I'll do so as soon as I'm able. Watch this space!
