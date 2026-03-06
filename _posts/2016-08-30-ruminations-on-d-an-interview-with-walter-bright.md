---
author: Joakim
comments: false
date: 2016-08-30 11:44:11+00:00
layout: post
link: https://dlang.org/blog/2016/08/30/ruminations-on-d-an-interview-with-walter-bright/
slug: ruminations-on-d-an-interview-with-walter-bright
title: 'Ruminations on D: An Interview with Walter Bright'
wordpress_id: 198
categories:
- Community
- Core Team
- Interviews
permalink: /ruminations-on-d-an-interview-with-walter-bright/
redirect_from: /2016/08/30/ruminations-on-d-an-interview-with-walter-bright/
---

_Joakim is the resident interviewer for the D Blog. He has also [interviewed](http://arsdnet.net/this-week-in-d/mar-15.html) [members](http://arsdnet.net/this-week-in-d/jun-28.html) [of the](http://arsdnet.net/this-week-in-d/jul-12.html) [D community](http://arsdnet.net/this-week-in-d/sep-06.html) for [This Week in D](http://arsdnet.net/this-week-in-d/) and is responsible for [the Android port of LDC](https://github.com/joakim-noah/android/releases)._



* * *



![d6](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)Walter Bright is [the creator and first implementer of the D programming language](http://walterbright.com). He was [an early developer of C++ compilers starting from the mid-'80s, including the first C++ compiler to translate source code directly to object code without using C as an intermediate](https://en.wikipedia.org/wiki/Walter_Bright), and has written compilers for ABEL, C, Java, and [Javascript](https://github.com/DigitalMars/DMDScript). He believes he is the only person to have written a full C++98 compiler by himself. Empire, one of the first computer strategy games, [was written by Walter at Caltech](http://www.classicempire.com/history.html). Before getting into computer programming full-time, Walter worked for Boeing from 1979-1982 on the 757's flight controls, particularly the stabilizer trim gearbox and system and the elevators.

**Joakim:** D appears to be picking up speed. The [fourth straight DConf took place earlier this year](https://www.youtube.com/playlist?list=PL3jwVPmk_PRyTWWtTAZyvmjDF4pm6EX6z), [Wired wrote a nice article a couple years ago](http://www.wired.com/2014/07/d-programming-language/), [Gartner ranked it in the top 20 languages soon afterwards](http://blogs.gartner.com/mark_driver/2014/10/02/gartner-programming-language-index-for-2014/), and [downloads of the reference compiler, DMD, average 1200 per day in recent years](http://erdani.com/d/downloads.daily.png). Talk about the current popularity and status of the language and what you're doing to take it to the next level.

**Walter:** I don't worry too much about that. I spend my efforts making D the best language possible, and let the metrics take care of themselves. It's like being a CEO; he shouldn't be sweating the stock price, he should be working on making money for the company, then the stock price will take care of itself.

There's always a stack of things to do, more or less with the most important one on the top, and I pop the top off and work on it, mostly the one with the maximum benefit/cost ratio. The ordering in the stack changes all the time. If an item has others strongly interested in working on it, I defer to them.

I get the jobs that nobody else wants to do. :-) Regression fixes for complex problems is a big one. It's much more fun designing new stuff than maintenance on the existing stuff, dealing with technical debt, etc.

**Joakim:** In the last year, `@nogc` and interfacing with C++ have been at the top of your stack. Why? You always used to say that interfacing more with C++ would be too much work.

**Walter:** It became clear that the garbage collector wasn't needed to be embedded in most things, that memory allocation could be decided separately from the algorithm. Letting the user decide seemed like a great way forward.

As for C++, I figured out a way to support it that avoided the problems I thought were impractical to deal with. [I did a talk about it](https://www.youtube.com/watch?v=IkwaV6k6BmM)- [ PDF slides here](http://walterbright.com/cppint.pdf)- and [a Rust user wrote up a partial transcript](https://internals.rust-lang.org/t/interfacing-d-to-legacy-c-code-a-summary-of-a-competing-languages-capabilities/1406).

Since that talk, I've come up with a solution for exceptions. C++ can throw/catch exceptions of any type. But few people do anything other than throw/catch a reference to a class, like `std::exception`. All D needed to do was allow throw/catch of a class marked with the `extern (C++)` attribute. Throw/catch of other types remains unsupported, and most likely will remain unsupported.

In order to make that work, the custom exception handling code in the code generation and the library had to change to work with the DWARF exception handling mechanism. That turned out to be a fair amount of work, as it is rather under-documented. But it's pretty simple for the user.

**Joakim:** What do you plan on working on in D for the rest of the year? Work on `@nogc` and C++ support seems to be ongoing and you've tried to increase the use of [ranges](https://www.packtpub.com/books/content/understanding-ranges) in the standard library for some time now. Anything else? What is the status of those efforts?

**Walter:** Those are still high-priority and ongoing efforts. But also we're flexible; if a major opportunity comes up that needs us to push in a different direction, we'll adapt. The pervasive use of ranges is advancing rapidly, I've been very pleased with the results.

The current focus is on improving memory safety. It's become more and more important to have verifiable memory safety in a programming language, as the expenses involved when unsafe code is exploited in order to install malware become greater and greater. D has always supported memory safety, but recently we've embarked on a much more comprehensive review of memory safety in the core language and are making changes to close the gaps.

**Joakim:** How much time do you spend on D and what is your daily routine?

**Walter:** I work full time on D. I probably spend half my time working on the language, and the other half helping others with it, discussing things, doing interviews (!), writing articles, doing conferences, etc.

**Joakim:** [You were 42 when you started working on D](http://www.digitalmars.com/articles/b89.html) and I guess it is the first language you designed? Talk about why you started working on it so late--you were probably older then than most of the D contributors now--and what insights your experience gave you that your younger self or other young contributors may not have.

**Walter:** [ABEL is the first language I designed](https://en.wikipedia.org/wiki/Advanced_Boolean_Expression_Language), and was a solid success for Data I/O. It's obsolete now, because the electronic devices it was aimed at are now obsolete, but it had a great run for 10 years or so.

Having been writing compilers my whole career, doing tech support for them, and following the various changes in the languages inevitably gave me much insight on what worked and what didn't. Probably the biggest thing is that simpler is better. But making something simple is actually quite difficult. Anybody can (and does) design a complex solution, but few can see through the complexity to find the underlying simplicity.

Many successful languages were designed by older engineers.

**Joakim:** Please give some examples of such simplicity and how you were able to find it.

**Walter:** We nailed it with arrays (Jan Knepper's idea), the basic template design, compile-time function execution (CTFE), and `static if`. I have no idea what the thought process is in any repeatable manner. If anything, it's simply a dogged sense that there's got to be a better way. It took me years to suddenly realize that a template function is nothing more than a function with two sets of parameters --compile time and run time--and then everything just fell into place.

I was more or less blinded by the complexity of templates such that I had simply missed what they fundamentally were. There was also a bit of the "gee, templates are hard" that predisposes one to believe they actually are hard, and then confirmation bias sets in.

I once attended a Scott Meyers presentation on type lists. He took an hour to explain it, with slide after slide of complexity. I had the thought that if it was an array of ints, his presentation would be 2 minutes long. I realized that an array of types should be equally straightforward.

With CTFE, we just went straight in the front door from asking the question "why can't we execute this function at compile time, just like constant folding?" I did the initial one just by extending the constant folding logic. Don Clugston took it much further, but at its heart it's still a modified dwarf. Stefan Koch is currently working on making a real interpreter out of it.

**Joakim:** Why do you think there hasn't been a killer app for D yet? For example, Ruby was kind of an obscure language for a dozen years till Rails propelled it into the spotlight.

**Walter:** I suspect the age of the killer app is behind us. There is so much software existing and being written, for every imaginable purpose, that it's hard to believe there will even be another killer app of any sort. Of course, predictions are notoriously unreliable.

**Joakim:** D has a unique approach in the compiled languages market, being mostly community-developed without a major corporate sponsor. C++ had Bell Labs, Rust has Mozilla, Go has Google; all have full-time paid devs on the language, even if they also take open-source contributions. Do you think this is a problem and is D being left behind?

**Walter:** We recently started [a D foundation](https://dlang.org/foundation.html) which will make it a lot easier for corporations to sponsor D.

**Joakim:** Do you make money off D? I know you've contracted with Facebook to write a C++ linter, [Flint](https://github.com/facebookarchive/flint), and preprocessor, [Warp](https://github.com/facebookarchive/warp), and that you work with Sociomantic and other companies using D.

**Walter:** I do some paid consulting work for D, but am careful not to take on so much that it interferes with working on D itself.

**Joakim:** Do you write much code in D outside of the standard library? If so, talk about recent stuff you've written and how the experience has been, plus a bit about using some of the new features.

**Walter:** D consumes so much of my efforts, there's not much time to write other D apps other than smallish utilities. Currently, of course, the D compiler front end itself is now in D. But it's been translated from C++, so isn't idiomatic D.

**Joakim:** OK, so I guess Warp is the largest program you've written in D lately, about 10 klocs. Can you talk about the experience of actually coding that in D, as opposed to C/C++? What stood out for you?

**Walter:** What stood out is the speed with which it went together, and the remarkably small number of bugs that surfaced in it after it was released. I credit the extensive use of unit tests for the latter.

Having written another preprocessor before certainly helped, and it would be hard to tease that out as a separate effect. But I still believe that unit testing made the difference, because the way the preprocessor worked was very different from the one I'd done before.

What stood out with D was [the ease of changing the data structures to try different ways, compared with doing this in other languages](https://code.facebook.com/posts/476987592402291/under-the-hood-warp-a-fast-c-and-c-preprocessor/).

**Joakim:** When you think back to your vision for D as you were starting out in 1999, does it at all resemble that today? Anything big missing that you wanted back then?

**Walter:** D is far more advanced than what I thought of 15 years ago. Programming language ideas have certainly advanced since then, and D along with it.

D originally wasn't going to have templates at all, based on my earlier experience with them. But finding a simple way to do them changed everything - even to the point where well over half of a modern D program is templates!

The idea of ranges slowly evolved over time, we're still learning how to do it right.

`bit` as a basic type was unworkable. `complex` as a basic type turned out to be pointless (it works better as a library type). Auto-decoding of UTF-8 to code points turned out not nearly as useful as expected.

_[Transitive const](https://dlang.org/const-faq.html)_ was a leap of faith, and is consistently overlooked by other languages looking to adopt D features. I have a lot of faith in transitive const, and it is already paying off in making it possible to have pure functions, a key feature for modern programming.
