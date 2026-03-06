---
author: DBlogAdmin
comments: false
date: 2021-06-18 12:54:25+00:00
layout: post
link: https://dlang.org/blog/2021/06/18/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/
slug: d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases
title: D News Roundup
wordpress_id: 2902
categories:
- Compilers &amp; Tools
- DConf
- DMD Releases
- GDC Releases
- LDC Releases
- News
- SAoC
permalink: /d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/
redirect_from: /2021/06/18/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/
---

Version 2.097.0 of DMD, the D programming language reference compiler, was released on June 5th in the middle of new GDC and LDC release announcements, while preparations for two major D community events were underway: the Symmetry Autumn of Code 2021 and DConf Online 2021. We'll cover it all in this post, with a focus first on the events.



## Symmetry Autumn of Code 2021



![Symmetry Investments logo]({{ '/assets/images/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/logo-1.png' | relative_url }})

As I write, [Symmetry Investments](https://symmetryinvestments.com/) employs in the neighborhood of 180 full-time workers and manages over US$8 billion of capital, and they're always [on the lookout for more employees](https://news.ycombinator.com/item?id=25272559), including programmers to work with D and other languages. They sponsored [DConf 2019 in London](https://dconf.org/2019/) and have sponsored the annual Symmetry Autumn of Code since 2018, in which a handful of programmers are paid to work for four months on projects of benefit to the D ecosystem.

This year marks the fourth annual SAoC, and we are now accepting applications. Participants will plan four milestones for projects that benefit the D ecosystem and will be expected to work at least 20 hours per week on each milestone. Each participant will be rewarded US$1000 for the successful completion of each of the first three milestones. At the end of the final milestone, the SAoC committee will review the overall progress of each of the remaining participants. One will be rewarded with a final $US1000 payment and a free pass to the next real-world DConf, with reimbursement for travel and lodging. In last year's event, a second participant was also awarded a fourth US$1000 payment.

Participation in SAoC has led to jobs for some lucky coders and has generally been a valuable learning experience for those who have completed it. Students currently enrolled in graduate or postgraduate university programs will be given priority, but applications are open to all. The application deadline is August 18th. Project ideas can be found in the D community's [projects repository at GitHub](https://github.com/dlang/projects/issues?q=is%3Aissue+is%3Aopen+label%3Asaoc). See [the Symmetry Autumn of Code page](https://dlang.org/blog/symmetry-autumn-of-code/) here at the D Blog for all the details on how to apply as a participant or as a mentor.



## DConf Online 2021



![]({{ '/assets/images/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/logo_256.png' | relative_url }})

For the second consecutive year, we were unable to hold a real-world DConf. Last year we launched the first annual DConf Online. And when I say _annual_, I mean _annual_! We're doing it again this year and will continue to do it going forward even after the real-world DConfs are back on.

DConf Online 2021 will take place November 20 and 21 on [the D Language Foundation's YouTube channel](https://www.youtube.com/channel/UC5DNdmeE-_lS6VhCVydkVvQ). Once again, we're looking for pre-recorded talks, livestream panels, and livecoding sessions. If you'd like to propose something in one of those categories, the application deadline is September 5. Please visit [the DConf Online 2021 homepage](https://dlang.github.io/dconf.org/2021/online/index.html) for all the details.

And if you haven't seen them yet, the [DConf Online 2020](https://www.youtube.com/watch?v=XQHAIglE9CU&list=PLIldXzSkPUXWsvFA4AuawPoMnq9Bh1lIZ) and [DConf Online 2020 Q & A](https://www.youtube.com/watch?v=4G1FoocVVPY&list=PLIldXzSkPUXX0PcnTlv175rEyfH66yaoI) playlists are available on the same channel. You can also find a full list of talks and all the links (talk videos, slides, and Q & A videos) on [the DConf Online 2020 homepage](https://dlang.github.io/dconf.org/2020/online/index.html).



## New compiler releases



D 2.097.0 is live in the latest release of DMD and the beta release of LDC, the LLVM-based D compiler. The new version of GDC also came into the world as part of GCC 11.1 at the end of April.



### DMD 2.097.0



![Digital Mars D logo]({{ '/assets/images/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/d6.png' | relative_url }})

[This version of DMD](https://dlang.org/download.html) comes with [29 major changes](https://dlang.org/changelog/2.097.0.html) and [144(!) fixed Bugzilla issues](https://dlang.org/changelog/2.097.0.html#bugfix-list) courtesy of 54 contributors. Changes include a few deprecations and several improvements to the standard library. Two things stand out:





  * **`while(auto n = expression)`** has been on a few wishlists for a while. [Now it's a reality](https://dlang.org/changelog/2.097.0.html#while-condition-assignment). The same syntax that was already possible with `if` statements is considered idiomatic in certain circumstances (such as when checking if an item exists in an associative array). Expect the while condition assignment to start popping up in open-source D projects soon.


  * **`std.sumtype`** is another wishlist item that is a wish no more. The new `SumType` is a replacement for `std.variant.Algebraic`. It's a discriminated union that makes good use of [Design by Introspection](https://youtu.be/HdzwvY8Mo-w) with a nice `match` syntax for those looking for that sort of thing. It's been quite a while since the last time a new module was added to the D standard library. Many thanks to Paul Backus for putting in the effort to see it through, and a very big Congratulations!





### LDC 1.27.0-beta1



![LDC logo]({{ '/assets/images/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/ldc.png' | relative_url }})

On the same day the new DMD was released, [the first beta of LDC 1.27.0](https://github.com/ldc-developers/ldc/releases/tag/v1.27.0-beta1), which also supports D 2.097.0, [was announced in the D forums](https://forum.dlang.org/post/zeorpkkhqorjbaacslcb@forum.dlang.org).

On top of 2.097.0 support, this version of LDC provides greatly improved DLL support on Windows. The prebuilt Windows packages ship with DRuntime and Phobos DLLs. This is big news for D developers on Windows. We've long had issues with D DLLs that have prevented heavy use outside of simple interfaces (with APIs exported as `extern(C)` being the most reliable).

There are some limitations to be aware of, such as the inability to directly access TLS variables across DLL boundaries (though it's fine with accessor functions). Please [see the release page](https://github.com/ldc-developers/ldc/releases/tag/v1.27.0-beta1) for the details.

Thanks to Martin Kinkelin and all the LDC maintainers and contributors for their continued work on LDC. They aren't getting paid for this. If you are a happy LDC user or just like the idea of the project, you can support their work by [sponsoring Martin Kinkelin on GitHub](https://github.com/sponsors/kinke).



### GDC 11.1



![]({{ '/assets/images/d-news-roundup-saoc-2021-dconf-online-2021-new-compiler-releases/compiler-gdc.jpg' | relative_url }})

In the GCC world, Iain Buclaw continues [to make strides on the GDC compiler](https://forum.dlang.org/thread/yyblavikfluxsbtrdxry@forum.dlang.org).

GDC 11.1 still uses the old C++ version of the D frontend, which feature-wise is mostly (see below) at D 2.076.1. There were significant issues in upstream DMD that prevented Iain from making the switch to the D version of the frontend in time to make the release window. He is currently aiming to make the switch in time for GDC 12. As a consolation, this release has support for three BSDs, Mac OS X, and MinGW!

Despite the older frontend, Iain has backported several fixes and optimizations, and even a few features, so it isn't your grandfather's D 2.076.1 that GDC supports. For example, [the new bottom type](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1034.md) that recently made its way through the D Improvement Proposal review process has found its way into this GDC release. [See the forum announcement](https://forum.dlang.org/post/yyblavikfluxsbtrdxry@forum.dlang.org) for details of all the new D goodness in GDC 11.1 and [Please consider sponsoring his work on GitHub](https://gdcproject.org/downloads>visit the GDC homepage</a> for information on how to get your hands on it.

Thanks to Iain for all of the work he continues to put into GDC. <a href=).



## One-off donations



If you aren't up for sponsoring Martin or Iain but would still like to support them financially, you can make one-time donations through the D Language Foundation. You can send money to the [D General Fund](https://www.flipcause.com/secure/cause_pdetails/NDMzMzE=), the [D Open Collective](https://opencollective.com/dlang), or [to our PayPal account](https://www.paypal.com/donate?token=DGsBiGtsi8auttUatHOoBD3bmaGmvssaVR5BURfuw5MT5YCiNpx-a0O0178egQF8j8-QGreBUqwhJneR). Whichever method you choose, please be sure to leave a note that the donation is intended for LDC, GDC, or any D project you would like to support. We'll make sure the appropriate person receives the money.

Other options for supporting the D programming language: visit [the D Language Foundation donation page](https://dlang.org/foundation/donate.html) and donate to one of our funds, head to [the DLang Swag Emporium](https://www.zazzle.com/store/dlang_swag?rf=238129799288374326) and purchase any items that catch your eye (the D Rocket stuff rocks, and DConf Online 2021 swag will be available shortly), or consider using [smile.amazon.com](https://smile.amazon.com/) and selecting the D Language Foundation as your charity the next time you shop at Amazon.com (we are only available through the .com domain; browser extensions like [SmartAmazonSmile](https://addons.mozilla.org/en-US/firefox/addon/smart-amazon-smile/) for Firefox and [AmazonSmileRedirect](https://chrome.google.com/webstore/detail/amazon-smile-redirect/ejglonclnjogoiegggjjcpapffbnangg?hl=en-US) for Chrome make it easy to do).

Thanks to everyone who has, will, or continues to support the D programming language, either through donations of time or money. We've gotten where we are through community effort, and community effort will keep pushing us forward. D rocks!
