---
author: AndreasZwinkau
comments: false
date: 2016-06-16 15:02:40+00:00
layout: post
link: https://dlang.org/blog/2016/06/16/find-was-too-damn-slow-so-we-fixed-it/
slug: find-was-too-damn-slow-so-we-fixed-it
title: Find Was Too Damn Slow, So We Fixed It
wordpress_id: 31
categories:
- Guest Posts
- Phobos
permalink: /find-was-too-damn-slow-so-we-fixed-it/
redirect_from: /2016/06/16/find-was-too-damn-slow-so-we-fixed-it/
---

_This is a guest post from Andreas Zwinkau, a problem solving thinker, working as a doctoral researcher at the IPD Snelting within the InvasIC project on compiler and language perfection. He manages and teaches students at the KIT. With a beautiful wife and two jolly kids, he lives in Karlsruhe, Germany._



* * *



_Please throw this hat into the ring as well_, Andrei wrote when he [submitted the winning algorithm](http://forum.dlang.org/post/nii09n$2jjq$1@digitalmars.com). Let me tell you about this _ring_ and how we made string search in D's standard library, _Phobos_, faster. You might learn something about performance engineering and benchmarking. Maybe you'll want to contribute to Phobos yourself after you read this.

The story started for me when I decided to help out D for some recreational programming. I went to the [Get Involved wiki page](http://wiki.dlang.org/Get_involved). There was [a link to the issue tracker](https://issues.dlang.org/buglist.cgi?keywords=preapproved&resolution=---) for _preapproved_ stuff and there I found [issue 9646](https://issues.dlang.org/show_bug.cgi?id=9646), which was about [splitter ](https://dlang.org/phobos/std_algorithm_iteration.html#.splitter)being too slow in a specific case. It turned out that it was actually [find](https://dlang.org/phobos/std_algorithm_searching.html#.find), called inside splitter, which was slow. The find algorithm searches for one string (the needle) inside another (the haystack). It returns a substring of the haystack, starting with the first occurrence of the needle and ending with the end of the haystack. Find being _slow_ meant that a naively implemented find (two nested for loops) was much faster than what the standard library provided.

So, let's fix find!

Before I started coding, the crucial question was: _How will I know I am done?_ At that moment, I only had one test case from the splitter issue. I had to ensure that any solution was fast in the general case. I wanted to fix issue 9646 without making find worse for everybody else. I needed a benchmark.

As a first step, I created a [repository](https://github.com/qznc/find-is-too-slow). It initially contained a simple program which compared Phobos's find, a naive implementation from issue 9646, and a copy from Phobos I could modify and tune. My first change: insert the same naive algorithm into my Phobos copy. This was the Hello World of bugfixing and proved two things:



 	
  1. I was working on the correct code. Phobos contained multiple overloads of find and I wanted to work on the right one. For example, there was an indirection which cast `string` into a `ubyte` array to avoid [auto decoding](https://jackstouffer.com/blog/d_auto_decoding_and_you.html).

 	
  2. It was not an issue of meta programming. Phobos code is generic and uses D's capabilities for meta programming. This means the compiler is responsible for specializing the generic code to every specific case. Fixing that would have required changing the compiler, but not the standard library.


At this point I knew which specific lines I needed to speed up and I had a benchmark to quickly see the effects of my changes. It was time to get creative, try things, and find a better find.

For a start, I tried the good old classic [Boyer-Moore](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string_search_algorithm), which [the standard library provides](https://dlang.org/phobos/std_algorithm_searching.html#.BoyerMooreFinder) but wasn't using for find. I quickly discarded it, as it was orders of magnitude slower in my benchmark. Gigabytes of data are probably needed to make that worthwhile with a modern processor.

I considered simply inserting the naive algorithm. It would have fixed the problem. On the other hand, Phobos contained a slightly more advanced algorithm which tried to skip elements. It first checks the end of the needle and, on a mismatch, we can advance the needle its whole length if the end element does not appear twice in the needle. This requires one pass over the needle initially to determine the step size. That algorithm looked nice. [Someone ](http://erdani.com/)had probably put some thought into it. Maybe my benchmark was biased? To be safe, I decided to fix the performance problem without changing the algorithm (too much).

How? Did the original code have any stupid mistakes? How else could you fix a performance problem without changing the whole algorithm?

One trick could be to use D's meta programming. The code was generic, but in certain cases we could use a more efficient version. D has `static-if`, which means we could switch between the versions at compile time without any runtime overhead.

```d
static if (isRandomAccessRange!Needle) {
   // new optimized algorithm
} else {
   // old algorithm
}
```
The main difference from the old algorithm was that we could avoid creating a temporary slice to use `startsWith` on. Instead, a simple for-loop was enough. The requirement was that the needle must be a random access range.

When I had a good version, the time was ripe for [a pull request](https://github.com/dlang/phobos/pull/4362). Of course, I had to fix issues like style guide formatting before the pull request was acceptable. The D community wants high-quality code, so the [autotester ](https://auto-tester.puremagic.com/)checked my pull request by running tests on different platforms. Reviewers checked it manually.

Meanwhile in the forum, others chimed in. [Chris ](http://forum.dlang.org/post/atzzjgujnmbcquhdnucw@forum.dlang.org)and [Andrei ](http://forum.dlang.org/post/nic41v$jb4$1@digitalmars.com)proposed more algorithms. Since we had a benchmark now, it was easy to include them. Here are some numbers:

    DMD:                       LDC:
    std find:    178 ±32       std find:    156 ±33
    manual find: 140 ±28       manual find: 117 ±24
    qznc find:   102 ±4        qznc find:   114 ±14
    Chris find:  165 ±31       Chris find:  136 ±25
    Andrei find: 130 ±25       Andrei find: 112 ±26
You see the five mentioned algorithms. The first number is the mean slowdown compared to the fastest one on each single run. The annotated ± number is the [mean absolute deviation](https://en.wikipedia.org/wiki/Average_absolute_deviation). I considered LDC's performance more relevant than DMD's. You see **manual**, **qznc**, and **Andrei find** report nearly the same slowdown (117, 114, 112), and the deviation was much larger than the differences. This meant they all had roughly the same speed. Which algorithm would you choose?

We certainly needed to pick one of the three top algorithms and we had to base the decision on this benchmark. Since the numbers were not clear, we needed to improve the benchmark. When we ran it on different computers (usually an Intel i5 or i7) the numbers changed a lot.

So far, the benchmark had been generating a random scenario and then measuring each algorithm against it. The fastest algorithm got a speed of 100 and the others got higher numbers which measured their slowdown. Now we could generate a lot of different scenarios and measure the mean across them for each algorithm. This design placed a big responsibility on the scenario generator. For example, it chose the length of the haystack and the needle from a uniform distribution within a certain range. Was the uniform distribution realistic? Were the boundaries of the range realistic?

After [discussion in the forum](https://forum.dlang.org/post/niiqu4$it8$1@digitalmars.com), it came down to three basic use cases:



 	
  1. Searching for a few words within english text. The benchmark has a copy of 'Alice in Wonderland' and the task is to search for a part of the last sentence.

 	
  2. Searching for a short needle in a short haystack. This corresponds to something like finding line breaks as in the initial splitter use case. This favors naive algorithms which do not require any precomputation or other overhead.

 	
  3. Searching in a long haystack for a needle which it doesn't contain. This favors more clever algorithms which can skip over elements. To guarantee a mismatch, the generator inserts a special character into the needle, which we do not use to generate the haystack.

 	
  4. Just for comparison, the previous random scenario is still measured.


At this point, we had a good benchmark on which we could base a decision.

_Please throw this hat into the ring as well._

Andrei found another algorithm in his magic optimization bag. He knew the algorithm was good in some cases, but how would it fare in our benchmark? What were the numbers with this new contender?

![](http://beza1e1.tuxen.de/img/a2phobos_plot.png)

In short: Andrei's second algorithm completely dominated the ring. It has two names in the plot: **Andrei2** as he posted it and **A2Phobos** as I generalized it and integrated it into Phobos. In the plots you see those two algorithms always close to the optimal result 100.

It was interesting that the naive algorithm still won in the 'Alice' benchmark, but the new Phobos was really close. The old Phobos `std` was roughly twice as slow for short strings, which we already knew from issue 9646.

What did this new algorithm look like? It used the same nested loop design as Andrei's first one, but it computed the skip length only on demand. This meant one more conditional branch, but modern branch predictors seem to handle that easily.

Here is the final winning algorithm. The version in Phobos is only slightly more generic.

```d
T[] find(T)(T[] haystack, T[] needle) {
  if (needle.length == 0) return haystack;
  immutable lastIndex = needle.length - 1;
  auto last = needle[lastIndex];
  size_t j = lastIndex, skip = 0;
  while (j < haystack.length) {
    if (haystack[j] != last) {
      ++j;
      continue;
    }
    immutable k = j - lastIndex;
    // last elements match, check rest of needle
    for (size_t i = 0; ; ++i) {
      if (i == lastIndex)
        return haystack[k..$]; // needle found
      if (needle[i] != haystack[k + i])
        break;
    }
    if (skip == 0) { // compute skip length
      skip = 1;
      while (skip < needle.length &&
             needle[$-1-skip] != needle[$-1]) {
        ++skip;
      }
    }
    j += skip;
  }
  return haystack[$ .. $];
}
```
Now you might want to run the benchmark yourself on your specific architecture. [Get it from Github](https://github.com/qznc/find-is-too-slow) and run it with `make dmd` or `make ldc`. We are still interested in results from a wide range of architectures.

For me personally, this was my biggest contribution to D's standard library so far. I'm pleased with the community. I deserved all criticism and it was professionally expressed. Now we can celebrate a faster find and fix the next issue. If you want to help, the D community will welcome you!
