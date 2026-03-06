---
author: DBlogAdmin
comments: false
date: 2018-02-17 12:46:19+00:00
excerpt: Sometimes projects are abandoned. Sometimes they aren&#8217;t updated as
  frequently as users would like. This can become an issue for those who depend upon
  these projects, but it&#8217;s alleviated by the fact that most D projects are open
  source and their repositories are publicly available. All it takes to keep a project
  alive and up-to-date are more volunteers willing to pitch in. That&#8217;s the motivation
  behind the <a href="https://tour.dlang.org/">dlang-community organization</a> at
  GitHub.
layout: post
link: https://dlang.org/blog/2018/02/17/project-highlight-the-d-community-hub/
slug: project-highlight-the-d-community-hub
title: 'Project Highlight: The D Community Hub'
wordpress_id: 1381
categories:
- Community
- Compilers &amp; Tools
- Project Highlights
permalink: /project-highlight-the-d-community-hub/
redirect_from: /2018/02/17/project-highlight-the-d-community-hub/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)

As [has been stressed](https://dlang.org/blog/2018/02/03/the-dbugfix-campaign/) on [this blog before](https://dlang.org/blog/2016/06/07/the-d-website-and-you/), D is a community-driven language. Because of that, the ecosystem depends on the work of volunteers who are willing to contribute their time and open their projects to the community at large. From [IDE](https://wiki.dlang.org/IDEs) and [editor](https://wiki.dlang.org/Editors) plugins to libraries in [the DUB registry](https://code.dlang.org/), it’s all about the efforts of people who are (usually) getting no monetary reward for their efforts.

There are some inherent downsides to that reality. Sometimes projects are abandoned. Sometimes they aren’t updated as frequently as users would like. This can become an issue for those who depend upon these projects, but it’s alleviated by the fact that most D projects are open source and their repositories are publicly available. To keep a project alive and up-to-date only requires more volunteers willing to pitch in.

That’s the motivation behind the [D Community Hub](https://github.com/dlang-community) (dlang-community) at GitHub. According to Sebastian Wilzbach, it started with Brian Schott’s popular tools used by several IDE and editor plugins:


There were maintenance issues with Brian’s (aka Hackerpilot) awesome projects. He has a full-time job and often could only respond to simple issues every few weeks. This meant that simple bug fix PRs sat in the queue for quite a while. For example, there was one case where the same PR to fix the Windows build script was submitted by three different people (there was no Windows CI at the time).


Brian’s projects weren’t the only ones that motivated the idea. Sebastian and Petar Kirov maintain the [DLang Tour](https://tour.dlang.org/), and some of the projects they depend upon were either inactive or slow to update. However, Brian’s tools are widely used, so they started with him. Eventually, they convinced him to move some of his projects to the new organization and others followed.

Sebastian lays out the following benefits that have come from moving multiple projects from disparate developers under an umbrella group:



 	
  * Common best policies (e.g. all repositories have GitHub branch protection)

 	
  * No need to fork an inactive repository - work can be shared easily.

 	
  * No dependence on a single person who might be busy or on vacation (this is especially important for swiftly pulling and releasing bug fixes )

 	
  * One common location whenever updates are required (e.g. package bumps or deprecation fixes)

 	
  * Many of the projects are enabled on the Project Tester (their test suite is run on every PR for the [DMD, DRuntime, Phobos, Dub, and tools repositories](https://github.com/dlang) to prevent regressions) - this is possible because many people have merge rights in case an improvement in the compiler finds critical bugs or deprecations are moved forward

 	
  * Shared knowledge (e.g. all projects support “auto-merge” like the dlang repositories)

 	
  * Automation with bots - Mark Rz (@skl131313) created a bot that automatically triggers update PRs whenever dependencies are updated (some of the projects in dlang-community still support builds with only git submodules and make)

 	
  * Less overhead for automation with CIs (everyone can connect a repo to a third-party provider or restart a failing CI job)


It has also resulted in increased participation. For example, other D users have joined the group, and [Sociomantic Labs](https://www.sociomantic.com/) (the D shop in Berlin that hosted the 2016 and 2017 editions of [DConf](http://dconf.org)) has [taken over the release process](https://github.com/dlang-community/dfmt/issues/320) for dfmt, Brian’s tool for formatting D source code.

There are currently 22 repositories in the dlang-community organization, including the following:



 	
  * [DCD](https://github.com/dlang-community/DCD) (the D Completion Daemon) - an autocomplete program that is used by several D [IDE](https://wiki.dlang.org/IDEs) and [editor](https://wiki.dlang.org/Editors) plugins

 	
  * [dfmt](https://github.com/dlang-community/dfmt) - a formatter for D source code, also used by many IDE and editor plugins

 	
  * [D-Scanner](https://github.com/dlang-community/D-Scanner) - a tool for analyzing D source code

 	
  * [dfix](https://github.com/dlang-community/dfix) - a tool for automatically upgrading D source code

 	
  * [libparse](https://github.com/dlang-community/libdparse) - a library for lexing and parsing D source code

 	
  * [drepl](https://github.com/dlang-community/drepl) - a DMD-based REPL for D

 	
  * [stdx-allocator](https://github.com/dlang-community/stdx-allocator) - a frozen version of [`std.experimental.allocator`](https://dlang.org/phobos/std_experimental_allocator.html) (which is due for an overhaul)

 	
  * [containers](https://github.com/dlang-community/containers) - a set of containers backed by `stdx.allocator` to easily switch between different allocation strategies, with or without the GC

 	
  * [D-YAML](https://github.com/dlang-community/D-YAML) - A YAML parser and emitter

 	
  * [harbored-mod](https://github.com/dlang-community/harbored-mod) - a documentation generator that supports both [D’s built-in Ddoc syntax](https://dlang.org/spec/ddoc.html) and Markdown


In addition to other D projects, there’s a repository set up specifically to [discuss the dlang-community organization](https://github.com/dlang-community/discussions) via GitHub issues, and repositories that contain artwork. If you decide to use any of these projects, the discussion repository is the place to ask for help when you need it.

Other projects may be added in the future. According to Sebastian, there are a few questions that form a set of loose criteria for inclusion under the dlang-community umbrella.



 	
  * Is there enough interest from the general public so that it is “worth maintaining”?

 	
  * Is there a similar library with active development out there?

 	
  * Is at least one DLang community member competent for the domain covered by the project? If no, is there anyone who’s willing to fill the role?


Sebastian and the others are looking to add a few features over time. These include:

 	
  * More automatic documentation builds

 	
  * Automatic build of binaries on new tags (especially for Windows)

 	
  * d-apt: Sociomantic is working on moving [d-apt](https://github.com/dlang-community/d-apt) to GitHub and enabling full automatic CI builds for it.

 	
  * dfmt: Leandro Lucarella / Sociomantic is introducing [neptune](https://github.com/sociomantic-tsunami/neptune) and a proper release process


For anyone interested in joining the dlang-community organization, there are two options. If you are already a well-known participant in the D community, simply ping one of the existing members for merge rights. For anyone else, the best approach is to start contributing to one or more of the dlang-community projects to build up trust. At some point, frequent trustworthy contributors will be welcomed into the fold.

As for the current contributors, Sebasitian says:


There are many people working behind the scenes on the dlang-community libraries. A special thanks goes to the active reviewers who make it possible that your potential PR gets merged in a timely manner.





 	
  * Basile Burg

 	
  * Brian Schott

 	
  * Jan Jurzitza

 	
  * Leandro Lucarella

 	
  * Martin Nowak

 	
  * Petar Kirov

 	
  * Richard Andrew Cattermole

 	
  * skl131313

 	
  * Stefan Koch


If you have or know of a D project that is suffering from a lack of attention, bringing it to the dlang-community might be the way to breathe new life into it. Don’t be shy in [asking for help](https://github.com/dlang-community/discussions).
