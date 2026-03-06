---
author: AliCehreli
comments: false
date: 2016-06-29 13:14:45+00:00
layout: post
link: https://dlang.org/blog/2016/06/29/programming-in-d-a-happy-accident/
slug: programming-in-d-a-happy-accident
title: 'Programming in D: A Happy Accident'
wordpress_id: 71
categories:
- Books
- Guest Posts
permalink: /programming-in-d-a-happy-accident/
redirect_from: /2016/06/29/programming-in-d-a-happy-accident/
---

_This is a guest post from Ali Çehreli, who not only uses D as a hobby, but also gets to use it as an employee of _[Weka.io](http://www.weka.io/)_. He is the author of _Programming in D_ and is frequently found in the _[D Learn forum](http://forum.dlang.org/group/learn)_ with ready answers to questions on using the language. He also is an officer of the [D Foundation](http://dlang.org/foundation.html)._



* * *



![progind](http://dlang.org/blog/wp-content/uploads/2016/06/progind.jpg)I consider the book [Programming in D](http://ddili.org/ders/d.en/index.html) a happy accident, because initially it was not intended to be a book.

I used to be a frequent contributor to Turkish C and C++ forums. Although there were many smart and motivated members on those forums, most of them were not well-versed enough to follow programming resources in English. If they were patient enough to wait about ten years and if a publisher decided to have them translated, then they might get their hands on Turkish versions of their favorite books.

In 2009, around the time when my interest in C++ had started to diminish, I read with great excitement Andrei Alexandrescu's _The Case for D_ article in ACCU's C Vu magazine (also [available at Dr. Dobb's](http://www.drdobbs.com/parallel/the-case-for-d/217801225)). To a person coming from a C++ background, D was a fresh breath of air, removing some of C++'s warts and bringing many new features, some unique, some borrowed from other languages.

I was instantly hooked. I immediately created the Turkish D site [ddili.org](http://ddili.org/), translated Andrei's article to Turkish, and published it there. One of the reasons for my excitement was the potential that D could be one software technology that Turkish programmers would not be left in the dark about. Since D was still being designed and implemented, there was time to write fresh Turkish documentation for it. I translated other D articles and started writing an HTML tutorial that would later become the book.

I knew very well that attempting to teach a topic is one of the best ways of learning that topic. I knew that I would be learning D myself. Little did I know then that this project would make me a better software engineer in general as well.

Teaching programming is a notoriously difficult task. According to some academic papers I found when I started the tutorial, one of the difficulties comes from the fact that different people model new concepts in their minds in different ways, rendering particular teaching methods inefficient at least for some students. Encouraged by the lack of one correct way of teaching programming, I picked one that was the easiest for me: introducing concepts in linear fashion with as few forward references as possible, starting with the most basic concepts like the `=` character confusingly meaning something different than _is equal to_.

Starting from the basics made it necessary for me to introduce lower-level concepts before higher-level concepts. For example, although the `foreach` statement is much more commonly used in practice, `while`, `for`, and `foreach` statements are introduced in the book in that order. I think that choice created a better foundation for the reader.

It took me two years to finish writing a flow of chapters from the assignment operator all the way to the garbage collector. It was very challenging and very rewarding to find a natural flow of presentation not only throughout the book but also within each chapter. The method I used for each chapter was to think about the presentation of the topic along with non-trivial examples beforehand, without touching the computer. I then wrote the chapter fairly quickly without much attention to detail, put it aside for a couple of days, then came back to review it from the point of view of a reader. I repeated that process perhaps five to ten times for each chapter until I thought it was fairly acceptable. Likely as a result of that process, a common feedback I receive is about how _to-the-point_ my writing style has been.

Based on feedback from the Turkish community and encouragement from Andrei Alexandrescu, I started translating the book to English in early 2011. The translation continued along with new chapter additions, many corrections, and some chapter rewrites.

I made a PDF version available in January 2012 and the translation was finally completed in July 2014. Not only had I achieved my initial goal of providing fresh Turkish documentation for D, this book might have been the first software resource that was translated in the other direction.

I readily agreed with the suggestion that the book should be available in paper form as well. That decision brought many different challenges related to self-publishing like layout, cover design, pricing, the printing company, etc. The first print publication was in August 2015. Surprisingly, producing an ebook version turned out to be even more challenging. In addition to different kinds of layout issues, all ebook formats require special attention.

I am awestruck that my humble idea of a humble tutorial turned into a well known resource in the D ecosystem. It makes me very happy that people actually find the book useful. I am also happy that, periodically, people express interest in translating it to other languages. As of this writing, in addition to the completed Turkish and English versions, there are ongoing translations by volunteers to French and Chinese (German, Korean, Portuguese, and Russian translations were started but not continued).

As for future directions, I would like to add more chapters; definitely one on allocators once they're added to the standard library (they currently live in the [std.experimental.allocator](http://dlang.org/phobos/std_experimental_allocator.html) package).

One thing that bothers me about the book is that most code samples don't take full advantage of D's universal function call syntax (UFCS), mainly because that feature was added to the language only after most of the book was already written. I would like to move the UFCS chapter to an earlier point in the book so that more code samples can be in the idiomatic D style.

The book will always be freely available online, allowing me to make frequent updates and corrections. Fortunately, my Inglish leaves a lot to improve on, so there will always be grammar and typo corrections as well.
