---
author: AdamRuppe
comments: false
date: 2019-01-24 13:50:14+00:00
layout: post
link: https://dlang.org/blog/2019/01/24/last-year-in-d/
slug: last-year-in-d
title: Last Year In D
wordpress_id: 1924
categories:
- Community
- D Foundation
- Guest Posts
permalink: /last-year-in-d/
redirect_from: /2019/01/24/last-year-in-d/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)2018 was one of the biggest years in D we’ve had for a while. It was the first complete year that DMD stuck to a steady release schedule. Over the course of the year, the language got new features and cruft cleanup. The compiler even got some new user-facing features. At the same time, the community-at-large continued to evolve the surrounding ecosystem.


## Release Rundown


The first DMD release of the year was [version 2.078 on January 1](https://dlang.org/changelog/2.078.0.html). It set the tone for much of the year, including more [support for a stripped-down runtime](https://dlang.org/changelog/2.078.0.html#raii) for `-betterC` mode, and [a deprecation change in arithmetic rules](https://dlang.org/changelog/2.078.0.html#fix11006).

The next major release [came out in March](https://dlang.org/changelog/2.079.0.html), and I personally found it to be the most exciting release in a couple of years because [it came with improved error messages](https://dlang.org/changelog/2.079.0.html#argument-mismatch) - notably, calling out the specific argument to a function that mismatched a type (though I will note there is still a lot of room for further improvement!). Moreover, this release [introduced DMD’s `-i` flag](https://dlang.org/changelog/2.079.0.html#includeimports), which automatically includes imported modules and has proven to be a major convenience to me over the year.

Meanwhile, more stripped-down runtime support and general language cleanup was included in the release. March also introduced [experimental `@nogc` exception throwing](https://dlang.org/changelog/2.079.0.html#dip1008) (as [proposed in DIP1008](https://github.com/dlang/DIPs/blob/99b09355e1161a8288a9d1e4a6a99a3cc58f850f/DIPs/DIP1008.md)). This release also [changed the garbage collector](https://dlang.org/changelog/2.079.0.html#lazy-gc-init) to be lazily initialized, part of the effort to make things cheaper and pay-as-you-go.

March was also the first time DMD on Windows could do a total build of a 64-bit program without Visual Studio installed because [it bundled the LLVM linker](https://dlang.org/changelog/2.079.0.html#lld_mingw).

Keeping to the release schedule, May and July brought us new major versions, which focused on language cleanup, with a lot of deprecations. There was [a #dbugfix campaign over the year](https://dlang.org/blog/2018/02/03/the-dbugfix-campaign/), and it brought a fix in [May’s release](https://dlang.org/changelog/2.080.0.html): allowing the `pow` operator (`^^`) and several `std.math` functions [to be used at compile time](https://dlang.org/changelog/2.080.0.html#fix5227). [July’s release](https://dlang.org/changelog/2.081.0.html) brought [a change in contract syntax](https://dlang.org/changelog/2.081.0.html#expression-based_contract_syntax), based on the [proposal in DIP 1009](https://github.com/dlang/DIPs/blob/99b09355e1161a8288a9d1e4a6a99a3cc58f850f/DIPs/accepted/DIP1009.md), allowing small expressions instead of requiring whole blocks.

[Version 2.082](https://dlang.org/changelog/2.082.0.html), in September, finally brought me something I have wanted for a long time: [User Defined Attributes](http://ddili.org/ders/d.en/uda.html) on [`enum` members and function parameters](https://dlang.org/changelog/2.082.0.html#uda-function-parameters), as well as the ability to [disable DRuntime’s exception trapping](https://dlang.org/changelog/2.082.0.html#exceptions-opt) via the `--DRT-trapExceptions=0` runtime command-line switch, which allows for easier debugging of uncaught exceptions. Moreover, with this release the D Language Foundation began [digitally signing the Windows binary releases](https://dlang.org/changelog/2.082.0.html#signed_windows_binaries), which has helped smooth out the end-user experience when installing and running DMD. It should continue to help solve the false-positive problem we have seen with some virus scanners.

At the end of the year, [version 2.083](https://dlang.org/changelog/2.083.0.html) saw extern(C++) get a big improvement its users have been waiting for: [namespaces without scopes](https://dlang.org/changelog/2.083.0.html#mangle_cpp), which makes C++ interoperability and code organization a lot easier. Work on making the C++ standard library accessible from D has been progressing throughout the year.

Overall, the year brought a lot of awaited improvements. D is more usable in various runtime library situations (even Phobos has some `-betterC` support, like `RefCounted` and `Tuple`!). The language had over ten deprecations of old cruft like comma expressions, read-modify-write on shared, comma expressions, class allocators (did you even know D still had class allocators? To be honest, I thought they were formally deprecated years ago, but it was actually 2018 when the change officially happened!), and more.

Debugging got better, with the uncaught exception switch, but also allowing the `debug` statement to escape more attribute restrictions, better error messages out of the compiler, and a `-verrors=context` switch to show the line right in the message. `-betterC` debugging aids also improved, with the C assert function being a possibility. I’ll also note that 2019 is already moving further—new `assert` printing code was merged just recently; we can choose how to control them and whether to use D or C facilities!

Of course, the library continued to get better range support and its support for `@nogc` and `-betterC` has grown as well.

And as a fun fact, DMD is now 98% ported to D, with another major part converted over the year.


## The D Community


More companies invested in D in 2018. Hosted by [QA Systems](https://www.qa-systems.de/), [DConf 2018 was held](https://dconf.org/2018/) in Munich, Germany, the first time DConf has been to that city (and the third time in Germany, after taking place in Berlin in [2016](https://dconf.org/2016/) and [2017](https://dconf.org/2017/)). Symmetry Investments sponsored the successful [Symmetry Autumn of Code](https://dlang.org/blog/symmetry-autumn-of-code/), funding three students to work on D-related projects.

[run.dlang.io came out](https://run.dlang.io/) at the end of 2017 and grew in popularity throughout 2018, becoming the new standard for running D code online, including from [the dlang.org homepage](https://dlang.org/). It even supports a whitelist of third-party libraries.

[The dlang-tour website](https://tour.dlang.org/) gained a few new translations from the community, including Vietnamese, Portugese, French, Turkish, German, and Ukrainian.

The D Language Foundation was busy in 2018 as well. They received [over $5000 on Open Collective](https://opencollective.com/dlang), $3000 of which was earmarked to support development of [the code-d VS Code plugin](https://github.com/Pure-D/code-d) and [the language server](https://github.com/Pure-D/serve-d) that drives it. With [a successful campaign launched through Flipcause](https://dlang.org/blog/2018/11/10/the-new-fundraising-campaign/), the DLF was able to hire a pull request manager for three months. [Donations through all platforms](https://dlang.org/foundation/donate.html) allowed them to fund some outreach efforts as well as student work via scholarships. To increase visibility, they submitted a history of D paper to the [Fourth ACM SIGPLAN History of Programming Languages Conference](https://hopl4.sigplan.org/).

Over 2,500 pull results have now been merged by the dlang-bot on GitHub, and the Project Tester has now gained 50 projects that it tests on every PR to give real-world data on compiler regressions and the impact of breakages.

[DIP 1014, “Hooking D’s struct move semantics”](https://github.com/dlang/DIPs/blob/99b09355e1161a8288a9d1e4a6a99a3cc58f850f/DIPs/accepted/DIP1014.md), was also accepted. It will open some doors that were closed by design in old D, but that many developers had found limiting. Before, the compiler could move your structs whenever it wanted. Now, it will be hookable to give more control to the programmer and avoid a nasty case of bugs.

I opened up my [documentation generation website at dpldocs.info](https://dpldocs.info/) to all DUB projects this year, and upstream linked to it, encouraging D library authors to better document their libraries and allowing users to better evaluate the libraries before downloading them.

On Twitter, [@dlang_ng was started](https://twitter.com/dlang_ng) and has gained about 200 followers. All announcement topics from the [Announce forum](https://forum.dlang.org/group/announce) (aka [the digitalmars.D.announce newsgroup](https://digitalmars.com/NewsGroup.html)) are tweeted out here.

2018 also saw community announcements of the autowrap and dpp projects. [autowrap automatically wraps](https://github.com/kaleidicassociates/autowrap) existing D code to be used from other environments, while [dpp runs a C pre-processor over D code](https://github.com/atilaneves/dpp) to make it possible to use C headers, unmodified, macros and all, directly in D. These two projects show the community’s desire to integrate D more fully into existing codebases and projects.

Of course, not all is perfect in the D development community, including a few areas where this author would like to see improvement.

The most +1-ed PR on DMD, [a string interpolation implementation](https://github.com/dlang/dmd/pull/7988) (and a very elegant approach in this writer’s opinion), remains open and of uncertain status, despite a renewed effort to get it merged in December. Similarly, [the State of D 2018 Survey](https://dlang.org/blog/2018/02/28/the-state-of-d-2018-survey/) identified [several areas for improvement](https://dlang.typeform.com/report/H1GTak/PY9NhHkcBFG0t6ig). Very few of these came to pass in 2018, though some progress was made. We deprecated some cruft, but auto-decoding, one of the top-five disliked Phobos features in the survey and one often derided in the forums, remains in place. We are still using Bugzilla, which got an average of 3.27 satisfaction rating out of a possible 5 stars in the survey. We achieved fewer regressions, but added even more to the attribute bloat. We have [an official blog](https://dlang.org/blog/) and [the annual DConf](https://dconf.org), but the D Language Foundation’s inner workings are still opaque.

It would be great if some of these weak points are addressed among the improvements and changes in 2019.


## The DUB Package Manager


[The code.dlang.org website](https://code.dlang.org/) received some criticism in 2017, and it was addressed in 2018. The site got a new front page with usage statistics and project ranking based on GitHub info. The DUB program itself got faster and more stable, no longer going online on every run to check for updates, and achieving better online uptime.

[The Mir library](https://github.com/libmir/mir), which has algorithms, collections, and more written to be 100% `-betterC` compatible (which, of course, means it also works in all other D environments, too), is now the fifth-most popular package on the DUB registry, while [the unit testing library unit-threaded](https://github.com/atilaneves/unit-threaded) takes the third-place slot. First, second, and fourth are all [related to vibe.d](https://vibed.org/), which is the project DUB was originally created to support.


## Conclusion


2018 was a really good year for D. There is still much work to do, but worthwhile developments in all facets of the language and ecosystem gave me renewed excitement going into 2019. I expect that March 2019 release of D is going to be another big step toward improving the best programming language we have on Earth today!



* * *



_Adam Ruppe is the author of [D Cookbook](http://amzn.to/1ZTE47m) and maintainer of [This Week in D](http://arsdnet.net/this-week-in-d). Modules from his freely available [arsd package](https://github.com/adamdruppe/arsd) are used throughout the D community. He is also known for his [legendary DConf 2014 presentation](http://dconf.org/2014/talks/ruppe.html)._



* * *





## Addendum


There were a total of 166 contributors listed on the DMD changelog in 2018. Special thanks to them and to all the others in the D community who flesh out the ecosystem and make our favorite programming language!



 	
  * 0xEAB

 	
  * Adam D. Ruppe

 	
  * Alexandru Caciulescu

 	
  * Alexandru Jercaianu

 	
  * Alexandru ermicioi

 	
  * Alexibu

 	
  * Ali Akhtarzada

 	
  * Ali Çehreli

 	
  * Andrei Alexandrescu

 	
  * Andrei-Cristian VASILE (87585)

 	
  * Andrey Penechko

 	
  * Andy Smith

 	
  * Aravinda VK

 	
  * Arun Chandrasekaran

 	
  * Atila Neves

 	
  * BBasile

 	
  * Basile Burg

 	
  * Bastiaan Veelo

 	
  * Benoit Rostykus

 	
  * Brad Roberts

 	
  * Brian Schott

 	
  * Carsten Schlote

 	
  * Chris Coutinho

 	
  * Daniel Kozak

 	
  * Dashster

 	
  * David Bennett

 	
  * David Gileadi

 	
  * David Nadlinger

 	
  * Denis Feklushkin

 	
  * Diederik de Groot

 	
  * Dmitry Olshansky

 	
  * Dragos Carp

 	
  * Duncan Paterson

 	
  * Eduard Staniloiu

 	
  * Elias Batek

 	
  * Erik van Velzen

 	
  * Eugen Wissner

 	
  * FeepingCreature

 	
  * GabyForceQ

 	
  * Giles Bathgate

 	
  * GoaLitiuM

 	
  * Greg V

 	
  * H. S. Teoh

 	
  * Harry T. Vennik

 	
  * Hiroki Noda

 	
  * Héctor Barreras Almarcha [Dechcaudron]

 	
  * Iain Buclaw

 	
  * Ilya Yaroshenko

 	
  * Ionut

 	
  * Jack Stouffer

 	
  * Jacob Carlborg

 	
  * Jan Jurzitza

 	
  * Jean-Louis Leroy

 	
  * JinShil

 	
  * Joakim Noah

 	
  * Johan Engelen

 	
  * Johannes Loher

 	
  * Johannes Pfau

 	
  * John Belmonte

 	
  * John Colvin

 	
  * Jon Degenhardt

 	
  * Jonathan M Davis

 	
  * Jonathan Marler

 	
  * Jordi Sayol

 	
  * Joseph Rushton Wakeling

 	
  * Kai Nacke

 	
  * Kevin De Keyser

 	
  * Kotet

 	
  * Laeeth Isharc

 	
  * Lance Bachmeier

 	
  * Leandro Lucarella

 	
  * LemonBoy

 	
  * Lucia Mcojocaru

 	
  * Luís Marques

 	
  * Manu Evans

 	
  * Manuel Maier

 	
  * Markus F.X.J. Oberhumer

 	
  * Martin Kinkelin

 	
  * Martin Krejcirik

 	
  * Martin Nowak

 	
  * Mathias Baumann

 	
  * Mathias Lang

 	
  * Mathis Beer

 	
  * MetaLang

 	
  * Michael Parker

 	
  * Mihails Strasuns

 	
  * Mike Franklin

 	
  * Mike Parker

 	
  * Márcio Martins

 	
  * Nathan Sashihara

 	
  * Nemanja Boric

 	
  * Nicholas Lindsay Wilson

 	
  * Nicholas Wilson

 	
  * Nick Treleaven

 	
  * Oleg Nykytenko

 	
  * Patrick Schlüter

 	
  * Paul Backus

 	
  * Per Nordlöw

 	
  * Petar Kirov

 	
  * Pjotr Prins

 	
  * Pradeep Gowda

 	
  * Quirin F. Schroll

 	
  * Radosław Rusiniak

 	
  * Radu Racariu

 	
  * Rainer Schuetze

 	
  * Razvan Nitu

 	
  * Remi Thebault

 	
  * Robert burner Schadek

 	
  * Roman Chistokhodov

 	
  * Ryan David Sheasby

 	
  * Ryan Frame

 	
  * Sebastian Wilzbach

 	
  * Simen Kjærås

 	
  * Simon Naarmann

 	
  * Spoov

 	
  * Stanislav Blinov

 	
  * Stefan Koch

 	
  * Steven Schveighoffer

 	
  * Superstar64

 	
  * Temtaime

 	
  * Tero Hänninen

 	
  * Thibaut CHARLES

 	
  * Thomas Mader

 	
  * Timon Gehr

 	
  * Timoses

 	
  * Timothee Cour

 	
  * Tomáš Chaloupka

 	
  * Tyler Knott

 	
  * Unknown

 	
  * Vlad Vitan

 	
  * Vladimir Panteleev

 	
  * Walter Bright

 	
  * Yannick Koechlin

 	
  * Yuxuan Shui

 	
  * Zach Tollen

 	
  * Zevenberge

 	
  * aG0aep6G

 	
  * abaga129

 	
  * byebye

 	
  * carblue

 	
  * cedretaber

 	
  * crimaniak

 	
  * devel

 	
  * deviator

 	
  * dhasenan

 	
  * dmdw64

 	
  * drug007

 	
  * dukc

 	
  * gapdan

 	
  * glitchbunny

 	
  * growlercab

 	
  * jercaianu

 	
  * jmh530

 	
  * kinke

 	
  * leitimmel

 	
  * look-at-me

 	
  * n8sh

 	
  * rracariu

 	
  * shoo

 	
  * shove70

 	
  * skl131313

 	
  * thaven

 	
  * viktor

 	
  * wazar

 	
  * olframw

 	
  * yashikno

 	
  * Ľudovít Lučenič


