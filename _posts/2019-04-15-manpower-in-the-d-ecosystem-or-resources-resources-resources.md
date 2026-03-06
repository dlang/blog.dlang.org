---
author: DBlogAdmin
comments: false
date: 2019-04-15 10:03:27+00:00
excerpt: There's lots of work in the D ecosystem waiting for someone to complete it.
  This blog post introduces two initiatives, the Human Resource Share and the Human
  Resource Fund, aimed at making that happen!
layout: post
link: https://dlang.org/blog/2019/04/15/manpower-in-the-d-ecosystem-or-resources-resources-resources/
slug: manpower-in-the-d-ecosystem-or-resources-resources-resources
title: Human Resources in the D Ecosystem (or Resources, Resources, Resources)
wordpress_id: 2033
categories:
- Community
- D Foundation
- Donations
permalink: /manpower-in-the-d-ecosystem-or-resources-resources-resources/
redirect_from: /2019/04/15/manpower-in-the-d-ecosystem-or-resources-resources-resources/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)

In November of last year, I announced here that we were [launching a Pull Request Manager campaign](https://dlang.org/blog/2018/11/10/the-new-fundraising-campaign/). We wanted to raise $3000 in three months as compensation for Nicholas Wilson to make a dent in the pull request queues of the core D projects. The community answered the call and Nicholas got to work. It was a successful run, so we decided to go another round.

About a month remains in the second round campaign, but donations have come in at a slower pace. Still, we’re confident the community will once again [help us reach the finish line](https://www.flipcause.com/secure/cause_pdetails/NDUwNTY=) so we can compensate Nicholas for his time. We intend to launch a third round at some point in the not-too-distant future, but before we do that we’ve got some different fish to fry.


### The Human Resource Share


In December 2018, the first Quarterly D Language Foundation meeting was held online. It came together at the prodding of the aforementioned Nicholas Wilson. Some well-known D shops—Dunhumby (Sociomantic), Funkwerk, WekaIO, and Symmetry Investments—were invited to send representatives to join Walter, Andrei, Ali Çehreli, Nicholas, and myself in a Skype call. Given that it was the inaugural meeting, the agenda was light. We primarily wanted to hear what the companies' biggest D issues were at the time so that we could prioritize them in the work pipeline. However, we did raise one idea that we wanted the companies to consider for our benefit.

A persistent problem in the D ecosystem is a lack of available human resources to work on the issues that don’t fall into the realm of personal or corporate interests. By that I mean that volunteers tend to contribute where they have a personal interest and contributions from the companies tend to be aimed almost exclusively at areas that have a direct benefit to their projects. The result is that a large number of issues that do not fit into either category fall by the wayside. This is only to be expected and we aren’t complaining. What we are doing is trying to determine how to direct energy toward those neglected issues without the need for raising money.

So we asked the companies to consider a form of “human resource sharing”, the idea being that each company would periodically designate one employee to spend a day on the company dime working on D ecosystem tasks that don’t necessarily have a direct impact on the company’s interests. The representatives promised to take it back to their bosses and give us an answer at the next meeting.

In the interim, I wrote up [a more concrete proposal](https://gist.github.com/mdparker/7e2894bef3e44daa1d1fac934a2c1aad) that outlined two options for approaching it: a monthly rotation where each company takes turns doing the work, and a quarterly system where each company commits to completing at least one item on the task list per quarter. I asked the companies to provide us with their preference at our second quarterly meeting in March.

At the March meeting, we invited a few more companies to join us. Given that these are either in the startup phase or aren’t using D exclusively, I’ll play it safe and keep quiet about who they are for now. Overall, the response to the Human Resource Share was positive. Unfortunately, most of the companies are already short on human resources as it is and cannot commit to our quarterly scheme. However, one company did commit to starting immediately and another committed to providing work as they can. All of them committed to helping in other ways, which includes the provision of funds (see below), and hope to have the human resources to spare in the future.

To that end, we’ve set up [the ecotasks repository](https://github.com/dlang/ecotasks) to house our Ecosystem Task List. The list was initially envisioned as a collection of specific tasks, e.g. specific Bugzilla issues, but that makes it more difficult for each tasked worker to decide what to do. Instead, we’ve cobbled together a set of task groups. For now, that consists primarily of links to the GitHub Issues page for different projects and a request to “close as many issues as possible”. The idea is that workers can go to an issues page and work on solving those they can squeeze into their allotted time.

We’ve put this on GitHub not just as a means of transparency, but also because we would like to invite the entire community to participate. The list is loosely sorted by priority in that items higher on the list are considered higher priority than those lower on the list, but there’s no relative priority between specific tasks. The ordering is sure to change over time.

We ask that anyone working through the list to, at the start of the work session, open an issue and leave a comment indicating which item is being worked on. For example:


I’m working on the dub registry issues right now. Specifically, I plan to tackle issues #I, #J, and #K.


Then, at the end of the work session, close the issue with a note indicating what was accomplished.

If you have a little time to spare one weekend, please consider visiting the ecotasks repository and taking on one or two issues. Even better, challenge yourself to go through it once a month and see what you can accomplish. D depends on volunteer effort to thrive. We have a lot of it already, but we always need more. This is one of many ways to make an impact even if it isn’t an enjoyable or very visible one.


### The Human Resource Fund


There are some tasks in the D ecosystem that no amount of cajoling and begging will get done because they’re too complex, too time consuming, require a specific skill set to properly complete, or all of the above. When the companies offered to throw money at us in place of human resources, that led us to a new idea.

We are now running a permanent fundraising campaign specifically aimed at solving the bigger issues. [The Human Resource Fund for D Ecosystem Tasks](https://www.flipcause.com/secure/cause_pdetails/NTUxOTc=) is intended to grow and grow and grow. We’re currently in talks with some of the companies about how often and how much they can contribute toward it and in what amounts. We also invite the community at large to donate to it now and again.

Recently, Andrei mentioned in the forums that we need to put together a qualified team to complete the spec and implementation of `shared`. That’s an example of the sort of big issue we want to use this fund to solve. Donations small and large are equally welcome. The sooner we can get a nice pile built up, the sooner we can start prioritizing issues and finding the people to solve them.

For now, we want to focus on beefing this fund up a bit so we're going to hold off temporarily on the next Pull Request Manager campaign, but we'll definitely come back to that again before too long. Anyway, the current campaign is still [in need of your attention](https://www.flipcause.com/secure/cause_pdetails/NDUwNTY=)!


### Effecting Change


Every D user has different priorities and goals, different needs and desires. A full-featured IDE is important to one person but not even worth mentioning for another. One programmer expects to see a native D GUI, another is happy with bindings to an existing C or C++ library, and yet another has no need of a GUI at all. One person contributes to a certain D project, but never to any others, while another person has a different set they contribute to, or starts their own. Somewhere in the middle are the issues that are too boring or too complex, the issues that never rise to anyone’s attention or are considered undoable for whatever reason.

In the time I’ve been following and involved with D, I’ve seen the leadership try a number of different ways to drive energy toward some of these unsolved issues. What every approach they’ve tried has had in common is that _they depend on the community_. When you don’t have the human resources to do the job, you need the money to hire the human resources. When you don’t have the money, you need to ask for and rely upon the charity and goodwill of others. And if there's no one with the bandwidth to continuously push the issue, fewer people step up. When the community doesn’t step up, then either someone on the core team has to (at the expense of time taken from their normal workload) or the issue languishes.

The two initiatives I’ve described above, the Human Resource Share and the Human Resource Fund, are the latest attempts to make things happen. Again, all contributions are welcome and appreciated! I don’t want this to come off as a complaint, because that’s not what is intended at all.

This is a call to arms! We’re asking members of the community to roll up their sleeves and do the dirty work they normally wouldn’t think to do, or even would prefer not to do. We’re asking for crowdsourced effort in solving problems that will make the D ecosystem better for all of us. A few dozen people spending an hour here or a weekend there will mean more issues closed and more members of the core team can stay focused more often on the work in their purview, which is another big win.

If you can’t help us out with your time, help us out with your money! [The Human Resource fund](https://www.flipcause.com/secure/cause_pdetails/NTUxOTc=) will always need boosting. So, too, [the General Fund](https://www.flipcause.com/secure/cause_pdetails/NDMzMzE=), the current [PR Manager campaign](https://www.flipcause.com/secure/cause_pdetails/NDUwNTY=), and any other campaigns we launch in the future. You can also support us when you shop at Amazon by doing so via [smile.amazon.com](https://smile.amazon.com/gp/chpf/about/ref=smi_se_rspo_laas_aas) and selecting “D Language Foundation” as your supported charity. When you buy products marked “Eligible for AmazonSmile donation” through [smile.amazon.com](https://smile.amazon.com/), the foundation will receive 0.5% of the purchase price. Finally, you’ll soon be able to support D through the DLang Swag Emporium, where you’ll be able to purchase D-themed t-shirts, coffee mugs, and more.

We’re always open to ideas on how to get more things done. If you have anything you’d like to suggest, bring it to the forums for community discussion or email me directly at [aldacron@gmail.com](mailto:aldacron@gmail.com).

Now, let’s make things happen!
