---
author: DBlogAdmin
comments: false
date: 2020-01-26 08:54:00+00:00
layout: post
link: https://dlang.org/blog/2020/01/26/dip-reviews-discussion-vs-feedback/
slug: dip-reviews-discussion-vs-feedback
title: 'DIP Reviews: Discussion vs. Feedback'
wordpress_id: 2285
categories:
- Community
- DIPs
- News
permalink: /dip-reviews-discussion-vs-feedback/
redirect_from: /2020/01/26/dip-reviews-discussion-vs-feedback/
---

![Digital Mars D logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)For a while now, I've been including [a link to the DIP Reviewer Guidelines](https://github.com/dlang/DIPs/blob/master/docs/guidelines-reviewers.md) in the initial forum post for every DIP review. My hope was that it would encourage reviewers to keep the thread on topic and also to provide more focused feedback. As it turns out, a link to reviewer guidelines is not quite enough. Recent review threads have turned into massive, 20+ page discussions touching on a number of tangential topics.

The primary purpose of the DIP review process, as I've tried to make clear in blog posts, forum discussions, and the reviewer guidelines, is to improve the DIP. It is not a referendum on the DIP. In every review round, the goal is to strengthen the content where it is lacking, bring clarity and precision to the language, make sure all the bases are covered, etc.

At the same time, we don't want to discourage discussion on the merits of the proposal. Opinions about the necessity or the validity of a DIP can raise points that the language maintainers can take into consideration when they are deciding whether to approve or reject it, or even cause the DIP author to withdraw the proposal. It's happened before. That's why such discussion is encouraged in the Community Review rounds (though it's generally discouraged in Final Review, which should be focused wholly on improving the proposal).


### The problem


One issue with allowing such free-form discussion in the review threads is that there is a tremendous amount of noise drowning out the signal. Finding specific DIP-related feedback requires trawling through every post, digging through multiple paragraphs of mixed discussion and feedback. Sometimes, one or more people will level a criticism that spawns a long discussion and results in a changing of minds. This makes it time consuming for me as the DIP manager when I have to summarize the review. It also increases the likelihood that I'll overlook something.

My summary isn't just for the 'Reviews' section at the bottom of the DIP. It's also my way of ensuring that the DIP author is aware of and has considered all the unique points of feedback. More than once I have found something the DIP author missed or had forgotten about. But if I overlook something and the DIP author also overlooks it, then we may have missed an opportunity to improve the DIP.

I have threatened to delete posts that go off topic in these threads,  but I can count on one hand the number of posts I've actually deleted. In reality, these discussions branch off in so many directions that it's not easy to say definitively that a post that isn't focused on the DIP itself is actually off topic. So I tend to let the posts stand rather than risk derailing the thread or removing information that is actually relevant.


### The Solution


Starting with the upcoming Final Review of [DIP 1027](https://github.com/dlang/DIPs/blob/dd8c17c5ebaea6cecbecba9e2ef91db813a2191c/DIPs/DIP1027.md), I'm going to take a new approach to soliciting feedback. Rather than one review thread, I'll be launching two for each DIP.

The Discussion Thread will be much the same as the current review thread. Opinions and discussion will be welcome and encouraged. I'll still delete posts that are completely off topic, but other than that I'll let the discussion flow where it may.

The Feedback Thread will be exclusively for feedback on the document and its contents. There will be **no discussion allowed**. Every post must contain specific points of feedback (preferably actionable items) intended to improve the proposal. Each post should be a direct reply to my initial post. There are only two exceptions: when a post author who has decided to retract feedback they made in a previous post, said poster can reply to the post in which they made the original feedback in order to make the retraction; and the DIP author may reply directly to any feedback post in order to indicate agreement or disagreement.

Posts in the feedback thread should contain answers to the questions posed in [the DIP Reviewer Guidelines](https://github.com/dlang/DIPs/blob/master/docs/guidelines-reviewers.md). It would be great if reviewers could take the time to do what Joseph Rushton Wakeling did in the Community Review for DIP 1028, where [he explicitly listed and answered each question](https://forum.dlang.org/post/tkjjspryyjiqzqnpclvb@forum.dlang.org), but we won't be requiring it. Feedback as bullet points is also very welcome.

Opinions on the validity of the proposed feature will be allowed in the feedback thread as long as they are backed with supporting arguments. In other words, "I'm against this! This is a terrible feature." is not valid for the feedback thread. That sort of post goes in the discussion thread. However, "I'm against this. This is a terrible feature because <reasoned argument goes here>" is acceptable.

The rules of the feedback thread will be enforced without prejudice. Any post that is not a reply to my initial post, retraction of previous feedback, or a response by the DIP author will be deleted. Any post that does not provide the sort of feedback described above will be deleted. If I do delete a post, I won't leave a new post explaining why. I'm going to update the DIP Reviewer Guidelines and each opening post in a feedback thread will include a link to that document as well as a paragraph or two summarizing the rules.

I'll require DIP authors to follow both threads and to participate in the discussion thread. When it comes time to summarize the review, the feedback thread will be my primary source. I will, of course, follow the discussion thread as well and take notes on anything relevant. But if you want to ensure any specific criticisms you may have about a DIP are accounted for, be sure to post them in the feedback thread.

Hopefully, this new approach won't be too disruptive. We'll see how it goes.


