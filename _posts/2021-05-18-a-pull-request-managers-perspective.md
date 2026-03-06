---
author: RazvanNitu
comments: false
date: 2021-05-18 12:53:19+00:00
layout: post
link: https://dlang.org/blog/2021/05/18/a-pull-request-managers-perspective/
slug: a-pull-request-managers-perspective
title: A Pull Request Manager's Perspective
wordpress_id: 2874
categories:
- Community
- D Foundation
permalink: /a-pull-request-managers-perspective/
redirect_from: /2021/05/18/a-pull-request-managers-perspective/
---

![]({{ '/assets/images/a-pull-request-managers-perspective/bug.jpg' | relative_url }})

Since January of this year, I have been working as a part-time PR (Pull Request) manager. During this time, I have mostly been reviewing PRs and going through issues on the D Bugzilla. I have also been trying to come up with ways of creating organizational structures and procedures that will ultimately aid the D leadership in motivating and focusing community effort. This blog post presents a few insights I've had regarding the PR queues of [the dmd, druntime, and phobos repositories](https://github.com/dlang), and a couple of proposals that, in my opinion, could benefit the D contribution process.



### PR rounds



As a PR manager, I spend most of my time reviewing PRs. Since I started on the job, I have been involved in the merger of more than 400 PRs across our repositories. From this experience I have extracted a few insights:





  * If a new PR is not reviewed within the first 3 days after it was opened, chances are that it will get abandoned.


  * If a PR is not merged during the first 2 weeks after it was opened, chances are that it will be abandoned.


  * Contributions, in terms of PRs per month, are as follows: phobos (130), dmd (85), druntime (30).


  * Although phobos benefits from more contributions, dmd has a larger contributor base.


  * Druntime needs morelove.


  * Veteran contributors are more likely to abandon PRs than new/first-time contributors.



Given the first 2 points, I try to make contact as fast as possible with PR authors. It often happens that I do not have the necessary expertise to technically review a PR. In that case, I try to find people who are willing to take a look. However, since we do not have a concrete community hierarchy, it is sometimes difficult to find the needed reviewers. A solution to this problem is proposed later in the blog post.

Regarding the ratio of contributions per repository, it is noteworthy that phobos and dmd get a lot of attention, whereas druntime is by far the least attractive repository. Another interesting aspect is the diversity of the contributor base: in the last month, there have been ten contributors who opened more than one PR for dmd, five for phobos, and four for druntime. Ths emphasizes the fact that druntime needs more love.

Lastly, I noticed that veteran contributors tend to abandon their PRs more often than newcomers. This can be explained by the fact that veteran contributors usually tackle multiple PRs at the same time, whereas newcomers usually focus on a single PR. I want to take this opportunity to urge all contributors not to abandon their PRs. It is disappointing for reviewers such as myself to put in the time to properly investigate the patch and offer advice to then see it go to waste. I know that it is much more appealing to start working on new things, but it is highly important not to let any work go to waste.



### Upcoming projects



From my perspective, D has come a long way from its early days: language features are maturing, adoption is steadily growing and the community is expanding around a nucleus of veteran contributors. But given that growth, it is surprising that from an organizational standpoint we are basically in the same spot: if a critical issue appears (a critical bug report, a CI failure, an expired certificate, etc.), the solution is to make a forum post or a comment on Slack and hope that someone who can fix it, or can get it fixed, notices it soon; non-critical issues depend on someone taking an interest: an issue might eventually be fixed, or we might be stuck with it indefinitely. The problem is not manpower or skill; our community has a lot of talent. Unfortunately, we fail to utilize it to its full potential.

If we want certain things to be done, it is the leadership’s responsibility to:





  1. specifically state what work needs to be done,


  2. organize the community, and


  3. incentivize contributors to do the needed work.



Although there is room for improvment, (1) has usually taken place in the form of forum discussions, DIPs, and blog posts. (2) is difficult to implement, given that people contribute in their own free time. As for (3), the mantra has been “fix it if you need it”, which works well for interesting topics, but not that well for important, hard-to-fix bugs, or high-impact, boring tasks.

Implementing points (2) and (3) in an open-source community and with limited financial resources is difficult. However, there are alternative approaches that have not been explored in the DLang ecosystem. I will outline them below.



#### Creating strike teams



One way of organizing the community is to create dedicated groups of people, or strike teams, that can be called upon for specific tasks. One will be assigned to each repository (dmd, druntime, phobos, dlang.org). The idea is to add people to these groups who either have expertise but lack time to contribute, or don’t lack expertise but are willing to actively contribute. This way, if you do not have time to contribute code, you can still help the community by offering implementation advice, whereas if you do have time to offer, you can contribute and develop expertise. The strike teams will be populated by a limited number of people who are trusted members of the community. These teams will be approached directly by the leadership (Walter, Atila, Mike, PR Managers) to fix issues or implement work defined in point (1). The components of the strike teams will receive recognition by having their name listed on the dlang.org site (thus satisfying point (3)).

Of course, this will work as long as there are folks out there willing to dedicate their time. If you want to contribute in some form to any of the strike teams, please contact me directly on Slack or via email.



#### Bugzilla Gamification



The D compiler has around 3000 reported bugs, druntime around 300, and phobos 900. These numbers have grown over time. Although some issues are fixed, we have had no means to incentivize people to work on the critical ones. To that end, we propose a simple gamification scheme: each issue has a severity associated; once a PR that closes an issue is merged, the github author of the PR is awarded points according to its severity level; a leaderboard, which is updated in real-time, is presented on dlang.org, and anyone can see who the top contributors are. At the end of each “season”, contributors will be awarded prizes and recognition based on different criteria, such as overall point total, number of total contributions, and so on (we have yet to finalize the kinds of prizes that will be awarded).

By implementing this scoring scheme, we offer some incentive for more experienced contributors to prioritize blocker/critical/major/regression issues over the more trivial or simpler ones, and encourage new contributors to try their hand at a level with which they’re comfortable. We are already working on implementing this and will announce the rules and prize categories once everything is up and running.



### Conclusion



We are at a point in the evolution of the D programming language and its ecosystem where motivating community effort towards a common goal is crucial. This is a long-term, complicated task, but we need to start somewhere. I hope that with this initiative we can pave the way to a more sophisticated and better-organized contribution process that is a more satisfying and rewarding experience for our contributors.
