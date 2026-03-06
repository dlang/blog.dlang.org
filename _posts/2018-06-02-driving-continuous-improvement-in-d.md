---
author: JackStouffer
comments: false
date: 2018-06-02 07:14:20+00:00
excerpt: Keeping a level of quality in software projects is a neverending battle.
  Bad practices and shortsighted design decisions make their way into code over time,
  whether from poor oversight, rushing things through, or simple code rot. There are
  three things we do in Phobos, other than tests, to fight this entropy.
layout: post
link: https://dlang.org/blog/2018/06/02/driving-continuous-improvement-in-d/
slug: driving-continuous-improvement-in-d
title: Driving Continuous Improvement in D
wordpress_id: 1568
categories:
- Code
- Compilers &amp; Tools
permalink: /driving-continuous-improvement-in-d/
redirect_from: /2018/06/02/driving-continuous-improvement-in-d/
---

_Jack Stouffer is a member of the Phobos team and a contributor to dlang.org. You can check out more of his writing [on his blog](https://jackstouffer.com/blog/index.html)._


* * *


![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)In my [previous article](https://dlang.org/blog/2017/01/20/testing-in-the-d-standard-library/), I went over the techniques we use in the D Standard Library (a.k.a Phobos) to develop a wide variety of testing mechanisms. I also briefly mentioned our style checker, [dscanner](https://github.com/dlang-community/D-Scanner). In this article, I’ll detail how we use dscanner to prevent style, documentation, and best practices regressions in the code, none of which can be covered by standard unit tests.

Keeping a level of quality in software projects is a neverending battle. Bad practices and shortsighted design decisions make their way into code over time, whether from poor oversight, rushing things through, or simple code rot. There are three things we do in Phobos, other than tests, to fight this entropy.

First, Pull Requests are required to be about one thing and one thing only. What counts as “one thing” is subjective, but, for example, a PR to fix a bug in a specific piece of code mustn’t also edit the documentation of that function. This allows Phobos reviewers to merge the documentation change quickly if there’s an issue in the bug fix preventing it from being merged (or vice versa). At the same time, it keeps the reviewers focused on one set of changes.

Second, PRs need to be small; ideally less than 100 lines of code changed. If that’s not possible, they need to be broken into multiple commits of smaller changes. This really helps reviewers keep D best practices in mind, while also fully understanding and internalizing the new code.

Third, we continuously improve the existing code with dscanner, which, among other things, is D’s official linting tool.


## What dscanner Can Do


As an example of the checks that dscanner can provide, let’s take a look at some code that contains a very hard-to-spot bug. The following code creates a type that mimics an `int`, but allows a `null` state:

```d
struct NullableInt
{
    private int value;
    bool isNull;

    int get()
    {
        assert(!isNull, "can't get a null");
        return value;
    }

    void nullify() { isNull = true; }

    bool opEquals(T rhs) // D's equality overloading function
    {
        if (isNull && rhs.isNull)
            return true;
        if (isNull != rhs.isNull)
            return false;
        return value == rhs.value;
    }
}
```
The bug is not in the code that’s there, it’s in the code that’s not there. All `struct`s in D have default versions of the standard operator overloading functions if they aren’t defined by the user. One of those functions provides a hash to represent the value to use in D’s built-in associative arrays. The default version uses all the type fields to make the hash, which is a problem for `NullableInt`, as we’ve decided that all instances of the type that are `null` are equal. Here’s an illustration of the bug:

```d
void main()
{
    auto a = NullableInt(10, false);
    auto b = NullableInt(10, false);
    auto c = NullableInt(10, true);
    auto d = NullableInt(0, true);

    assert(a == b);
    assert(b != c);
    assert(c == d);

    import core.internal.hash : hashOf; // D's internal hashing function
    assert(c.hashOf != d.hashOf);
}
```
dscanner will emit a message every time it finds a type that defines a custom `opEquals`, but doesn’t define a custom `toHash` as well, alerting us to this bug.


## Continuous Improvement and “Ratcheting” Quality


dscanner ties into continuous improvement, the philosophy that improvements to processes or designs should be made in a periodic, timely manner, rather than as one-time “breakthroughs”. Such large breakthrough changes tend to be pushed back infinitely; in a phrase, it’s letting the perfect be the enemy of the good. This easily fits with the continuous delivery or rapid release philosophies, and can vastly reduce bugs in software given enough time.

By using the warnings from dscanner, we can ratchet Phobos’s quality over time. If you’ve never used a socket wrench, a ratchet is a mechanism that allows the user to freely move the wrench back and forth, but allows the bolt to spin only when the wrench moves clockwise. Similarly, we can move the quality of Phobos forward, while not letting it ever slip backwards, with dscanner. It works as follows:



 	
  * Run dscanner with every check but one turned off on all files.

 	
  * Populate a black list for that check in dscanner’s configuration file containing each of the files that issued a warning.

 	
  * Repeat until the current code passes with all checks turned on with the black-list enabled.


Once we have our list-of-blacklists, new PRs will trigger warnings for issues detected by dscanner for files that weren’t included in the blacklists, thereby keeping the status quo of quality.

Next, we ratchet quality by making a PR that fixes one issue in one file, and then removing that blacklist entry from the configuration. This way, that file will be checked in the future for every new PR. Over time, quality issues will be removed from Phobos, and they won’t crop up again. For example, from the release of 2.076 to now, Phobos has gone from 624 public symbols without examples, to 328.

Currently, we’re using this ratchet technique with dscanner to

 	
  * remove unused variables.

 	
  * add `const` or `immutable` to variables that aren’t modified after their construction.

 	
  * make sure every public symbol is documented.

 	
  * make sure every symbol in Phobos that has documentation, has a code example in that documentation.

 	
  * force every assert to have an error message to print in case it fails.

 	
  * make sure only `Exceptions`, and not `Error`s, are caught in `try/catch` blocks.


among other things.

This process also has the added benefit of dogfooding dscanner, which helps us find bugs and know which features would be helpful to add from a user perspective. If you have a project that isn’t using a linting tool as part of your test suite, it’s only a matter of time before code rot creeps in.
