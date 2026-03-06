---
author: DBlogAdmin
comments: false
date: 2016-12-05 13:30:04+00:00
layout: post
link: https://dlang.org/blog/2016/12/05/the-d-language-foundations-scholarship-program/
slug: the-d-language-foundations-scholarship-program
title: The D Language Foundation's Scholarship Program
wordpress_id: 507
categories:
- Core Team
- D Foundation
- Interviews
- News
permalink: /the-d-language-foundations-scholarship-program/
redirect_from: /2016/12/05/the-d-language-foundations-scholarship-program/
---

![d6](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)The D Language Foundation [recently announced](http://dlang.org/dlangupb-scholarship.html) a new scholarship program aimed at EE and CS majors attending [University "Politehnica" Bucharest](http://www.upb.ro/en/) (UPB). I contacted Andrei Alexandrescu for a few details on how the initiative came together, hoping for just enough tidbits of backstory to craft a blog post around. He obliged in a big way, turning my one question and "a few details" into an informative conversation.

**Mike**: I assume quite a lot of work went into this. Could you share a few details about how it came about?

**Andrei**: Gladly! The story starts back in 2012, when I gave a talk at the [How to Web](http://2012.howtoweb.co/) conference in Bucharest, my native city. It was a great event and I got to meet many great people. Except for one whose name kept coming up all over the Romanian IT space, Andrei Pitis.

I heard he was an instructor in the CS department at UPB (the best IT school in Romania, also noted internationally). He's been directly involved in a number of IT-related foundations and professional organizations, and he created and led the immensely successful [Vector Smart Watch](http://vectorwatch.com/) startup. So, having heard he'd be around, I went to the conference speakers' dinner hoping to bump into him.

Not knowing what he looked like, I was just craning my neck in search of someone who seemed popular. Meanwhile, I was passing time by making chit chat with a nice fellow who introduced himself to me. Now, you know how these group parties go. There's always loud music and conversation, so I didn't even hear his name and assumed he hadn't heard mine.

As the evening progressed, I figured Andrei Pitis wasn't going to show, so I had more time to chat with that fine gentleman. And I noticed two things. First, he was incredibly insightful. Second, he seemed equally excited about meeting me as I was about meeting Andrei Pitis. After a long while, the coin dropped: they were one and the same.

Thus started a great friendship. Andrei gave me great tips about how to start and conduct The D Language Foundation. Recently, he introduced me to two UPB CS systems professors, Razvan Deaconescu and Razvan Rughinis (together, the three had created the [Tech Lounge](http://tech-lounge.ro) nonprofit organization dedicated to helping graduating CS students start their careers).

Razvan Rughinis came up with the scholarship idea while we were chatting over beers in the quaint old town of Bucharest. In great part the idea was motivated by the strong interest UPB systems graduate students had in participating in a high-impact open source project such as the D language as part of their MSc thesis. In systems research (unlike e.g. CS theory), actual system building is a key part of the research project; therefore, a visible OSS project makes for a much stronger dissertation than the usual throwaway experimental code.

Clearly a strong opportunity had presented itself, and the DLang UPB scholarship is its realization.

**Mike**: How does the selection process work?

**Andrei**: The two professors introduce a few candidates, which I pass through the rigors of the typical Facebook interview. We also ask for the usual suspects - proof of enrollment, transcripts, motivation letter, and references.

Of all components, the most important are (in order) the interview, the quality of the BSc projects, and the recommendation letters from their professors. The four current scholarship recipients passed the interview with flying colors and have very strong BSc projects and references. Some of them returned from summer internships at prestigious companies such as Bloomberg, others won CS awards. I have no doubt any company in the Bay Area or elsewhere would be happy to work with them. Once they finish their MSc, of course :o).

And I should mention here that the two professors aren't only involved in the selection process. They will make themselves available to help manage the students on an ongoing basis. We're very fortunate to have them.

**Mike**: Can you provide any info on the current recipients and their projects?

**Andrei**: The current recipients are Alexandru Razvan Caciulescu, Lucia Cojocaru, Eduard Staniloiu, and Razvan Nitu. I have posted an introduction to each [on the D forums](http://forum.dlang.org) and, now that you mention it, I told them to create a wiki page with a blurb for each. They are hosted in a nice shared office kindly donated by [Tech-Lounge.ro](http://www.tech-lounge.ro) and... we're in the process of getting a coffee machine up there :o).

They are all obviously interested in taking large systems projects that benefit their research interests and have an impact on the D language. To get them started, I took a page from Facebook's practice and defined a "bootcamp" program. Bootcamp is a month-long process (six weeks at Facebook) during which the so-called n00bs get familiar with the technologies used in the organization: the language proper; the core runtime and standard library; the build process; the way code changes are created, reviewed, accepted, and committed; and, last but not least, the community ethos and the kind of problems we are facing that are fit for ingenious solutions.

To kickstart the bootcamp program, I defined a "bootcamp" label in our [Bugzilla](http://issues.dlang.org) and applied it to a bunch of existing bugs, with an eye for the kind of bug that simultaneously has low surface (you don't need to know a lot of internal details to get into it) and offers a good learning experience. Right now each student is busy fixing a couple of such bugs.

Long-term we are looking at high-impact libraries and tools. I do have [a few ideas](https://wiki.dlang.org/Project_Ideas), but I have no doubt the students will come up with their own. Just give them time.

**Mike**: Speaking of time... is there any room here for an update on the D Foundation's finances?

**Andrei**: Of course. To be honest, right now we're in better shape than ever before (and than I would have hoped). Thanks to Sociomantic, who footed a large part of DConf 2016's bills, we have quite a bit of change left from conference registration fees. I have also personally carried a number of high-profile appearances at public tech events and private corporate training events, with proceeds flowing to the Foundation.

So we have accumulated a little war chest - not much, but definitely not negligible. With our current funds and operational costs, we are covered for over two years. Of course, the situation is fluid and I am working on expanding both income and (useful) expenditures.

We're running a very tight operation, and I want to keep it that way. By the Foundation bylaws, its officers (Walter Bright, Ali Çehreli, and myself) cannot get income from the Foundation, which preempts a variety of conflicts of interest. We are a public charity, which reduces and simplifies our taxation. We use modern, low-overhead money transfer methods such as [transferwise.com](http://transferwise.com) and constantly scan for better ones. Anyone who considers donating should know that about every five dollars donated goes straight to pay for one hour of an exceptional graduate student's time.

**Mike**: Are there more applications in the queue? Do you plan to extend scholarships to other universities?

**Andrei**: UPB seems to be off to a great start, but it's also a happy case for many reasons: it's my undergrad alma mater, we know professors there, and we don't need to pay tuition. If we wanted to extend a scholarship to another university we'd need to avail ourselves of similar strategic advantages. Needless to say, if anyone who reads this has ideas on the matter, please contact me.

Anyhow, for the time being, we got one more strong DLang UPB scholarship application literally today.

**Mike**: To close out, is there anything you'd like to say to people who'd like to help out?

**Andrei**: I'm very excited about this scholarship program and possible extensions to it. The reason for my excitement is that this is but a part of a larger strategy. Allow me to explain.

Up until now, we had no idea what to do with money even if we had it. A while ago, I met this potential donor who said, "OK, say I gave the Foundation half a million dollars over two years, no strings attached. What would you do with it?" To my own surprise, I had only vague answers. I asked Walter the same question, and he had even less of a clue than me.

So then I figured it's essential for the Foundation to have a strong response to that. I'm a big believer in the adage "luck helps the prepared", of which the converse is "luck is wasted on the unprepared". By that paradigm, not knowing what we'd do with money was a definite way to ensure we'd never be big. Now that we have the scholarship program, there exists a powerful reason for people to donate to the Foundation: donations help us find and support good students to work on high-impact D-related projects that push the state of CS systems research forward.

Another thing that would be great to have "donations" of is contributor time. Receiving more students starts pushing against our management capacity. Currently, and somewhat to my surprise, I am effectively a manager, seeing that all of these things I just gave you an earful of (bringing money in to the Foundation, managing bootcamp, finances, operations) take enough time to be a full-time job that leaves little time for coding. At some point, I won't be able to help everyone with their research, so I'll need to delegate some of that work to other folks. I'm talking any capacity here - from code reviews to managing to co-authoring papers to co-advising.

There are more things I have in mind, but it's early to share those. In brief, we need to organize ourselves for further growth. What's clear to me is we're no longer a seat-of-the-pants operation in a (virtual) basement. The D Language is exiting its adolescence.
