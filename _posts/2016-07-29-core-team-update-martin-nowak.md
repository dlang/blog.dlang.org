---
author: DBlogAdmin
comments: false
date: 2016-07-29 12:00:56+00:00
layout: post
link: https://dlang.org/blog/2016/07/29/core-team-update-martin-nowak/
slug: core-team-update-martin-nowak
title: 'Core Team Update: Martin Nowak'
wordpress_id: 131
categories:
- Compilers &amp; Tools
- Core Team
permalink: /core-team-update-martin-nowak/
redirect_from: /2016/07/29/core-team-update-martin-nowak/
---

In the early days of [DMD](https://dlang.org/download.html), new releases were put out when Walter decided they were ready. There was no formal process, no commonly accepted set of criteria that defined the point at which a new compiler version was due or the steps involved in building the release packages. In the early days, that was just fine. As time passed, not so much.

According to Martin Nowak:


The old release process was completely opaque, inconsistent, irregular, and also very time-consuming for Walter. So at some point it more or less failed and Walter could no longer manage to build the next release.


Martin was eager to do something about it, but he wasn't the first person to take action on the issue. He decided to start with the work that had come before.


At this point, I took a fairly complex script from Nick Sabalausky, which tried to emulate Walter's process, and started to improve upon it. It [got trimmed down](https://github.com/dlang/installer/commits/77f4523692c50b75f794f198dbdb43c573cf0e61/create_dmd_release/create_dmd_release.d) to a much smaller size over time.


But that was just the beginning.


Next, I prepared OS images for [Vagrant](https://www.vagrantup.com/) for all supported platforms. That alone took more than a week. After that, I wired up the release script with Vagrant. From there on we had kind of [reproducible builds](https://github.com/dlang/installer/commits/77f4523692c50b75f794f198dbdb43c573cf0e61/create_dmd_release/build_all.d).


That was a major step forward and eliminated some of the common mistakes that had crept in now and again with the previous, no-so-reproducible, process. With the pieces in place, Martin got some help from another community member.


Andrew Edwards took over the actual release building, but I was still doing the main work of keeping the scripts running and managing bugfixes. At some point, Andrew no longer had time. As the back and forth between us was almost as much work as the release build process itself, I completely took over. Nowadays, we have the script running fully automated to build nightlies.


Thanks to Martin, the process for building DMD release packages is described step-by-step [in the D Wiki](https://wiki.dlang.org/DMD_Release_Building) so that anyone can do it if necessary. Moreover, he coauthored and implemented [a D Improvement Proposal](https://wiki.dlang.org/DIP75) to clearly define a release schedule. This process was adopted beginning with DMD 2.068 and continues today.

With the improved release schedule, DMD users can plan ahead to anticipate new releases, or take advantage of nightly builds and point releases to test out bug fixes or, whenever they come around, new features in the language or the standard library. However, the schedule is not etched in stone, as any particular release may be delayed for one reason or another. For example, [the 2.071 release](http://dlang.org/changelog/2.071.0.html) introduced major changes to D's import and symbol lookup to fix some long-standing annoyances, with [the subsequent 2.071.1](http://dlang.org/changelog/2.071.1.html) fixing some regressions they introduced. The release of 2.072 has been delayed until all of the known issues related to the changes have been fixed.

Those who were around before Martin stepped up to take charge of the release process surely notice the difference. He and the others who have contributed along the way (like Nick and Andrew) have done the D community a major service. As a result, Martin has become an essential member of the core D team.
