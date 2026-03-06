---
author: RazvanNitu
comments: false
date: 2021-09-16 11:43:46+00:00
layout: post
link: https://dlang.org/blog/2021/09/16/bugzilla-reward-system/
slug: bugzilla-reward-system
title: Bugzilla Reward System
wordpress_id: 2989
categories:
- Community
- D Foundation
- News
permalink: /bugzilla-reward-system/
redirect_from: /2021/09/16/bugzilla-reward-system/
---

![Digital Mars D logo]({{ '/assets/images/bugzilla-reward-system/d6.png' | relative_url }})

[The Dlang bot](https://bot.dlang.io/) has been updated to track [Bugzilla issues](https://issues.dlang.org/)
that have been fixed. It went live for testing on the 2nd of July. Each GitHub user who fixes a bug via a merged pull request is awarded a number of points depending on the severity of the issue. The current results can always be seen on [the contributor stats page](https://bot.dlang.io/contributor_stats). This blog post covers all of the details regarding the implementation, rules, and prizes of the reward system.



## Raison d'être



I want to start by saying that the motivation of this system is not to start a fierce competition between contributors to fix as many issues as possible. The primary reasons: we see this as a means for the D Language Foundation to reward committed contributors and to channel their efforts towards more important bugs. If, as a side effect, the system motivates people to fix more bugs, that's great! We won't complain.

There are some negative side effects that are possible with any sort of gamification system, and we'll be keeping an eye out for them. We think we have one of the best online communities out there. Our members are generally friendly and helpful, and we don't want to do anything that causes tension or proves negatively disruptive. We think this will be a fun way to reward our contributors, but we will pull the plug if it proves otherwise.



## Scoring system



The scoring is designed to reward contributors based on the importance of the issues they fix, rather than the total number fixed. As such, issues are awarded points based on severity:





  * **enhancement**: 10


  * **trivial**: 10


  * **minor**: 15


  * **normal**: 20


  * **major**: 50


  * **critical/blocker**: 75


  * **regression**: 100



Of course, the severity of an issue does not necessarily reflect the complexity of the solution. There might be regressions that are trivial to solve, and enhancements that require an extremely complicated fix. The message that we are trying to send is that **complexity is secondary to need**. That is why regressions are given top priority and critical/blocker/major issues aren't far behind.



## Rules



The following rules will guide how points are awarded from the initial launch of the reward system. They are not set in stone and are open to revision over time.

**Rule #1:** The severity of an issue will be decided by the reviewers of a proposed patch.

Severity levels are not always accurately set when issues are first reported and may not have been updated since. The reviewer of a pull request that closes a Bugzilla issue will evaluate the issue's severity level and may change it if he or she determines it is inaccurate. I will moderate any disagreements that may arise about severity levels.

**Rule #2:** A PR fixing a bug may not be merged by the same person that proposed the patch.

This is already an unwritten rule that applies to the DLang repositories, so it should not surprise anyone.

**Rule #3:** Anyone who adopts an orphaned PR that fixes a bug may be awarded its associated points.

To avoid any authorship conflicts, it is best if the adopter contacts the original author to ask if it is okay to adopt the PR. Rule #3 will apply if there is no response or if the response is affirmative. Otherwise, no points will be awarded.

**Rule #4:** Only one person may receive points per fixed issue.

This rule is specifically designed for reverted PRs. Imagine that a PR that presumably fixes an issue is merged and the author gets points for it. Later on, it is decided that the fix is incorrect and the PR is reverted. If someone else proposes the correct fix, the points will be subtracted from the original contributor and awarded to the new author. Hopefully, this will motivate the original contributor to propose the correct fix after the reversion.

**Rule #5:** Incomplete fixes still get points.

A Bugzilla report usually includes a snippet of code that reproduces the issue. A frequent pattern is that the bug is correctly fixed for the provided snippet, then someone comes up with a slightly modified example that does not work and reopens the issue. Since the original fix was correct, but not complete, the procedure here is that the original issue should be left closed and a new one should be opened. The original author keeps the points awarded for the original issue.



## Implementation



Since most of you are die-hard geeks and are eagerly awaiting the code, [here's the database implementation](https://github.com/dlang/dlang-bot/pull/279/files) hosted on the dlang-bot, and [here's the web page implementation](https://github.com/dlang/dlang-bot/pull/282). You will notice that the web page is extremely minimal. That is because I am a total n00b when it comes to web programming, so if anyone has the skills and the time to make a cooler web page, feel free to make a PR :D.

In short, for each of the issues that are fixed, the database stores the Bugzilla issue number, the GitHub ID of the person who fixed it, the date when the fix was merged, and the severity of the issue. Every time [the leaderboard page](https://bot.dlang.io/contributor_stats) is accessed, a query is issued to the database to compute the total points for all of the contributors and sort them in descending order. Easy peasy.

I would like to thank Vladimir Pantaleev for his continued support and assistance throughout the period that I implemented the system.



## Prizes



As Mike briefly described in [this forum post](https://forum.dlang.org/thread/rdqskizblbcdtahlxxsm@forum.dlang.org), we are going to have quarterly competitions. The quarterly prizes will vary. At the end of the year, the person who has acquired the most points will be awarded a bigger prize.

For the inaugural competition, which will officially start on the 20th of September 2021 and will last until the start of DConf Online (the 20th of November), the prizes will be:





  * First Place: a $300 Amazon eGift Card


  * Second Place: a $200 Amazon eGift Card


  * Third Place: a $100 Amazon eGift Card



The next set of prizes will be announced at DConf Online, so stay tuned!



## That's all folks!



If there are any questions or suggestions regarding any aspect of the bugfix reward system, please contact me at [razvan.nitu1305@gmail.com](mailto:razvan.nitu1305@gmail.com). Also, feel free to directly propose changes to the existing infrastructure.

Happy coding everyone!
