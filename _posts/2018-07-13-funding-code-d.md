---
author: DBlogAdmin
comments: false
date: 2018-07-13 14:08:10+00:00
excerpt: The D Language Foundation hit its goal of raising $3000 to fund development
  on the D plugin for Visual Studio Code, code-d, and its supporting tools. This post
  explains where that money is going and talks about the future of the Foundation's
  ecosystem funding initiative.
layout: post
link: https://dlang.org/blog/2018/07/13/funding-code-d/
slug: funding-code-d
title: Funding code-d
wordpress_id: 1628
categories:
- Community
- D Foundation
- Donations
permalink: /funding-code-d/
redirect_from: /2018/07/13/funding-code-d/
---

At the [DConf 2018 Hackathon](https://youtu.be/xNWRgEHxOhc), I announced that the D Language Foundation would be raising money to further the development of [the suite of tools comprising code-d](https://github.com/Pure-D), the [Visual Studio Code plugin](https://code.visualstudio.com/). The project is developed and maintained by [Jan Jurzitza, a.k.a. Webfreak001](https://github.com/WebFreak001).

![](https://avatars3.githubusercontent.com/u/2035977?s=460&v=4)

The plan was that if we reached $3,000 in donations [on our Open Collective page](https://opencollective.com/dlang#), we’d make the money available to Jan. When we finally reached the goal, I was quite happy to tweet about it:




> 
> Thanks to a burst of donations in the past few days, we've hit our goal at the [#dlang](https://twitter.com/hashtag/dlang?src=hash&ref_src=twsrc%5Etfw) Open Collective to support the code-d/serve-d for VS Code. Great work, everyone! Details to come. [https://t.co/T1KtXJS7gL](https://t.co/T1KtXJS7gL)
> 
> 
— D Language (@D_Programming) [July 1, 2018](https://twitter.com/D_Programming/status/1013270678416420864?ref_src=twsrc%5Etfw)




Jan and I still hadn’t finalized the details at that point, but we finally got it all sorted a few days later. Before I show how we’ll be using the money, I’d first like to talk about why we’re doing this.


## The motivation


As the D programming language community has grown, so has the amount of work that needs to be done. Since D is a community-driven language, much of the work that gets completed is done by volunteers. Unfortunately, the priorities for those who actively contribute aren’t necessarily aligned with those of everyone else, and this can lead to some users feeling that there is little progress at all, or that their concerns are being deliberately ignored. That’s just the nature of the beast. More potential pain points and unfulfilled wishlists means there’s a continual need for more manpower.

Therein lies one of the most fundamental issues that has plagued D’s development for years: how can more people be motivated to directly participate so we can keep pace with the consequences of progress?

To be clear, everyone involved in the core team is aware that lack of manpower is not the only negative side effect we suffer from growth, but it is certainly a big one. Part of my role these days is to figure out, along with Sebastian Wilzbach, ways to mitigate it. My inbox is littered with conversations exploring this topic, along with recommendations from Andrei to research how other projects handle one thing or another for inspiration.

To that end, in early February [I launched the #dbugfix campaign](https://dlang.org/blog/2018/02/03/the-dbugfix-campaign/) with the goal of increasing community participation in the bug fixing process by providing a simple way for folks to bring attention to their pet peeve issues (keep it going people – participation has slowed and we need to pick it up!). We also announced [the State of D Survey](https://dlang.org/blog/2018/02/28/the-state-of-d-2018-survey/), initiated by Seb, at the end of February to get an idea of what some of the personal pain points are and where people think things are going swell. We’ve got other ideas in development, one of which I’ll be telling you about in the coming days.

Among the data that jumped out at us from the survey is that Visual Studio and Visual Studio Code were the most popular IDEs/editors among participants. The plugins for both [Visual Studio (Visual D)](http://rainers.github.io/visuald/visuald/StartPage.html) and [VS Code (code-d/serve-d/workspace-d)](https://github.com/Pure-D) are each maintained by solo developers.

That’s a heavy workload for each of them. [Look at the issues list for code-d](https://github.com/Pure-D/code-d/issues) and you’ll find several there (39 open/132 closed as I write), but [the PR list is much shorter](https://github.com/Pure-D/code-d/pulls) (1 open/21 closed as I write). What I infer from that is that the demand for improvements is higher than the supply for implementing them. And from personal experience, I know there are projects like code-d whose maintainers would love to add new features, or improve existing ones, but simply can’t (or aren’t motivated to) make the time for it.

Here then is a good place for a little experiment. What if the D Language Foundation started raising money to help fund the development of specific projects like code-d? By creating the opportunity for those with extra money but little time to directly contribute toward fixing their personal pain points and shrinking their wishlists, can development be focused toward getting bugs fixed and improvements implemented more quickly?

Long story short, I picked the VS Code plugin for our trial run because it was prominent in the survey data, it’s cross-platform, and the serve-d component can be beneficial to other IDE/editor plugins. Open Collective launched their goal system just as I was trying to determine how best to go about it, so it seemed the obvious choice (though we didn’t learn until after our announcement that it wasn’t the system we thought it was, i.e. it’s not a system of multiple, targeted fund raising goals but instead a milestone on the [global donation progress bar at the top of our Open Collective page](https://opencollective.com/dlang#)). I contacted Jan just before DConf to gauge his interest, then finally had a meeting with the Foundation board the night before the Hackathon to get approval and decide how to go about doling out the money (we decided the money should be paid out in increments based on completion of milestones, with the option to provide money as needed for targeted development costs, e.g. a Windows license, bug bounties, etc). It all came together fairly quickly.

We plan to continue this initiative and raise money for more work in the future. Some will be like this trial run, with the goal of motivating project maintainers to focus development and spend that extra hour or two a day to get things done. Some will be aimed at specific development tasks, as bounties or as payment for a single part-time or full-time developer. We’ll experiment and see what works.

We won’t be using Open Collective's goals for this (or any other service) going forward. Instead, we’re going to add a page to the web site where we can define targets, allow donations through Open Collective or PayPal, and track donation progress. Each target will allow us to lay out exactly what the donations are being used for, so potential donors can see in advance where their money is going. We’ll be using the State of D Survey as a guide to begin with, but we’ll always be open to suggestions, and we’ll adapt to what works over what doesn’t as we go along.

As long as there are people willing to put either time or money into the D language, community, and ecosystem, this will surely pay off in the long run with more visible progress, hopefully solving a wider range of pain points than currently happens in our “what-I-want-to-work-on” system of community development. We’re looking forward to see the synergy that results from this and other initiatives going forward.


## The milestones


Jan has some specific ideas about how he wants to improve his plugin tools and was happy to take this opportunity to put them all front and center. I asked him to make a list, then estimate how much time it would take to complete each task if he were working on it full time, then divide that list into roughly equal milestones of 3–4 weeks each. That does not mean he will be working full time, nor that each milestone will take 3–4 weeks; payment is not dependent on deadlines. It was simply a means of organizing his task list.

We ended up with two sets of tasks as a starting point and agreed to a $500 payout upon completion of both milestones. Once all the tasks in a milestone are completed, the $500 will be paid. At the end of the second milestone, we’ll determine what to do next – another milestone of feature improvements, a heavy round of bug fixing, or something else – and how to make use of the remaining $2000.

Following is a list of the tasks in both of the initial milestones, with a brief description of each directly from Jan (lightly edited).


### Improvements & maintainence





 	
  * **add automated tests for windows 7, 10, archlinux, ubuntu, fedora, osx for startup, from scratch project initialization, reloading, adding dependencies**  
There are automated tests in the workspace-d repo which test different scenarios and combinations with dub, DCD, and d-scanner, ranging from general usage to weird setups. Identifying more issues (especially on windows), creating more tests and especially running all the tests on several platforms would be a good first step towards a less buggy version of serve-d which should work on all platforms equally well.

 	
  * **Single file mode**  
A requested feature and one which makes the usage of the plugin a lot easier would be single file usage. Currently, single file usage is very limited and without an open workspace it doesn’t work at all. Adding this (with quick startup time and low resource usage) will make code-d a good alternative to vim or notepad for quickly editing a D file.

 	
  * **fix unknown DDoc macro parsing**  
Currently, if any unknown DDoc macros are used, they are omitted from the documentation renderer, which makes some documentation unreadable and weird. For this, libddoc needs to be worked on by identifying where exactly the macros are discarded, how they could be kept track of, and how to make it more obvious in the API without breaking backwards compatibility.

 	
  * **project template creator as separate plugin (with standardized template format if possible)**  
Currently the “Create Project” command is a very static command reading from a directory in the plugin. Making this a separate VS Code plugin with an API for other plugins to extend with more functionality would make it more flexible and open it to new platforms and APIs. Also, research on if there are already some standards for this and if we can parse other template formats.

 	
  * **detect used imports & add command to remove unused ones & add Remove unused imports + sort command**  
Add a way to find out if imports are used and add a command to remove unused ones. DCD could probably be extended for this.

 	
  * **code folding on grammar**  
The expand/collapse feature on arrays & functions (bracket based) and lambdas. (Instead of indentation based)

 	
  * **parse dub commandline arguments**  
Dub command line arguments should be able to be passed into the build buttons and future tasks. This will require parsing them because dub is used as an API and not as a program.

 	
  * **experiment with adding a mode to DCD to make all symbols available**  
A mode to always give all public symbols regardless of imports and additionally provide the import where the symbol comes from (so when completing it, the IDE adds an import)




### UX & Tools for beginners





 	
  * **improve UI/UX for building and running projects (implement tasks api) **  
As the tasks API is now actually released, I can incorporate the different build types into the task system of VS Code. This would include having `build` (`debug`, `release`), `run`, `test` for each detected dub project and submodule.

 	
  * **make workspace symbol search show more symbols **  
The workspace symbol search (`Ctrl-P -> #`) currently is using just DCD to find exact matches for types. I would include DScanner ctag/etag results in this response to make the search better.

 	
  * **code action: improve implement interface (parse existing functions and not include them)**  
Test cases need to be written, existing interfaces must be parsed, cross-file it should all still work, inherit documentation, etc. Need to check in real world applications where interfaces and classes are used. Also add support for abstract classes.

 	
  * **auto complete in diet templates (both HTML tags and D code)**  
Add static HTML auto completion for tags & attributes and create a virtual D file for D code for DCD for autocompletion.


 	
  * **code action: create getter/setter for variable**  
For code like `int _foo;`, create the property functions `int foo() @property const { return _foo; } int foo(int value) @property { return _foo = value; }` with automatic detection if `const` can be used. This makes writing classes easier (snippets already exist, but this is more intuitive) and shows beginners how properties are created. The property should be added where other properties are. It could check the comments for “Getters”, “Setters” and “Properties” if possible, otherwise just put it after a block of members.

 	
  * **code action: convert boolean parameter to Flag (+import)**  
This should change the definition of a function `void foo(bool createDirectory)` to `alias CreateDirectory = Flag!"createDirectory"; void foo(CreateDirectory createDirectory)` and add the appropriate `import` if it doesn’t exist yet.

 	
  * **code action: make foreach parallel (with import if neccessary)**  

Replace a `foreach (a; array)` with `foreach (a; parallel(array))` and `import std.parallelism`.

 	
  * **code action: convert enum to bitflag enum (isBitFlagEnum assert + modify values)**  
Convert any enum `enum Foo { a, b, c }` to `enum Foo { a = 1 << 0, b = 1 << 1, c = 1 << 2 }`, add `static assert(isBitFlagEnum!Foo);` and `import std.typecons` if neccessary.


  * **search for unknown symbol or import online**  
Use [dpldocs.info](http://dpldocs.info/). Example: `import mongoschema;` -> Error importing -> Search online code fix -> Add dub package mongoschemad, etc. For undefined symbols the same: check local projects for symbols, otherwise see if online providers know anything.

 	
  * **show local references from DCD & local rename**  
Highlight a symbol in the document when hovering on a member and provide the ability to rename it (not cross-file, because DCD doesn’t support that).




## Thank you


Thanks again to all of the donors who pushed us over the goal line for this. Though we realize not everyone will be entirely pleased with the milestone list, and that it would have been better if we had finalized everything before asking for donations rather than coming at it haphazardly like we did, we hope that most of you find more here to like than not.

It will be a little while yet before we set up the new system and announce the next goal, as we’ve got other initiatives taking priority at the moment. So please be patient. We haven’t forgotten about the State of D Survey, nor are we going to let the results sit and bitrot. This train may be slow sometimes, but it’s rolling on.
