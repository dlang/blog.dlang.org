---
author: DBlogAdmin
comments: false
date: 2019-06-04 12:24:20+00:00
layout: post
link: https://dlang.org/blog/2019/06/04/revisions-to-the-dip-process/
slug: revisions-to-the-dip-process
title: Revisions to the DIP Process
wordpress_id: 2071
categories:
- DIPs
permalink: /revisions-to-the-dip-process/
redirect_from: /2019/06/04/revisions-to-the-dip-process/
---

At [the AGM that was held](https://youtu.be/cpTAtiboIDs) prior to the Hackathon at [DConf 2019 in London](https://dconf.org/2019/index.html), I announced that I would be making revisions to the DIP progress aimed at shortening the length of time required to go from the Community  Review to a final verdict. I also, in response to Joseph Rushton Wakeling's feedback about guidance for reviewers, agreed to enhance the existing documentation to clarify what is expected of reviewers during the Community and Final Review stages and to provide guidance on how to provide a good review.

The new documentation is now live in four documents under the new `docs` subdirectory in [the DIP repository](https://github.com/dlang/DIPs) (all of which are linked in the README). The PROCEDURE and GUIDELINES documents are still there so that existing links remain valid, but all of their content has been replaced with links to the new documentation. Please consider the new documentation to be in draft form. I have not yet subjected them to intense editing, so any corrections are welcome, as are suggestions on how to enhance them.

Anyone intending to participate in a DIP review by leaving feedback in a review thread should familiarize themselves with the [DIP Reviewer Guidelines](https://github.com/dlang/DIPs/blob/master/docs/guidelines-reviewers.md). I can't force anyone to do so, but I do consider this mandatory. In my role as DIP manager, trying to summarize long threads that have gone off on in-depth discussions of one thing or another, reading through post after post for any sign of actual feedback, is a time-consuming (and highly annoying!) process. Henceforth, I will be deleting any posts in Community and Final review discussion threads that do not adhere to the guidelines laid out in the above document. As I declare in the document, such posts will be copied and pasted in a separate thread where off-topic discussions of the DIP may continue. It's not my intention to stifle debate or censor anyone or in any other ray restrict discussion of the DIP--I just want to make my job, and the DIP author's, easier. So please, for my sake and yours, read and understand [the reviewer guidelines](https://github.com/dlang/DIPs/blob/master/docs/guidelines-reviewers.md).

The content of the GUIDELINES document, which was targeted at DIP authors, has been moved (and slightly modified) into the [DIP Author Guidelines](https://github.com/dlang/DIPs/blob/master/docs/guidelines-authors.md). The portion of the PROCEDURE document that was aimed at DIP authors is now in [The DIP Authoring Process](https://github.com/dlang/DIPs/blob/master/docs/process-authoring.md) document. All potential DIP authors should read and understand both documents before submitting a DIP. Failure to do so may result in surprises. For example, I'm going to be more proactive in closing pull requests that are submitted while a DIP is still in development and not yet in a first draft state. So please read!

Finally, the portion of the PROCEDURE document that described the different review stages is now found in the document titled [The DIP Review Process](https://github.com/dlang/DIPs/blob/master/docs/process-reviews.md). Everyone should read this. The primary difference from the previous document is that I've explicitly declared that Community Reviews will always begin in the first seven days of a month and Final Reviews will always begin in the third week of a month. Additionally, where I would formerly allow multiple Community Reviews to take place simultaneously, I now restrict it to one at any given time. The goal is to streamline the process and minimize the time it takes to go from Community Review to a final verdict.

As the new document outlines, the best-case scenario, in which only one round of Community Review is required and no DIPs are in active consideration of the language maintainers when another DIP finishes the Final Review, should look like this:



 	
  * the DIP enters Community Review in the first week of Month A.

 	
  * after Community Review, the DIP author will have four weeks to complete any required revisions.

 	
  * in the third week of Month B, the Final Review begins.

 	
  * after the Final Review, no revisions are required and no other DIP is under active consideration, so the DIP may immediately move into Formal Assessment.

 	
  * the language maintainers have enough information to render a verdict on the DIP within 30 days.


So it should take between two and three months for the review process to complete. Again, this is the best-case scenario. I expect it more likely that it will typically take between four and five months, given that some DIPs will need multiple Community Review rounds and some will require revision during Formal Assessment.

As I said at the AGM, I'm always open to improving the process to the extent I can within the boundaries of the current framework. Any fundamental structural changes will need approval by Walter and Átila. If you have any suggestions to strengthen the documentation, please let me know.
