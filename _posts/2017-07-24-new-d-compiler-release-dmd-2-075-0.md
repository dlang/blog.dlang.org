---
author: DBlogAdmin
comments: false
date: 2017-07-24 13:11:13+00:00
layout: post
link: https://dlang.org/blog/2017/07/24/new-d-compiler-release-dmd-2-075-0/
slug: new-d-compiler-release-dmd-2-075-0
title: 'New D Compiler Release: DMD 2.075.0'
wordpress_id: 983
categories:
- Code
- Compilers &amp; Tools
- Core Team
- DMD Releases
- News
permalink: /new-d-compiler-release-dmd-2-075-0/
redirect_from: /2017/07/24/new-d-compiler-release-dmd-2-075-0/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)DMD 2.075.0 was released a few days back. As with every release, the changelog is available so you can [browse the list of fixed bugs and new features](https://dlang.org/changelog/2.075.0.html). 2.075.0 can be fetched from the [dlang.org download page](https://dlang.org/download.html), which always makes available the latest DMD release alongside a nightly build.


### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#notable-changes)Notable Changes


Every DMD release brings with it a number of bug fixes, changes, and enhancements. Here are some of the more noteworthy changes in this release.


#### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#two-array-properties-removed)Two array properties removed


Anyone who does a lot of work with D's ranges will likely have encountered this little annoyance that arises from the built-in `.sort` property of arrays.

```d
void main()
{
    import std.algorithm : remove, sort;
    import std.array : array;
    int[] nums = [5, 3, 1, 2, 4];
    nums = nums.sort.remove(2).array;
}
```
The `.sort` property has been deprecated for ages, so the above would result in the following error:

    sorted.d(6): Deprecation: use std.algorithm.sort instead of .sort property
The workaround would be to add an empty set of parentheses to the sort call. With DMD 2.075.0, this is no longer necessary and the above will compile. Both the `.sort` and `.reverse` array properties [have finally been removed](http://dlang.org/changelog/2.075.0.html#removeArrayProps) from the language.

For the uninitiated, D has two features that have proven convenient in the functional pipeline programming style typically used with ranges. One is that parentheses on a function call are optional when there are no parameters. The other is [Universal Function Call Syntax (UFCS)](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs), which allows a function call to be made using the dot notation on the first argument, so that a function `int add(int a, int b)` can be called as: `10.add(5)`.

Each of D's built-in types comes with a set of built-in properties. Given that the built-in properties are not functions, no parentheses are used to access them. The `.sort` array property has been around since the early days of D1. At the time, it was rather useful and convenient for anyone who was happy with the default implementation. When D2 came along with the range paradigm, the standard library was given a set of functions that can treat arrays as ranges, opening them up to use with the many range based functions in the `std.algorithm` package and elsewhere.

With optional parentheses, UFCS, and a range-based function in `std.algorithm` called `sort`, conflict was inevitable. Now range-based programmers can put that behind them and take one more pair of parentheses out of their pipelines.


#### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#the-breaking-up-of-stddatetime)The breaking up of std.datetime


The `std.datetime` module has had a reputation as the largest module in D's standard library. Some developers have been known to use it a stress test for their tooling. It was added to the library long before D got the special [package module](http://dlang.org/spec/module.html#package-module) feature, which allows multiple modules in a package to be imported as a single module.

Once package modules were added, Jonathan M. Davis, the original `std.datetime` developer, found it challenging to split the monolith into multiple modules. Then, at [DConf 2017](http://dconf.org/2017/index.html), he could be seen toiling away on his laptop in the conference hall and the hotel lobby. On the final day of the conference, the day of the DConf Hackathon, he announced that `std.datetime` was now a package. DMD 2.075.0 is [the first release](http://dlang.org/changelog/2.075.0.html#split-std-datetime) where the new module structure is available.

Any existing code using the old module should still compile. However, any static libraries or object files lying around with the old symbols stuffed inside may need to be recompiled.


#### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#colorized-compiler-messages)Colorized compiler messages


This one is missing from the changelog. DMD now has the ability to output colorized messages. The implementation required going through the existing error messages and properly annotating them where appropriate, so there may well be some messages for which the colors are missing. Also, given that this is a brand new feature and people can be picky about their terminal colors, more work will likely be done on this in the future. Perhaps that might include support for customization.




#### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#compiler-ddoc-documentation-online)Compiler Ddoc documentation online


DMD, though originally written in C++, was converted to D some time ago. Now that more D programmers are able to contribute to the compiler, work has gone into documenting its source using D's built-in [Ddoc syntax](https://tour.dlang.org/tour/en/gems/documentation). The result is now online, accessible from the sidebar of the existing library reference. A good starting point is the [ddmd.mars module](https://dlang.org/phobos/ddmd_mars.html).


### [](https://gist.github.com/mdparker/ee8372c0275c79037252ed087d70043c#and-more)And more...


The above is a small part of the bigger picture. The [bugfix list](http://dlang.org/changelog/2.075.0.html#bugfix-list) shows 89 bugs, regressions, and enhancements across the compiler, runtime, standard library, and web site. See [the full changelog](http://dlang.org/changelog/2.075.0.html) for the details.

Thanks to everyone who contributed to this release, whether it was by reporting issues, submitting or reviewing pull requests, testing out the beta, or carrying out any of the numerous small tasks that help a new release see the light of day.
