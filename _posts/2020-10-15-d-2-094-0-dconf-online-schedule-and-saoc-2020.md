---
author: DBlogAdmin
comments: false
date: 2020-10-15 15:01:09+00:00
layout: post
link: https://dlang.org/blog/2020/10/15/d-2-094-0-dconf-online-schedule-and-saoc-2020/
slug: d-2-094-0-dconf-online-schedule-and-saoc-2020
title: D 2.094.0, DConf Online Schedule, and SAOC 2020
wordpress_id: 2726
categories:
- Community
- D Foundation
- DConf
permalink: /d-2-094-0-dconf-online-schedule-and-saoc-2020/
redirect_from: /2020/10/15/d-2-094-0-dconf-online-schedule-and-saoc-2020/
---

![Digital Mars D logo]({{ '/assets/images/d-2-094-0-dconf-online-schedule-and-saoc-2020/d6.png' | relative_url }})

The end of September saw [a new release of the reference D compiler](https://forum.dlang.org/post/yiwfozsqnbakmbgzriiq@forum.dlang.org), DMD 2.094.0, sporting the latest language features. That was followed not long after by [a beta release of LDC](https://forum.dlang.org/post/shwneawohteeqvegvqoq@forum.dlang.org), the LLVM-based D compiler, based on the same frontend version. The DMD 2.094.1 patch release [entered into beta](https://forum.dlang.org/post/oroaeukkyymnjcbibjoa@forum.dlang.org) a few days before this post was published. Meanwhile, the first Milestone of the Symmetry Autumn of Code has come to an end, and the DConf Online 2020 schedule has been published.





## DMD 2.094.0





This release of DMD incorporates [21 major changes and 119 fixed Bugzilla issues](https://dlang.org/changelog/2.094.0.html), thanks to [the efforts of 49 contributors](https://dlang.org/changelog/2.094.0.html#contributors). Here are some highlights.





### This ain't your grandpa's `in` parameter





Back in the days of yore, when DMD was still a pre-1.000 alpha, the D language supported `in`, `out`, and `inout` parameter storage classes. [They had the following meanings](https://www.digitalmars.com/d/1.0/function.html):







  * `in` (input), the default, was the bog standard function parameter which is a mutable copy of its argument, i.e., the normal passed-by-value parameter.


  * `out` (output) parameters were passed by reference and, upon function entry, initialized to [the default initializer value](https://dlang.org/spec/type.html#basic-data-types) for the parameter type (e.g., `0` for `int`, `float.nan` for `float`, etc).


  * `inout` (input/output) parameters were passed unmodified by reference.





When D2 came along, there were some changes. `inout` was replaced by the `ref` keyword and `out` kept the same meaning, but now there was an explicit restriction that these parameters could only take lvalue references; rvalue references, commonly used in C++, were forbidden as arguments. With `in`, things became a little muddy. And that brings us to `scope` parameters, a D2 feature that has evolved over time.





For quite some time, it was not fully implemented and only affected parameters that were delegates: the compiler would only allocate a closure for a `scope` delegate if it absolutely needed to. The D2 version of `in` was intended to be equivalent to `const scope`, but it was never fully implemented and was effectively equivalent to `const`. Today, `scope` is intended to be applied to `ref` or `out` parameters [to prevent them from escaping the function scope](https://dlang.org/spec/function.html#scope-parameters), and [with DMD 2.092.0](https://dlang.org/changelog/2.092.0.html), `in` finally became equivalent to `const scope`. In DMD 2.094.0, `in` [has been reimagined and extended](https://dlang.org/changelog/2.094.0.html#preview-in) to solve the rvalue reference issue.





The first thing to know about the new `in` is that it's still equivalent to `const scope`, but now the compiler is free to choose whether to pass an `in` parameter's argument by reference or by value. The second thing to know is that `in` parameters can now take rvalue references. All of this is implemented behind the `-preview=in` command line switch first introduced in 2.092.0.





Like any preview feature, the new `in` may or may not make it into the language proper, and if it does it might not be without changes. But for now, it's there and waiting to be put through its paces. The more people using it, pushing it, and looking for holes, the sooner we can know if this is the `in` we're looking for.





### Ddoc Markdown support





Quite a while ago, Ddoc, D's built-in documentation syntax, was enhanced to support some Markdown features. It was hidden beind a `-preview` switch. Now, that switch is no longer necessary--Ddoc [supports Markdown out of the box](https://dlang.org/changelog/2.094.0.html#markdown-default).





Note that this is not full-on Markdown. For example, although asterisks are supported for italic and bold text, underscores are not. But Markdown-style links, code blocks, inline code, and images are supported. For the details, see [the Documentation Generator documentation](https://dlang.org/spec/ddoc.html).





### More speed please





Since the release of DMD 2.091.0, the DMD binaries in the Windows release packages are being compiled with LDC. This is a good thing because LDC has a better optimizer than DMD, which makes DMD's fast compile times even faster. Now, LDC is used to compile binary releases on Linux, OSX, and FreeBSD. As a side effect, there are now no more 32-bit releases for FreeBSD, and additional binary tools are no longer included. If you need them, you can still pick them up from [https://digitalmars.com/](https://digitalmars.com/) or from older DMD releases.





### Download





The latest release of DMD is always available for download at [https://dlang.org/download.html](https://dlang.org/download.html). The latest Beta or Release Candidate can always be found there as well. You can also find links to download LDC and GDC, the GCC-based D compiler (which is now [an official component of GCC](https://gcc.gnu.org/)). While you're there, if you enjoy the D programming language, consider [leaving a tip to the D Language Foundation](https://www.flipcause.com/secure/cause_pdetails/NDMzMzE=).





## DConf Online 2020 Schedule





[DConf Online 2020](https://dconf.org/2020/online/index.html) is coming together nicely. [Over the two days of November 21 and 22](https://dconf.org/2020/online/index.html#schedule), we have nine prerecorded talks, a livestream Q & A with the language maintainers, and a livecoding session. We'll also be bringing [our annual real-world BeerConf](https://dlang.org/blog/2017/04/19/the-dconf-experience/) to the virtual world.





### The talks





The prerecorded talks will be scheduled to premiere [on our YouTube channel](https://www.youtube.com/channel/UC5DNdmeE-_lS6VhCVydkVvQ/) at the UTC times [listed on the schedule](https://dconf.org/2020/online/index.html#schedule). For the duration of each talk and for 15 minutes after, each speaker will be avalailable in a separate livestream for questions and answers related to the talk. We want to record the questions and answers verbally for posterity. The idea is that viewers of the prerecorded talk can ask questions in the video's chat, or ask in the livestream chat during or up to 15 minutes after the talk. The speaker will read the questions out loud. Short answers will be provided both verbally and in the chat. Longer answers will be provided verbally only. Commenters asking questions during the talk will be notified in the chat if their questions were selected so that they don't have to tab out to the Q & A and miss a portion of the talk. They can go back and watch the Q & A video later on our YouTube channel.





The livestream Q & A with the language maintainers will run on our YouTube channel. We'll be streaming a video conference call and questions will be taken from the livestream chat. During the livestream, some viewers will be invited to join in on the conference call and ask their question directly in order to provide more opportunity for follow up and feedback. Details on how to participate will be released on the day of the livestream.





Throughout the weekend, we'll be handing out prizes to random viewers. Eligibility details will be provided during the course of the event, so pay attention!





### BeerConf





BeerConf is a real-world DConf tradition dating back to the first edition of the conference, though the name didn't come around until Ethan Watson [coined it a few years later](https://dlang.org/blog/2017/04/19/the-dconf-experience/). Every year, we designate a gathering spot where DConf attendees can mingle every evening to unwind. The DConf days are where we all wear our D programmer hats and spend our time talking about our favorite programming language, but BeerConf is our chance to be human. We still talk about D, but we also have the opportunity to go beyond the code and get to know each other on a more personal level.





So for DConf Online, we're taking BeerConf online. On the evening (UTC) of Friday, November 20, we'll open the BeerConf video conference to any and all, and we'll leave it open all weekend. Despite the name, no alcohol is required to participate. All you need is an internet connection and a web browser, and you can come and go as you please. We've been running monthly BeerConf events since June of this year, so we know that, though it's not quite the same as being in the same place, it's still a lot of fun.





We hope to see you November 20-22 in BeerConf and DConf!





## Symmetry Autumn of Code





We are currently running our [third annual Symmetry Autumn of Code (SAOC)](https://dlang.org/blog/symmetry-autumn-of-code/). Sponsored by [Symmetry Investments](http://symmetryinvestments.com/), the event provides an opportunity for D programmers to make a little money working on projects aimed at improving the D ecosystem. Particpants each get paid $1000 for the successful completion of each of three milestones. At the end of a fourth milestone, the progress of each participant will be evaluated by the SAOC committee, then one participant will be awarded a final $1000 payment, and receive free registration and reimbursement for transportation and lodging for the next real-world DConf.





We currently have four programmers coding away toward their goals. Milestone 1 has just come to an end and Milestone 2 is set to begin. The participants will soon be sending in their milestone reports, their mentors will send in progress evaluations, and the SAOC Committee will review it all to determine if everyone has put forth the effort required to continue through the event (we expect no issues on that front!). You can follow the progress of each participant, and perhaps provide them with some timely advice, through their weekly updates [in the D General Forum](https://forum.dlang.org/group/general). Search for "SAOC2020".
