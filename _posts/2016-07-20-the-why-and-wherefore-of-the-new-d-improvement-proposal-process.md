---
author: DBlogAdmin
comments: false
date: 2016-07-20 13:18:08+00:00
layout: post
link: https://dlang.org/blog/2016/07/20/the-why-and-wherefore-of-the-new-d-improvement-proposal-process/
slug: the-why-and-wherefore-of-the-new-d-improvement-proposal-process
title: The Why and Wherefore of the New D Improvement Proposal Process
wordpress_id: 108
categories:
- Community
permalink: /the-why-and-wherefore-of-the-new-d-improvement-proposal-process/
redirect_from: /2016/07/20/the-why-and-wherefore-of-the-new-d-improvement-proposal-process/
---

When a programmer sees fellow programmers contributing code to open source projects, it is unlikely that the question of _why_ will ever come to mind in terms of motive. Regarding technical merit or usefulness, sure. But motive? Coders who are inclined to contribute bug fixes, new features, build systems, or other resources to open source projects do so for a variety of reasons, most of which other programmers will intuitively understand. But when a programmer volunteers to take over what is solely an administrative process for an open source project, that's a different story altogether.

Code contributions are usually one-off affairs. They require a time investment up front, but once the PR or patch is merged, that's often the end of it. Managing an administrative process, on the other hand, is an ongoing commitment. Time that could be spent coding is instead spent doing the proverbial paperwork. And the benefit to the volunteer may not always be clear. Implementing a ray tracer and going bungee jumping are things you might do for the hell of it. Administrative tasks? Not so much.

[The most recent post](https://dlang.org/blog/2016/07/13/the-dlang-vision-and-improvement-process/) on this blog highlighted two non-coding initiatives which were kicked off by members of the D community. One of them is a new process for handling D Improvement Proposals, or DIPs. This was the work of Mihails Strasuns. Not only did he put together a new process, he also volunteered to manage it. What on earth was he thinking?


There isn't anything special or unique about DIPs. Most programming languages that rely heavily on community input get something similar at some point. Once a language matures, changing even small bits of it becomes challenging and needs careful consideration. All of the relevant information has to be published somewhere.

The previous DIP process looked like a collection of articles on a community wiki. It seemed to work pretty well initially, when the DIP count was low enough, but with the number of improvement proposals reaching 100 and most of them still remaining in "draft" status, scalability issues became noticeable.


He highlights three issues specifically. First up is _quality control_.


There was no quality control for submitted proposals and, as a result, submissions were often lacking detailed information about language impact.


With the DIPs being Wiki-based, anyone with a D Wiki account could add a page for a new DIP, but there was no one designated to ensure that new DIPs met any sort of standard. This resulted in quality ranging from a detailed treatise that covered all the bases, to something that looked like notes on a cocktail napkin. Some remain in draft status, having never been finalized by their authors.

Another issue is the lack of response from the leadership.


It was very rare to get a formal response from Andrei or Walter regarding a DIP, partially because proper reviewing of incomplete proposals was demanding too much time.


Walter and Andrei already have their hands full. The last thing they need is to spend time reviewing a DIP that isn't ready for review. A DIP should be ready for their consideration before it is brought to their attention and not require them to ask for information that should already be there. Without a designated arbiter to decide when a DIP is complete, how are they to know? The author of a DIP may believe it ready, but all too often will be wrong.

Finally, there is the issue of maintaining the list of DIPs.


Abandoned or rejected proposals often were not marked as such because editing the list was purely a community effort.


With no one person responsible for maintaining the list, it was updated only when someone in the community noticed an update was needed and was willing to make the appropriate edits. The result was that there were only a small number of people who ever touched it.

So the solution Mihails came up with to address all of these problems, which he says is similar to that used with other modern programming languages, is described [in the README](https://github.com/dlang/DIPs) of the new github project he set up to replace the Wiki. It outlines the procedure, including how to submit a DIP and how to get it approved, gives advice on how to write a great DIP, and even outlines the responsibilities of the new DIP manager. Which, for now, is him.

And that brings us back to the topic of the first couple of paragraphs. We now know the problems he saw with the process and how he decided to improve it, but we still don't know _why_. So, why, Mihails?


There is, of course, a personal reason which caused me to step up with this process proposal. In a way, this is an attempt to solve a conflict of interest that arises from being employed by one of the largest D commercial users while being actively engaged in the language on my own. I want for the language to change and improve, but, at the same time, I have to ensure any such changes don't cause too much disruption to our systems at work.


That commercial user of which he speaks is [Sociomantic](https://www.sociomantic.com/), hosts of the excellent [DConf 2016](http://dconf.org/2016/index.html) in Berlin and maintainers of the [ocean ](https://github.com/sociomantic-tsunami/ocean)and [makd ](https://github.com/sociomantic-tsunami/makd)projects they opened up for the D community.


Having a formalized process for language changes helps a lot in that regard. It means there is a single information feed to follow if one doesn't want to miss anything of major impact. It's much more concise than a newsgroup or mailing list. And volunteering to handle the DIP bureaucracy allows me to make timely suggestions for better ways to handle any changes so that they don't cause too much trouble to existing code.


The DIP process arose organically to fill a void. Its was, perhaps, inevitable that its _ad hoc_ existence take another step in its evolution. It was just waiting for the right person with the right ideas and the willingness to make it happen. That person turned out to be Mihails Strasuns. The D communnity will be better off thanks to his efforts.
