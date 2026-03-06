---
author: CraigDillabaugh
comments: false
date: 2017-01-13 13:08:01+00:00
layout: post
link: https://dlang.org/blog/2017/01/13/the-d-language-foundation-google-summer-of-code-2016-postmortem/
slug: the-d-language-foundation-google-summer-of-code-2016-postmortem
title: The D Language Foundation Google Summer of Code 2016 Postmortem
wordpress_id: 589
categories:
- D Foundation
- GSoC
- Guest Posts
permalink: /the-d-language-foundation-google-summer-of-code-2016-postmortem/
redirect_from: /2017/01/13/the-d-language-foundation-google-summer-of-code-2016-postmortem/
---

_Craig Dillabaugh was first drawn to D by its attractive syntax and Walter Bright's statement that D is "a programming language, not a religion". He maintains bindings to the geospatial libraries [shapelib](https://github.com/craig-dillabaugh/shplib_d) and [gdal](https://github.com/craig-dillabaugh/gdal), volunteered to manage the GSoC 2015 & 2016 efforts for D, and has taken it on again for 2017. He lives near Ottawa, Canada, and works for a network monitoring/security company called [Solana Networks](http://www.solananetworks.com/)._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/09/GSoC-icon-192.png)The [2016 Google Summer of Code](https://summerofcode.withgoogle.com/archive/2016/organizations/) (GSoC) proved to be a great success for [the D Language Foundation](https://summerofcode.withgoogle.com/archive/2016/organizations/4688223091556352/). Not only did we have, for us, a record number of slots allotted (four) and all projects completed successfully, perhaps most important of all we attracted four excellent students who will hopefully be long time contributors to the D Language and its community. This report serves as a review for the community of our GSoC efforts this past summer, and tries to identify some ways we can make 2017 an equal, or better, success.


### Background


