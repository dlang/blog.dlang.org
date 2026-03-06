---
author: RazvanEduard
comments: false
date: 2021-08-26 14:27:52+00:00
layout: post
link: https://dlang.org/blog/2021/08/26/d-summer-school-v3/
slug: d-summer-school-v3
title: D Summer School v3
wordpress_id: 2966
categories:
- Community
- Guest Posts
permalink: /d-summer-school-v3/
redirect_from: /2021/08/26/d-summer-school-v3/
---

![]({{ '/assets/images/d-summer-school-v3/logo_256-200x195.png' | relative_url }})

The third edition of the D Summer School, held at [University POLITEHNICA of Bucharest](https://upb.ro/en/), took place from the 5th to the 25th of July. It was three weeks of boot camping bachelor students into the basics of D across eight sessions of hands-on workshops and a hackathon. We will describe our experience in organizing the program, teaching the students, and trying to integrate them into the D community.



## Recap from past editions



For the first two editions we had the following process:





  * we started advertising the summer school two months before the event;


  * we selected students from among the applicants;


  * the students had to complete a project during the summer school;


  * we helped students improve their projects during the hackathon;


  * we collected feedback at the end.



For more information, we [recommend our previous article on the first edition](https://dlang.org/blog/2019/08/08/d-summer-school-postmortem/).



## DSSv3





### Pre-event actions



In contrast to previous years, this time we started marketing the event very early in February, five months before the start date. We used the most influential vectors we could to promote it: the most popular professors. We nagged them ceaselessly to promote DSSv3 during their courses. The results were spectacular: we received 86 student applications (as opposed to 25 in the previous years). Since this was an online version of the summer school, we decided to drop the selection process and open the door to everyone. This had the added benefit that we didn't have to conduct interviews with everyone and a wider range of students had a chance to be introduced to D.

To cope with the larger number of participants, we had to grow our team. Former [Symmetry Autumn of Code](https://dlang.org/blog/symmetry-autumn-of-code/) participants Robert Aron and Adela Vais, and Symmetry employee Alexandru Militaru, agreed to join us. As such, we were able to diversify our teaching materials and raise the overall quality of the presentations.

[The teaching materials](https://ocw.cs.pub.ro/courses/dss) were mostly the same as in the previous years; we simply reshuffled the order to have an incremental level of complexity and added a lab, "WebApp Tutorial using Vibe.d". Besides this, we also changed the project competition. During the previous editions of DSS we found that students were not very engaged with the project assignment, so this time we came up with something different. We created a Dlang Bug Fix Competition: the top three students who had the largest number of **merged** PRs that solved a Bugzilla issue would win [Raspberry Pi 400 kits](https://www.raspberrypi.org/products/raspberry-pi-400/) (you may have noticed the "DSSv3" labeled PRs on our three main repositories). You may think that contributing to a programming language is a scary task, however, the truth is there are dozens of approachable, easy-to-fix issues that give instant gratification to the contributor.

With all the planning into place, we were now ready to start DSSv3.



### The teaching sessions



A teaching session is comprised of an hour-long lecture and two hours of hands-on exercises. This year, we abandoned the slides in favor of tutorial-like examples. Since everything was online, we simply shared our screen and highlighted the different aspects of D in a practical way by directly compiling and running examples. This made the lessons more interactive as students enthusiastically asked "what happens if…?" questions, and we could easily demonstrate the results.

For the exercises, we followed a team-play system where students were grouped in teams of four, and they worked together to solve their tasks. This made it easier for us to organize everything on the Teams platform (we would enter rooms of four students instead of talking students individually) and it encouraged them to help each other.

From among the 86 applicants, we had an average of 35 students attending the sessions, with a record high of 56 ("Introduction to D") and a low of 25 ("C\C++ Interoperability and Tooling"). It seems that from the first lecture to the last, almost half of the students abandon the course. This may seem like a grim figure, but note that we had students from all types of backgrounds, some of them in their first year at university. Since we are teaching subjects like memory management and multithreading, it's only natural that some of them will be lost along the way. Regardless, the lowest number of attendees was higher than the highest number from previous years.

The hackathon was attended by eight people. Again, a low figure, but that was not a surprise. Keep in mind that the majority of the participants had never made an open-source contribution. We expected that only the best students would manage to contribute. One other factor that may have influenced the low number was our uninspired decision to organize the hackathon on a Sunday; several people noted in our feedback form that they would have participated had they not had other plans. The result of the hackathon:





  * **9 PRs submitted to Phobos**: 5 were merged, 2 were closed but led to closing the associated Bugzilla entries, and 2 were rejected


  * **1 PR submitted to DRuntime** that was merged


  * **4 PRs submitted to DMD**: 2 were merged, 2 were rejected



We had hoped that students would submit PRs before the hackathon, but we were wrong. It seems that students should be assisted when making their first PR.

The winners of the hackathon were:



  1. **danandrei279** with 3 PRs merged


  2. **vladchicos** with 2 PRs merged


  3. **lucica28** with 1 PR merged




### Feedback



We asked the students to [fill out a feedback form](https://docs.google.com/forms/d/1QZ3LRL-yCmb1F58l-1OuF0sMkvser-97snj0cZFTpFc/edit), and we received 15 responses. It is highly possible that the results are biased since the feedback form was available at the end of the hackathon; by that time some students had already dropped out. Although it would have been great to understand their perspective, we still had valuable feedback on what went well and what could be improved.

From aggregating the results, we have the following conclusions.





  * The difficulty of DSS is perceived as being high. Those who are well prepared love it, but those who don't have too much programming experience are lost along the way.


  * The introductory courses are much more popular than advanced ones such as "C\C++ interop" and "Multithreading in D".


  * The hackathon was appreciated by hardcore programmers (a small percentage of the total number of attendees), but the rest were intimidated by it.


  * Students appreciated the relaxed interactive atmosphere of the sessions, with some commenting: "The general feel of the summer school was a chill evening hanging out with your friends."



Overall, DSSv3 generated enthusiasm among the programming geeks, but we still have some work to do to make it attractive to a less savvy audience.



### Future plans



Now that our team has grown, we plan to do a bigger restructuring of the course. Given the high drop-out rate, we would like to make the course welcoming for any type of CS student regardless of background or experience. To that end, we are considering creating two tracks for the course: one at a beginner level, and one for more advanced students. That way we can accommodate any type of audience.

Another aspect to think about is the hackathon. We still haven't found the most appealing project that would motivate students to commit to it. Experience has shown that creating a project from scratch in a language you've just learned doesn't really work (or we haven't found the adequate project) and contributing to the D language may be intimidating. We are still searching for better solutions, so if you have any ideas, please contact us.

Also, right now, UPB is going through a major redesign of its curriculum. Proposals for new courses are being accepted, and we will forward this course as our choice. There's a long wait for acceptance, but we're keeping our fingers crossed.



## Conclusions



Overall, we are happy with this year's edition. We managed to expand our team, grow our reach, and motivate some students to contribute to the language. Even though we still have some work to do to engage less passionate students, we think we are on the right track.

See you next time at DSSv4!
