---
author: DBlogAdmin
comments: false
date: 2016-06-10 18:18:42+00:00
layout: post
link: https://dlang.org/blog/2016/06/10/core-team-update-vladimir-panteleev/
slug: core-team-update-vladimir-panteleev
title: 'Core Team Update: Vladimir Panteleev'
wordpress_id: 24
categories:
- Core Team
- News
permalink: /core-team-update-vladimir-panteleev/
redirect_from: /2016/06/10/core-team-update-vladimir-panteleev/
---

When Walter Bright unleashed the first public release of DMD on the world, the whole release and maintenance process surrounding the language and the compiler was largely a one man show. Today, there are a number of regular contributors at every step of the process, but at the heart, in addition to Walter, are three core team members who keep the engine running. One of them is Vladimir Panteleev.

Vladimir maintains [a number of D projects](https://github.com/CyberShadow?tab=repositories) that have proven useful to D users over the years. Today's entry focuses on one that had as much of an impact as any other D project you could name at the time upon its release, if not more.

If the name [DFeed](https://github.com/CyberShadow/DFeed) isn't familiar to you, it's the software that powers the [D Forums](http://forum.dlang.org/). Posts pop up now and again from new users (and even some old ones!) asking for features like post editing or deletion, or other capabilities that are common in forum software found around the web. What they don't realize is that DFeed is actually not the traditional sort of forum package they may be familiar with.

Go back a few years and the primary vehicle powering D discussions was an NNTP server that most people accessed via newsreaders. Over time, there were additional interfaces added, as Vladimir points out:


D only had an NNTP news server, a mailing list connected to it, and two shoddy PHP web interfaces for the NNTP server, both somehow worse than the other. A lot of people were asking for a regular web forum, such as phpBB. However, Walter would always give the same answer - they were inferior in many aspects to NNTP, thus he wouldn't use them. What's the point of an official forum if the community leaders don't visit them?


Vladimir decided to take the initiative and do something about it. He took a project he had already been working on and enhanced it.


The program was originally simply an IRC bot (and called "DIrcFeed"), which announced newsgroup posts, GitHub activity, Wikipedia edits and such to the #d channel on FreeNode. Adding a local message cache and a web interface thus turned it into a forum.


And so DFeed was born. [Walter announced](http://www.digitalmars.com/d/archives/digitalmars/D/dfeed_151518.html) that it had gone live on Valentine's Day 2012. It even managed to get [a positive reddit thread](https://www.reddit.com/r/programming/comments/ppre5/the_new_d_online_forum_software_written_in_d/) behind it, which was a big deal for D in those days. The [NNTP server](news://news.digitalmars.com) and [the mailing list interface](http://lists.puremagic.com/) to it are still alive and well, but it's probably a safe assumption that a number of users eventually ditched their newsreaders for DFeed's web interface and never looked back. I know I did.

So if you've ever wondered why you can't delete your posts in the D Forums, you now have the reason. DFeed does not have the authoritative database. That belongs to the NNTP server. To illustrate two major problems this brings about, consider the idea of adding Markdown support to DFeed.


First, people using NNTP/email won't see the rendered versions. Which isn't a big deal by itself since it's just text, but does create feature imparity. It *is* possible to write Markdown that looks fine when rendered but is unreadable in source form, especially with some common extensions such as GitHub Flavored Markdown.

Second, unless we're careful with this, people using NNTP/mailing lists might trigger Markdown formatting that could make their post unreadable. This could be avoided, though, by only rendering messages with Markdown if they originate from the web interface, which allows previewing posts.


Even so, he's hoping to add support for Markdown at some point in the future. Another enhancement he's eyeing is optional delayed NNTP/email posting.


A lot of younger people communicate mainly through web forums, and are very used to being able to edit their post after sending it. This is not supported on forum.dlang.org for the simple reason that you can't unsend an email. One solution to this problem is to save all messages sent from the web interface locally, but delay their propagation to NNTP/email for a few minutes. This creates a window during which the message can still be edited, or even deleted.


Other features coming in a future update are [OAuth ](http://oauth.net/)support, a thread overview widget, and performance improvements. He says DFeed isn't as fast as it used to be. He wants to change that. Don't look for the new version of DFeed too soon, though. Right now, Vladimir's attention is turned in directions that he believes will have a bigger impact on D. As soon as I know what that means, you'll read about it here.

In the meantime, if you have any ideas on how to improve the forums, keeping in mind that any new features have to play nice with both the NNTP server and the mailing list, don't hesitate [to bring it up](http://forum.dlang.org/group/general).

Thanks to Vladimir for taking the time to provide me with an update and for maintaining DFeed over the years. Here's to many more!