Back in [2011](https://www.google-melange.com/archive/gsoc/2011/orgs/dprogramminglanguage) and [2012](https://www.google-melange.com/archive/gsoc/2012/orgs/dlang), Digital Mars applied to participate in, and was accepted to, Google Summer of Code. In each of those years we were awarded three slots and had successful projects. Additionally, a number of long time D contributors, including David Nadlinger, Alex Bothe, and Dmitry Olshansky, were involved as students. Sadly, in the succeeding two years we were not awarded any slots. After 2014's unsuccessful bid, Andrei asked on the forums if anyone wanted to take the lead for the 2015 GSoC, as he had too many things on his plate. This is when I decided to volunteer for the job.

I prepared for the 2015 GSoC and worked on getting some solid items for [our Ideas page](https://wiki.dlang.org/GSOC_2015_Ideas). I even prepared what I thought was a beautifully typeset document in [LaTeX](https://www.latex-project.org/) for our final submission. Needless to say, I was very disappointed when I had to copy/paste each section into the simple web form that Google provided for submissions. Sadly, that year we were rejected once more, though I felt our list of ideas was solid.

We applied again in 2016 for the first time as The D Language Foundation. Again, the community contributed lots of solid suggestions for [the Ideas page](https://wiki.dlang.org/GSOC_2016_Ideas) and we were accepted for the first time in four years. I think that perhaps getting accepted involves a bit of luck, as our ideas were similar to, or repeated from, those that were not accepted in 2015. However, more effort was put into polishing up the page, so perhaps that helped.


### The Selection Process


Once we were accepted as a mentoring organization, the process of receiving student proposals began. We received interest from a large number of students from all over the world (about 35). In the end, a total of 23 proposals were officially submitted, ranging from very short--obviously last minute--pieces, to several excellent efforts, including Sebastian Wilzbach’s 20-page document.

Our selection process was, I felt, very rigorous. We had seven of our potential admins/mentors screen the initial proposals. This involved reading all 23 proposals, which was a significant amount of work. From this initial screening we identified eight students/proposals that we thought could become successful projects. We then had all mentors individually rank each of the shortlisted proposals, another significant time commitment on their part.

Finally, interviews were arranged with all eight students. In most cases, two mentors interviewed each student, and the interviews were fairly intense, job-style interviews that involved coding exercises. A number of our mentors were involved in this process, but I think Amaury Sechet interviewed all of the students. It is no small feat to arrange and then conduct interviews with students in so many different time zones, so a huge thanks to all the mentors, but Amaury in particular. Those involved in the screening/interview process included Andrei Alexandrescu, Ilya Yaroshenko, Adam Ruppe, Adam Wilson, Dragos Carp, Russel Winder, Robert Schadek, Amaury, and myself.


### Awarding of Slots


The next step for our organization was to decide how many slots we would request from Google. I really had no idea what to expect, but I was hoping we might get two slots awarded to us, as there were many good organizations vying for a limited number of slots. We felt that most of the short-listed projects could have been successful, but decided to not be too greedy and requested just four slots. As it turned out, perhaps we should have asked for more; we were awarded all four. We then selected our top four ranked students from the interview process. They were, in no particular order:



 	
  * **Sebastian Wilzbach**: Science for D - a non-uniform RNG (_Ilya Yaroshenko mentor_)

 	
  * **Lodovico Giaretta**: [Phobos: std.xml](https://dlang.org/blog/2016/10/14/gsoc-report-std-experimental-xml/) (_Robert Schadek mentor_)

 	
  * **Wojciech Szeszol**: [Improvements to DStep](https://dlang.org/blog/2016/09/09/gsoc-report-dstep/) (_Russel Winder mentor_)

 	
  * **Jeremy DeHaan**: Precise Garbage Collector _(Adam Wilson mentor_)




### Summer of Code


Once the projects were awarded, I must say that most of my work was done. From there on the mentors and students got down to work. I tried to keep tabs on progress and asked for regular updates from both the mentors and the students. These were, in most cases, promptly provided.

While there were some challenges, and a few projects had to be modified slightly in some instances, everyone progressed steadily throughout the summer, so there were no emergencies to deal with. All of our students passed their mid-term evaluations and by the end of the summer all four projects were completed (although Jeremy has some on-going work on his precise GC). As a result, everyone got paid and, I presume, everyone was happy.

In addition to our original mentors, thanks are due to Jacob Carlborg (DStep) and Joseph Rushton Wakeling (RNG) for providing additional expertise.


### Mentor Summit


Google offered money for students to attend academic conferences and present results based on their GSoC work. Google also offered to pay travel costs for two mentors to travel to the mentor summit in California. Regrettably, none of our students had the time to take advantage of the conference money, but Robert Schadek was able to attend the Mentor Summit from Oct 28th to 30th in Sunnyvale, California. There he was able mingle with, and learn from, mentors from the other organizations that participated.


### Looking Forward


It is hard to believe, but the process starts all over again in a few short months. The success of this past year will create expectations for 2017, and I hope that we can replicate that success. A number of lessons were learned from this past year that we can carry forward into the next round. So in this section, I will try to distill some of what we learned to help guide our efforts in the coming year.


### The Ideas Page and Advertising


Most of the work of identifying projects was carried out through [the D Forums](http://forum.dlang.org/), with the odd email to past mentors. This was generally successful, but a number of proposals from previous years ended up being recycled. While it may be inevitable, it seemed that many of the proposal ideas were added at the last minute. Since a number of our best ideas from the 2016 page are now completed projects, we will need to replenish [the Ideas page for 2017](https://wiki.dlang.org/GSOC_2017_Ideas).
**Recommendations**



 	
  1. We should post a PDF version of one of the successful proposals on our Ideas page to give students an example of what we expect. Although it was excellent, we likely shouldn’t use Sebastian Wilzbach’s treatise, as that may scare some people off.

 	
  2. Try to get a decent set of solid proposals with committed mentors earlier in the process. In 2016 a number of the mentors were signed up at the last minute. The earlier the proposals are posted the more time we have to polish them and make them look more attractive.




### Interview and Selection Process


The selection process went well, but was a lot of work. Having input from a number of different mentors/individuals was invaluable.
**Recommendations**



 	
  1. Streamline the selection process, but reuse much of what was done last year. Having a rigorous selection process was a key contributor to 2016’s success.

 	
  2. Start the interview portion of the selection process earlier so that we have more time to set up and carry out the interviews.




### Project Progress and Mentoring


Much of the success of an individual project involves having a good relationship and work plan between the student and mentor. From this perspective, the organization isn’t heavily involved. Since all of our students worked well with their mentors, even less organizational administration was required. This is a byproduct of good screening and a solid set of ideas, and being fortunate enough to get good students.

However, there are areas where we could have run things a bit better. Students and mentors were asked to regularly provide updates on their progress, and they generally did this well, but there was no formal reporting process. Also, it would be worthwhile to have a centralized collection of project timelines/milestones where administrators and others involved in the projects (we had a few individuals working in advisory roles) can keep an eye on project progress.

**Recommendations**



 	
  1. We should keep a centralized version of project timelines somewhere (ie. Google Docs Spreadsheet) where we can check on project milestones. This should be shared with all individuals involved in a project (student/mentors/advisors/admins).

 	
  2. Have a more formalized process for students and mentors reporting on their progress. This would involve weekly student updates and biweekly mentor updates.




### Summary


The 2016 GSoC was a great success, and with any luck will be a good foundation for our successful participation in the year to come. We were fortunate that everything seemed to fall nicely into place, from our being awarded all four projects, to having all of our students complete their projects. Perhaps Sebastian, Lodovico, Wojciech or Jeremy will be involved again as students (or even mentors), and in any case continue to contribute to the D Language.
