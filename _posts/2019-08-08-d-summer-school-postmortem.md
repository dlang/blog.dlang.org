---
author: RazvanEduard
comments: false
date: 2019-08-08 12:59:41+00:00
layout: post
link: https://dlang.org/blog/2019/08/08/d-summer-school-postmortem/
slug: d-summer-school-postmortem
title: D Summer School Postmortem
wordpress_id: 2159
categories:
- Community
- Education
permalink: /d-summer-school-postmortem/
redirect_from: /2019/08/08/d-summer-school-postmortem/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)The first edition of the D summer school, held at [University POLITEHNICA of Bucharest](https://upb.ro/en/), took place from the 17th of June to the 4th of July. It was three weeks of bootcamping bachelor students into the basics of D during eight sessions of hands-on workshops, a homework project, and ending with a hackathon. We will describe our experience in organizing the program, teaching the students, and trying to integrate them into the D community.


## Who we are


We are Edi Staniloiu and Razvan Nitu, two PhD students at UPB, doing our theses in close relationship with the D programming language. For the past three years been recipients of scholarships from the D Language Foundation for contributing to the D ecosystem—you might also know us under the pseudonym of “Andrei’s students”.

During our first two years of contributing to D our focus was entirely on technical aspects, but last year we started thinking on how we can raise the popularity of the language both in our community and in the local industry. The summer school represents our first step in this direction.


## Inception


The idea of organizing a summer school first occurred to us during [DConf 2018 in Munich](https://dconf.org/2018/). We presented our thoughts to Andrei and he reacted with enthusiasm, but unfortunately there was too little time to organize something before the start of July. Why so early? Because that’s when students finish with their finals and prepare to leave for their internships. In UPB we have a wide range of summer schools that all start in that period and it doesn’t play well to go against tradition. So we decided to postpone it.

Even though we had to put off the summer school, we still wanted to introduce D to bachelor students one way or another. We thought that the best way to do so would be through a bachelor thesis project. That’s how we ended up working with Alex Militaru on his bachelor project, “D for a @safer Linux Kernel”, which he presented at [DConf 2019 in London](https://www.youtube.com/watch?v=weRSwbZtKu0&feature=youtu.be).

Alex is a top student, but he had never even heard of D. It’s not a difficult language to learn for anyone with programming experience, but it does take time to adapt to the details and subtleties of the language. This made for a rough start, as Alex had a very small time frame to learn it. The potential benefits of an introductory D school came to mind again, making us even more motivated to turn it into a reality. That’s when we committed to making the D summer school happen in 2019.


## First steps


The first thing that we had to agree on was the set of topics we were going to cover. D has a lot of interesting features. [Books of 300 to 500 pages have been written](https://wiki.dlang.org/Books) describing them in various levels of detail. We would not have the time to delve into all the details, so our goal was to touch all the basic concepts without being boring, but also highlight some of the most interesting aspects without being too complex. After a polite debate (which was neither polite, nor a debate)
we agreed on the following topics:



 	
  1. Introduction to D: builtin types, arrays (static, dynamic, associative), slices, imports, functions
(UFCS), unit tests, contract programming, user-defined types.

 	
  2. Introduction to meta-programming: `enum`, `static if`, `static foreach`, templates, template constraints.

 	
  3. Memory safety: `@safe`, type qualifiers, template inference, template `this` parameters.

 	
  4. Advanced D concepts: operator overloading, `alias`, `alias this`, overload sets, function attributes, ranges.

 	
  5. Multi-threading: data sharing, concurrency, `synchronized`, fibers

 	
  6. GC vs. Manual Memory Management.

 	
  7. Interoperability with C/C++ and tooling.

 	
  8. [Design by introspection](https://youtu.be/29h6jGtZD-U): `__traits`, `mixin`, tuples, CTFE, pragmas.


Each session was expected to last three hours: one hour of theoretical presentation and two hours of hands-on exercises (keyboard bashing). The theoretical structure was inspired (and some times shamelessly copy-pasted) from[ Ali’s awesome (and freely available) book](http://ddili.org/ders/d.en/). On this occasion, we would like to publicly thank him for allowing us to use his material.

The practical hands-on segment was split into two: a tutorial and observation period where the student, typically, had to run a program and understand the outcome; a hands-on period where the student had to write code to solve a given problem or fix some intentionally inserted bug.

In order to apply for the summer school, students had to [complete an assignment we devised](https://ocw.cs.pub.ro/courses/dss/assignment). The was the basis on which they were selected. If you are interested in details you can [check our official page](https://ocw.cs.pub.ro/courses/dss) where all the materials are located.


## Marketing


Now that we knew what the summer school was going to look like, all we had to do was find the students that would attend it. This may not sound like hard work, but here are a few considerations that will put things into perspective:



 	
  1. Every summer school, naturally, wants to attract the best possible students.

 	
  2. There are at least 6 summer schools taking place in UPB during the same period.

 	
  3. Many students have summer internships.


As you can see, not only did we have to compete with other established summer schools, but we also had to convince students that their free time after work would be well spent and that they would learn something cool, interesting, and, most importantly, useful.

Considering that both of us are complete noobs when it comes to marketing, we can say that this was the most challenging part. Luckily, our mentors Razvan Rughinis and Razvan Deaconescu, well established professors in our university and traditional summer school creators, were available to coach us through the basics of human manipulation, a.k.a. marketing. Their contributions came in many forms:

 	
  * they used their status to promote the summer school on all the university’s social media platforms

 	
  * they highlighted the fact that “Secure and Fast Programming in D” is a catchier title than the
blunt “D Summer School” that we had originally used

 	
  * they provided the grounds for obtaining funding from [a local organization, Tech Lounge](https://tech-lounge.ro/)


With that, all we had to do was wait for students to apply.


## The Actual Summer School


Contrary to our expectations, we weren’t flooded with student applications. Actually, rather disappointingly, we did not fill all of our spots. However, after consulting the history of the other summer schools, we learned that our expectations of gathering 40 students were rather unrealistic. To make a comparison, the star summer school in our university, which is the Security Summer School (currently in its 6th edition), had only 15 student applications and 10 participants in its debut year, and saw 40 student applications this year. By that standard, our total of 11 student applications doesn’t look that bad. After checking the submitted assignments, we decided that we would accept all of them.

Most of the summer school went according to plan, with some minor differences:



 	
  * The theoretical part, usually, took more than the planned timeslot of 1 hour, due to the high interest that students expressed with regard to the presented topics. Although we appreciated the level of interest, the remaining time often wasn’t sufficient to finish the practical part.

 	
  * It happened that Andrei Alexandrescu was in Romania during the summer school, so we thought it would be neat if he would teach the “Design by Introspection” course. As was expected, Andrei nailed it and the students were thrilled. However, after the presentation, the hype did not permit us to continue with the practical part, so we had to postpone it to another day. This put us into the position of dropping the “GC vs MMM” course, because we felt that it was more important to have the students get their hands dirty with some DbI.

 	
  * We had also planned that the students complete homework project during the summer school period. The assignment was to implement a simple peer-to-peer file sharing application [using vibe-d](https://vibed.org/). The project was intended for them to exercise their newly developed skills, but given the fact that most of the students were attending the summer school after they had previously gone to work, this left them with little time and energy to put into the project.

 	
  * The end-of-school hackathon was intended to have them finish their projects, but in reality, most of the projects were still in an incipient phase. The consequence was that nobody finished their project.


The funding that we obtained from Tech Lounge was used to buy beverages and snacks that were offered throughout the workshops. In addition, we used the money to buy personalized D T-shirts, pizza, and beer/soda/water for [everyone during the hackathon](https://photos.google.com/share/AF1QipMRoQCmOcPh4E9nvn4hL4gGXeebPDYV9lSlH8lMhaZJmL4z6lt6QcCNs8iFvPkmxw?key=WUU3eXY3T05vR09HZlQ3X3ZBTXdidTBNQ3YzU01n).


## Next edition


For the second edition (which will take place in June 2020) we will start marketing early, as soon as the school year starts in October 2019 (this year we started marketing in April). Hopefully, this will raise awareness and will lead to an increased number of participants.

We would also like to expand our team by integrating this years' participants into the teaching and material development process, thus increasing the quality of the experience for future participants.

As it has proven ineffective, the homework project will be dropped; instead, the students will be encouraged at the hackathon to make a Pull Request in [one of the core D projects](https://github.com/dlang).


## Conclusions


At the end of the summer school, we felt that the students were impressed with the language and its capabilities. Most of them have expressed their desire to get involved in the community. In response, we have encouraged them to apply to [the Symmetry Autumn of Code](https://dlang.org/blog/symmetry-autumn-of-code/) or contribute to the wider D ecosystem. We cannot know which path they will take from here but hope that they will continue to build on their recently acquired D skills.

The summer school did not represent the only way in which students could get a crash course on D, but it did increase our exposure to other departments at UPB. As a result, we’ve had multiple discussions on how we can integrate D in various university projects. To name a few:



 	
  * introduce D to specific programming courses

 	
  * [integrate D with Wyliodrin](https://wyliodrin.com/)

 	
  * [add D support to unikernel](http://unikernel.org/)


All these discussions are now materializing in student project proposals, thus expanding our community.

All in all, we feel that this was a great first edition. Not only did we have great students who will hopefully join our community, but we are now also on the radar of our university peers.

Now we’re looking forward for “Secure and Fast Programming in D” V2!
