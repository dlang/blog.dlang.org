---
author: DBlogAdmin
comments: false
date: 2020-08-30 14:05:33+00:00
layout: post
link: https://dlang.org/blog/2020/08/30/symmetry-investments-and-the-d-language-foundation-are-hiring/
slug: symmetry-investments-and-the-d-language-foundation-are-hiring
title: Symmetry Investments and the D Language Foundation are Hiring
wordpress_id: 2704
categories:
- Community
- Companies
- D Foundation
- Jobs
- News
permalink: /symmetry-investments-and-the-d-language-foundation-are-hiring/
redirect_from: /2020/08/30/symmetry-investments-and-the-d-language-foundation-are-hiring/
---

![Digital Mars D logo]({{ '/assets/images/symmetry-investments-and-the-d-language-foundation-are-hiring/d6.png' | relative_url }})

The D Language Foundation is hiring! Thanks to generous funding from Symmetry Investments, we are looking to fill two (mostly) non-programming positions geared toward improving the D ecosystem. Symmetry is also offering a bounty for a specific improvement to [DUB, the D build tool and package manager](https://code.dlang.org/). And on top of all of that, [they are hiring D programmers](https://news.ycombinator.com/item?id=24048006).





## D Pull Request/Issue Manager





A lot of good work goes into [the D Programming Language GitHub repositories](https://github.com/dlang). Unfortunately, some of that good work sometimes gets left behind. A similar story can be told for [our Bugzilla database](https://issues.dlang.org/), where some issues are fixed almost as soon as they're reported and others fall victim to a lack of attention. Efforts have been made in the past to tidy things up, but without someone in a position to permanently keep at it, it's a task that is never complete.





The D Language Foundation is looking for one or two motivated individuals to take on that permanent position, get the work done, and keep things running smoothly. Symmetry Investements is generously funding this role with $50,000 per year for one person, or $25,000 per year for each of two.





The ideal candidate is someone who:







  * is familiar with git, GitHub, and Bugzilla;


  * is familiar enough with D to be able to review simple pull requests;


  * is able to recognize when more specialized reviews are required and


  * is able to proofread English text (for reviewing documentation and web site pull requests).





Examples of the role's responsibilities include:







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





We are hoping to hire from within the D community, though we will accept queries from anyone. If you are interested in taking on the role, please send your resume to [social@dlang.org](mailto:social@dlang.org). You should also indicate if you are willing to do the job full time (just you) or part time (share the responsibilities with someone else).





## Community Relations Assistant





I've been working with the D Language Foundation for the past three years. Much of what I do falls loosely in the category of Community Relations. These days, I'm in need of an assistant. Symmetry Investments is providing $600 per month for the role.





The job will involve a number of different activities as the need arises, such as:







  * seeking out guest authors and projects to highlight for the D Blog;


  * monitoring our social media accounts;


  * sending out messages from the D Language Foundation (such as thank you notes to new donors);


  * assisting with maintenance of pages at dlang.org and dconf.org;


  * assisting with the organization of events like DConf and SAOC and


  * any odd jobs that pop up now and again.





If you have good communication skills, an optimistic disposition, and enthusiasm for the D Programming Language, I'd like to talk to you. I don't need a resume. Instead, please send an email to [social@dlang.org](mailto:social@dlang.org) explaining why you're the right person for the job.





## DUB Bounty





![Symmetry Investments logo]({{ '/assets/images/symmetry-investments-and-the-d-language-foundation-are-hiring/logo-1.png' | relative_url }})DUB has become a critical component in the D ecosystem. A significant number of projects depend on it and we need it to be able to meet a wide range of project needs. To that end, there are certainly improvements to be made. One such is in how DUB determines which of a project's source files are in need of recompilation. Currently, DUB follows in the tradition of the venerable `make` and uses timestamp comparisons to make that determination.





A new generation of version control and build tools (git, buck, bazel, scons, waf, plz, and more) rely on file checksums to assess the need for action. This is a much more robust approach because it detects actual changes in file content. Timestamps can change in any number of irrelevant ways. Robustness is important if one is to depend on a build working properly even when files are moved, copied, and shared across people, machines, and teams. As hashes are fast to compute on modern hardware, the impact on speed is very low.





Symmetry Investments is offering a $2,000 bounty to the programmer who either converts DUB's use of timestamp-dependent builds to use SHA-1 hashing throughout, or implements it as a global option to preserve the current behavior.





For inspiration, see [this clip from Linus Torvald's Google talk](https://www.youtube.com/watch?v=4XpnKHJAok8&t=56m16s), and the article [Build-Systems Should Use Hashes Over Timestamps](https://medium.com/@buckaroo.pm/build-systems-should-use-hashes-over-timestamps-54d09f6f2c4). Note that `shasum $(git ls-files)` in Phobos takes 0.05 seconds on a warm SSD drive in a desktop machine.





Anyone interested in taking on this bounty should contact social@dlang.org beforehand. Anyone interested in contributing to the bounty amount can do so via the bounty card [Support for Hash-Based Recompilation in DUB](https://www.flipcause.com/secure/cause_pdetails/OTE5MjY=) at [our Task Bounties page](https://www.flipcause.com/secure/cause_pdetails/NjI2Njg=).
