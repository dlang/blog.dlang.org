---
author: JackStouffer
comments: false
date: 2017-01-20 13:31:01+00:00
layout: post
link: https://dlang.org/blog/2017/01/20/testing-in-the-d-standard-library/
slug: testing-in-the-d-standard-library
title: Testing In The D Standard Library
wordpress_id: 596
categories:
- Guest Posts
- Phobos
- The Language
permalink: /testing-in-the-d-standard-library/
redirect_from: /2017/01/20/testing-in-the-d-standard-library/
---

_Jack Stouffer is a member of the Phobos team and a contributor to dlang.org. You can check out more of his writing on [his blog](https://jackstouffer.com/blog/index.html)._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)In the D standard library, colloquially named [Phobos](https://github.com/dlang/phobos), we take a multi-pronged approach to testing and code review. Currently, there are five different services any addition has to go through:



 	
  1. The whole complier chain of tests: DMD's and DRuntime's test suite, and Phobos's unit tests

 	
  2. A documentation builder

 	
  3. Coverage analysis

 	
  4. A style checker

 	
  5. And a community project builder/test runner


Using these, we're able to automatically catch the vast majority of common problems that we see popping up in PRs. And we make regressions much less likely using the full test suite and examining coverage reports.

Hopefully this will provide some insight into how a project like a standard library can use testing in order to increase stability. Also, it can spark some ideas on how to improve your own testing and review process.


## Unit Tests


In D, unit tests are an integrated part of the language rather than a library
feature:

```d
size_t sum(int[] a)
{
    size_t result;

    foreach (e; a)
    {
        result += e;
    }

    return result;
}

unittest
{
    assert(sum([1, 2, 3]) == 6);
    assert(sum([0, 50, 100]) == 150);
}

void main() {}
```
Save this as `test.d` and run **dmd -unittest -run test.d**. Before your `main` function is run, all of the `unittest` blocks will be executed. If any of the asserts fail, execution is terminated and an error is printed to `stderr`.

The effect of putting unit tests in the language has been enormous. One of the main ones we've seen is tests no longer have the "out of sight, out of mind" problem. Comprehensive tests in D projects are the rule and not the exception. Phobos dogfoods inline `unittest` blocks and uses them as its complete test suite. There are no other tests for Phobos than the inline tests, meaning for a reviewer to check their changes, they can just run **dmd -main -unittest -run std/algorithm/searching.d** (this is just for quick and dirty tests; full tests are done via **make**).

Every PR onto Phobos runs the inline unit tests, DMD's tests, and the DRuntime tests on the following platforms:



 	
  * Windows 32 and 64 bit

 	
  * MacOS 32 and 64 bit

 	
  * Linux 32 and 64 bit

 	
  * FreeBSD 32 and 64 bit


This is done by [Brad Roberts](https://github.com/braddr)'s [auto-tester](https://auto-tester.puremagic.com/pulls.ghtml?projectid=1). As a quick aside, work is currently progressing to make bring D to [iOS](https://forum.dlang.org/thread/m2io5g7uvj.fsf@comcast.net) and [Android](https://forum.dlang.org/thread/ovkhtsdzlfzqrqneolyv@forum.dlang.org).


### Idiot Proof


In order to avoid pulling untested PRs, we have three mechanisms in place. First, only PRs which have at least one Github review by someone with pull rights can be merged.

Second, we don't use the normal button for merging PRs. Instead, once a reviewer is satisfied with the code, we tell the auto-tester to merge the PR if and only if all tests have passed on all platforms.

Third, every single change to any of the official repositories has to go through the PR review process. This includes changes made by the BDFL Walter Bright and the Language Architect Andrei Alexandrescu. We have even turned off pushing directly to the master branch in Github to make sure that nothing gets around this.


### Unit Tests and Examples


Unit tests in D solve the perennial problem of out of date docs by using the unit test code itself as the example code in the documentation. This way, documentation examples are part of the test suite rather than just some comment which _will_ go out of date.

With this format, if the unit test goes out of date, then the test suite fails. When the tests are updated, the docs change automatically.

Here's an example:

```d
/**
 * Sums an array of `int`s.
 * 
 * Params:
 *      a = the array to sum
 * Returns:
 *      The sum of the array.
 */
size_t sum(int[] a)
{
    size_t result;

    foreach (e; a)
    {
        result += e;
    }

    return result;
}

///
unittest
{
    assert(sum([1, 2, 3]) == 6);
    assert(sum([0, 50, 100]) == 150);
}

// only tests with a doc string above them get included in the
// docs
unittest
{
    assert(sum([100, 100, 100]) == 300);
}

void main() {}
```
Run **dmd -D test.d** and it generates the following un-styled HTML:

![](http://dlang.org/blog/wp-content/uploads/2017/01/Screen-Shot-2017-01-12-at-12.06.27-PM-290x300.png)

Phobos uses this to great effect. The vast majority of examples in the [Phobos documentation](https://dlang.org/phobos/index.html) are from `unittest` blocks. For example, [here](https://dlang.org/phobos/std_algorithm_searching.html#.find.3) is the documentation for `std.algorithm.fin`d and [here](https://github.com/dlang/phobos/blob/48704296eff8e8f9133563bf1de2ed73ab07a32c/std/algorithm/searching.d#L1786) is the unit test that generates that example.

This is not a catch all approach. Wholesale example programs, which are very useful when introducing a complex module or function, still have to be in comments.


### Protecting Against Old Bugs


Despite our best efforts, bugs do find their way into released code. When they do, we require the person who's patching the code to add in an extra unit test underneath the buggy function in order to protect against future regressions.


## Docs


For Phobos, the documentation pages which were changed are generated on a test server for every PR. Developed by [Vladimir Panteleev](https://github.com/CyberShadow), the [DAutoTest](http://dtest.thecybershadow.net/) allows reviewers to compare the old page and the new page from one location.

For example, [this PR](https://github.com/dlang/phobos/pull/4987) changed the docs for two `struct`s and their member functions. [This page](http://dtest.thecybershadow.net/results/816c7f4ea65d76a2b8c995fe9075afa6024f751a/a17204a3a666df395e47965515121a24d234ed38/) on DAutoTest shows a summary of the changed pages with links to view the final result.


## Coverage


Perfectly measuring the effectiveness of a test suite is impossible, but we can get a good rough approximation with test coverage. For those unaware, coverage is a ratio which represents the number of lines of code that were executed during a test suite vs. lines that weren't executed.

DMD has built-in coverage analysis to work in tandem with the built-in unit tests. Instead of **dmd -unittest -run main.d**, do **dmd -unittest -cov -run main.d** and a file will be generated showing a report of how many times each line of code was executed with a final coverage ratio at the end.

We generate this report for each PR. Also, we use codecov in order to get details on how well the new code is covered, as well as how coverage for the whole project has changed. If coverage for the patch is lower than 80%, then codecov marks the PR as failed.

At the time of writing, of the 77,601 lines of code (not counting docs or whitespace) in Phobos, 68,549 were covered during testing and 9,052 lines were not. This gives Phobos a test coverage of [88.3%](https://codecov.io/gh/dlang/phobos), which is increasing all of the time. This is all achieved with the built in `unittest` blocks.


## Project Tester


Because test coverage doesn't necessarily "cover" all real world use cases and combinations of features, D uses a [Jenkins](https://jenkins.io/index.html) server to download, build, and run the tests for [a select number of popular D projects](https://ci.dawg.eu/job/projects/) with the master branches of Phobos, DRuntime, and DMD. If any of the tests fail, the reviewers are notified.


## Style And Anti-Pattern Checker


Having a code style set from on high stops a lot of pointless Internet flame wars (tabs vs spaces anyone?) dead in their tracks. D has had such a [style guide](https://dlang.org/dstyle.html) for a couple of years now, but its enforcement in official code repos was spotty at best, and was mostly limited to brace style.

Now, we use CircleCI in order to run a series of bash scripts and the fantastically helpful [dscanner](https://github.com/Hackerpilot/Dscanner) which automatically checks for [all sorts of things](https://github.com/Hackerpilot/Dscanner#implemented-checks) you shouldn't be doing in your code. For example, CircleCI will give an error if it finds:



 	
  * bad brace style

 	
  * trailing whitespace

 	
  * using whole module imports inside of functions

 	
  * redundant parenthesis


And so on.

The automation of the style checker and coverage reports was done by [Sebastian Wilzbach](https://github.com/wilzbach). dscanner was written by [Brian Schott](https://github.com/Hackerpilot).


## Closing Thoughts


We're still working to improve somethings. Currently, Sebastian is writing a script to [automatically check](https://github.com/dlang/phobos/pull/4998) the documentation of every function for at least one example. Plus, the D Style Guide can be expanded to end arguments over the formatting of template constraints and other contested topics.

Practically speaking, other than getting the coverage of Phobos up to >= 95%, there's not too much more to do. Phobos is one of the most throughly tested projects I've ever worked on, and it shows. Just recently, Phobos hit under [1000 open bugs](https://issues.dlang.org/buglist.cgi?component=phobos&limit=0&list_id=213089&order=bug_id&product=D&query_format=advanced&resolution=---), and that's including enhancement requests.
