---
author: DBlogAdmin
comments: false
date: 2022-01-05 11:03:28+00:00
layout: post
link: https://dlang.org/blog/2022/01/05/new-year-dlang-news-hello-2022/
slug: new-year-dlang-news-hello-2022
title: 'New Year DLang News: Hello 2022'
wordpress_id: 3028
categories:
- Community
- D Foundation
- News
- SAoC
permalink: /new-year-dlang-news-hello-2022/
redirect_from: /2022/01/05/new-year-dlang-news-hello-2022/
---

![Digital Mars D logo]({{ '/assets/images/new-year-dlang-news-hello-2022/d6.png' | relative_url }})

For many people around the world, 2021 is a year they'd like to forget. The ongoing pandemic has touched all of our lives indirectly, but for too many, including some in the D community, it has had a more direct impact. We wish a full recovery for those of you who have been physically or emotionally affected by the virus. Please don't forget: the D community is a network of people located around the globe. We are linked by our interest in the D programming language, but we are people before we are D programmers. If you find yourself in circumstances that disrupt any commitments you have in the community, it's nothing to fret over. Get it sorted and we'll be here when you get back. And if you need help to get it sorted, there are many among us willing to help if they can. Don't be afraid to reach out.

Collectively, 2021 was a pretty good year for D. Some highlights:





  * Symmetry Investments sponsored three paid positions for the D Language Foundation;


  * two past Symmetry Autumn of Code (SAOC) projects bore fruit ([forking GC merged into DRuntime](https://dlang.org/changelog/2.098.0.html#forkgc), [D backend for Bison released](https://savannah.gnu.org/forum/forum.php?forum_id=10047));


  * significant progress was made on SAOC 2021 projects that will bare fruit down the road;


  * [ImportC came into existence](https://youtu.be/c3kJoFCzA-0) and has continued to improve;


  * all of the D compilers [were ported to OpenBSD](https://youtu.be/4Uu96MEoHqk);


  * most of the work to update GDC to the D version of the D frontend has been completed;


  * DLL support in LDC made a large leap forward, and progress has been made on broader linker support;


  * our new "strike teams" quietly fixed a number of Bugzilla issues;


  * [we slimmed down our Pull Request queues](https://dlang.org/blog/2021/10/29/dlang-news-september-october-2021-d-2-098-0-openbsd-saoc-dconf-online-swag/);


  * the active population on [the community-managed D Community Discord server](https://discord.gg/bMZk9Q4) saw notable growth and


  * a whole lot more.



A small amount of the work done in 2021 was paid for. The rest was carried out by volunteers, without whom the D programming language would not be where it is today. On behalf of the D Language Foundation, thanks again to all of our contributors, large and small, for all that you do.

Now for some updates to lead us into 2022.



## We're hiring



Symmetry Investments has informed us that they will continue sponsoring the three positions they started sponsoring last year. Razvan Nitu will continue in his role as a Pull Request Manager, and Max Haughton will go on as a general purpose assistant. The second Pull Request Manager role is currently vacant. We are looking for someone to fill it.

The position pays $25,000 USD per year. The ideal candidate is someone who:





  * is familiar with git, GitHub, and Bugzilla;


  * is familiar enough with D to be able to review simple pull requests;


  * is able to recognize when more specialized reviews are required and


  * is able to proofread English text (for reviewing documentation and web site pull requests).



The person who fills the position will work closely with Razvan Nitu. Examples of the role's responsibilities include:



  * ensuring all pull requests follow procedure;


  * reviewing simple pull requests;


  * finding appropriate reviewers for more complex pull requests;


  * ensuring that pull requests are reviewed in a timely manner;


  * reviving stale pull requests;


  * coordinating between pull request submitters and reviewers to prevent pull requests from going stale;


  * closing pull requests that are no longer valid;


  * identifying Bugzilla issues that are duplicates or invalid;


  * identifying Bugzilla issues that are candidates for bounties;


  * publicizing Bugzilla issues in need of a champion and


  * other related tasks.



We are hoping to hire from within the D community, though we will accept queries from anyone. If you are interested in taking on the role, please send your resume to [social@dlang.org](mailto:social@dlang.org).



## Symmetry Investments is hiring



Symmetry Investments is looking for people to fill a number of roles. [Their monthly job announcement at HackerNews](https://news.ycombinator.com/item?id=29405056&p=4#29521339) lists those roles along with qualifications, details on how to apply, and more. If you think you don't qualify because you lack a degree or haven't built up a history of experience, please pay special attention to the following lines from the job announcement:




We look for virtues and capabilities over only experience and credentials although those things aren't a disadvantage. Do not let a lack of credentials or qualifications prevent you from applying.




They are hiring for full-time, fixed-term contracts with flexible hours, with the possibility for both remote work and sponsorship for a visa in London, Hong Kong, Singapore, or Jersey.



## Symmetry Autumn of Code 2021



Milestone 4 of SAOC 2021 kicked off on December 15th. As this point, only two participants remain eligible for the final Milestone 4 reward, but four of [the original five projects](https://dlang.org/blog/2021/08/30/saoc-2021-projects/) are on the road to completion.





  * **Replace DRuntime hooks with templates** - Teodor Dutu has been steadily making progress on his project and has faced some tough challenges along the way. He successfully completed Milestones 1 - 3 and is continuing the project through Milestone 4.


  * **Implement support for D in LLVM Debugger (LLDB)** - Luís Ferreira has also faced some hard problems in passing Milestones 1 - 3 and continues his work as well. One major step in his progress: he has been granted commit access to LLVM and is now part of the team that reviews, accepts, and merges D-related code into the LLVM tree.


  * **Rethinking the default class hierarchy** - [Robert Aron submitted a DIP](https://github.com/dlang/DIPs/pull/219) for the `ProtoObject` at the end of Milestone 1. Unfortunately, he was unable to complete SAOC Milestone 3, but we will launch the first round of Community Review for the DIP in mid-January.


  * **Light Weight DRuntime (LWDR)** - Dylan Graham had to withdraw from the SAOC event after Milestone 2. However, his LWDR is a passion project that existed prior to SAOC and will still be there after the event ends. He intends to pick up the project again when he is able. We wish him the best and look forward to his future work.


  * **Improve DUB: solve dependency hell** - Ahmet Sait Koçak picked this project from the community-maintained [DLang Project Idea repository](https://github.com/dlang/projects). The SAOC judges had concerns about the proposed solution, so before accepting it for SAOC 2021, we discussed the project at [the D Language Foundation's monthly meeting in August](https://forum.dlang.org/thread/rdqskizblbcdtahlxxsm@forum.dlang.org). The final decision was to accept the project, but that Ahmet should explore a specific alternative and only attempt his proposed solution if that was not viable. The alternative proved a dead end, so he moved forward on his initial proposal. He was able to make progress until he encountered issues which will likely require work beyond the scope of the project to resolve. As such, he will be unable to complete the event. Future work on solving the DUB dependency hell problem may well need to take a different approach.





## DConf Online 2021 Q & A videos



To date as I write, I have published six of the eight Q & A videos that I cut and trimmed down from the Day One and Day Two livestreams. I'll have the remaining two published, along with the 'Ask Us Anything!' session with Walter, Atila, and Razvan, before the middle of January. All of the Q & A videos are available on the [DConf Online 2021 Q & A playlist](https://www.youtube.com/watch?v=g26eJcs2QB0&list=PLIldXzSkPUXVSD9mrF1rHk2G-6AzdddPk) and links are available in [the description of each talk at dconf.org](https://dconf.org/2021/online/index.html). The AUA will be listed on [the DConf Online 2021 playlist](https://www.youtube.com/playlist?list=PLIldXzSkPUXXA0Ephsge-DJEY8o7aZMsR) and linked from [its description in the DConf Online 2021 schedule](https://dconf.org/2021/online/index.html#aua).

On a related note, we're all itching to get the real-world DConf going again. We're currently evaluating the possibility of doing so later this year and what it will look like if it happens. Stay tuned.



## Onward and upward!



We've got a number of things going on for 2022. Some examples: I'll be publishing a tutorial series on our YouTube channel; we'll finally publish a new vision document; we'll be taking the first steps toward bringing the services in our ecosystem under one roof with multiple admins; we'll either give Bugzilla an overhaul or port our issues to GitHub; we'll finally have an implementation of the named arguments DIP; and more.

We are always in need of contributors. There are several ways to contribute:





  * If you're working on your own D project, please contact me to write about it on this blog. Or write about it on your own blog. Or tweet about it. Let the world know what you're doing! D exists and people are using it, so we need to be shouting out loud so that more people know about it.


  * If you find an issue, please report it. If there's an issue you can solve, please submit a PR. If you're interested in solving multiple issues, please contact Razvan Nitu about joining one of his strike teams.


  * If you don't have time to solve issues, please consider supporting us financially by [posting a bounty on any issues you care about, or donating to one of our funds](https://dlang.org/foundation/donate.html). Or maybe support us by buying swag at the DLang Swag Emporium using [the link in the sidebar](https://www.zazzle.com/store/dlang_swag?rf=238129799288374326) so that we get a referral bonus on top of royalties. Or perhaps select the D Language Foundation as [your preferred charity at smile.amazon.com](https://smile.amazon.com/) so that we get a small percentage of your purchase amount when you shop there. (The D Language Foundation is only available as an option through Amazon's .com domain.)


  * One of the most impactful ways you can contribute is to help newcomers to the D programming language. Hang out on [the D Community Discord server](https://discord.gg/bMZk9Q4) or [in the D Forums](https://forum.dlang.org/) and employ the knowledge you've gained about D in helping others solve their problems. Help us in continuing to grow one of the most helpful communities on the internet.



Together, we can make 2022 a great year for our favorite programming language.

Happy New Year!
