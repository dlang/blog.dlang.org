---
author: DBlogAdmin
comments: false
date: 2017-05-19 14:09:25+00:00
layout: post
link: https://dlang.org/blog/2017/05/19/dconf-2017-ex-post-facto/
slug: dconf-2017-ex-post-facto
title: DConf 2017 Ex Post Facto
wordpress_id: 762
categories:
- D Foundation
- DConf
permalink: /dconf-2017-ex-post-facto/
redirect_from: /2017/05/19/dconf-2017-ex-post-facto/
---

![](http://dlang.org/blog/wp-content/uploads/2017/04/dconf.jpg)Another May, [another DConf](http://dconf.org/2017/index.html) in the rear view. This year's edition, organized and hosted once again by the talented folks from Sociomantic, and emceed for the second consecutive year by their own Dylan Cromwell, brought more talks, more fun, and an extra day. The livestream this year, barring a glitch or two with the audio, went almost perfectly. And for the first time in DConf history, videos of the talks started appearing online almost as soon as the conference was over. [The entire playlist](https://www.youtube.com/playlist?list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB) is available now.

As usual, there were three days of talks. The first opened with [a keynote by Walter](https://www.youtube.com/watch?v=iDFhvCkCLb4&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=1) and the last with [one by Andrei](https://www.youtube.com/watch?v=29h6jGtZD-U&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=19) (he gave [a longer version of the same talk](https://youtu.be/es6U7WAlKpQ) at Google's Tel Aviv campus a few days later). [The keynote from Scott Meyers](https://www.youtube.com/watch?v=RT46MpK39rQ&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=11) on Day Two, in his second DConf appearance, was actually the second talk of the day thanks to some technical issues. He told everyone about the things he finds most important in software development, a talk recommended for any developer no matter their language of preference.

The keynotes were followed by presentations from a mix of DConf veterans and first-time speakers. This year, livestream viewers were treated to some special segments during the lunch breaks. On Day One, Luís Marques showed off [a live demo](https://www.youtube.com/watch?v=vjaKFTHQSxQ&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=4) using D as a hardware description language, which had been the subject of [his presentation](https://www.youtube.com/watch?v=kTRkTOSHki4&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=3) just before the lunch break (he used a Papillo Pro from [Gadget Factory](http://store.gadgetfactory.net) in his demo, and the company was kind enough to provide an FPGA for one lucky attendee to take home). On Day Two, Vladimir Panteleev, after being shuffled from the second spot to the first in the lineup, gave [a livestream demo](https://www.youtube.com/watch?v=eDBsCC82cko&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=13) of concepts he had discussed in his talk on [Practical Metaprogramming](https://www.youtube.com/watch?v=sDz-0tdh5Ko&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=10). And on the last day of presentations, Bastiaan Veelo [presented the livestream audience](https://www.youtube.com/watch?v=3ugQ1FFGkLY&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=22) with an addendum to his talk, [Extending Pegged to Parse Another Programming Language](https://www.youtube.com/watch?v=t5y9dVMdI7I&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=21).

Day Two closed out with [a panel discussion](https://www.youtube.com/watch?v=Lo6Q2vB9AAg&list=PL3jwVPmk_PRxo23yyoc0Ip_cP3-rCm7eB&index=18&t=1495s) featuring Scott, Walter and Andrei.

[caption id="" align="aligncenter" width="5312"]![](http://i.imgur.com/96odrPF.jpg) Scott Meyers, Walter Bright, and Andrei Alexandrescu in a panel discussion moderated by Dylan Cromwell.[/caption]

It was during this discussion that Walter made the claim that memory safety will be the bane of C, eliciting skepticism from Scott Meyers. That exchange prompted more discussion on [/r/programming](https://www.reddit.com/r/cpp/comments/6b4xrc/walter_bright_believes_memory_safety_will_kill_c/) almost two weeks later.

The newest segment of the event this year came in the form of an extra day given over entirely to the DConf Hackathon. As originally envisioned, it was never intended to be a hackathon in the traditional sense. The sole purpose was for members of the D community to get together face-to-face and hash out the pain points, issues, and new ideas they feel strongly about. DConf 2016 had featured a "Birds of a Feather" session, with the goal of hashing out a specific category of issues, as part of the regular talk lineup. It didn't work out as intended. The hackathon, suggested by Sebastian Wilzbach, was conceived as an expansion of and improvement upon that concept.

The initial plan was to present attendees with a list of issues that need resolving in the D ecosystem, allow them to suggest some more, and break off into teams to solve them. Sebastian put a lot of effort into a shared document everyone could add their ideas and their names to. As it turned out, participants flowed naturally through the venue, working, talking, and just getting things done. Some worked in groups, others worked alone. Some, rather than actively coding, hashed out thorny issues through debate, others provided informal tutoring or advice. In the end, it was a highly productive day. Perhaps the most tangible result was, as Walter put it, "a tsunami of pull requests." It's already a safe bet that the Hackathon will become a DConf tradition.

In the evenings between it all, there was much food, drink, and discussion to be had. It was in this "downtime" where more ideas were thrown around. Some brought their laptops to hack away in the hotel lobby, working on pet projects or implementing and testing ideas as they were discussed. It was here where old relationships were strengthened and new ones formed. This aspect of DConf [can never be overstated](http://dlang.org/blog/2017/04/19/the-dconf-experience/).

A small selection of more highlights that came out of the four days of DConf 2017:



 	
  * Walter gave the green light to change the D logo and a strategy was devised for moving forward.

 	
  * Jonathan Davis finally managed to get [std.datetime](https://dlang.org/phobos/std_datetime.html) split from a monolithic module into a package of smaller modules.

 	
  * Some contentious issues regarding workflow in the core repositories were settled.

 	
  * Vladimir Panteleev gave [DustMite](https://github.com/CyberShadow/DustMite/wiki) the ability to reduce diffs.

 	
  * Timon Gehr [implemented](https://github.com/dlang/dmd/pull/6760) `static foreach` (yay!).

 	
  * Ali Çehreli finished updating his book [Programming in D](http://ddili.org/ders/d.en/index.html) to [2.074.0](http://dlang.org/changelog/2.074.0.html).

 	
  * Nemanja Boric fixed the broken exception handling on FreeBSD-CURRENT.

 	
  * Steven Schveighoffer and Atila Neves earned their wizard hats for submitting their first pull requests to DMD.

 	
  * Adrian Matoga and Sönke Ludwig (and probably others) worked on fixing issues with [DUB](https://code.dlang.org/download).

 	
  * Progress was made on the D compiler-as-a-library front.


This is far from an exhaustive list. The venue was a hive of activity during that last day, and who knows what else was accomplished in the halls, restaurants, and hotel lobbies. This short list only scratches the surface.

A very big Thank You to everyone at Sociomantic who treated us to another spectacular DConf. We're already looking forward to DConf 2018!

[caption id="" align="alignnone" width="4032"]![](http://i.imgur.com/JdTFDmr.jpg) Thanks Sociomantic![/caption]
