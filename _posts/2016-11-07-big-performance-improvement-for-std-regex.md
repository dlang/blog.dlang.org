---
author: DmitryOlshansky
comments: false
date: 2016-11-07 13:05:03+00:00
layout: post
link: https://dlang.org/blog/2016/11/07/big-performance-improvement-for-std-regex/
slug: big-performance-improvement-for-std-regex
title: Big Performance Improvement for std.regex
wordpress_id: 469
categories:
- Guest Posts
- Phobos
permalink: /big-performance-improvement-for-std-regex/
redirect_from: /2016/11/07/big-performance-improvement-for-std-regex/
---

_Dmitry Olshansky has been a frequent contributor to the D programming language. Perhaps his best known work is his overhaul of the [std.regex](http://dlang.org/phobos/std_regex.html) module, which he architected as part of [Google Summer of Code 2011](https://www.google-melange.com/archive/gsoc/2011/orgs/dprogramminglanguage/projects/dolsh.html). In this post, he describes an algorithmic optimization he implemented this past summer that resulted in a big performance win._



* * *



Optimizing [`std.regex`](http://dlang.org/phobos/std_regex.html) has been my favorite pastime, but it has gotten harder over the years. It eventually became clear that micro-optimizing the engine's state copy routine, or trying to avoid that extra write, wasn't going to cut it anymore. To move further, I needed a new _algorithmic_ improvement. This is how the so-called "Bit-NFA" came to be implemented. Developed in May of this year, it has come a long way to land in the main repository.

Before going into details, a short overview of the engine is called for. A user-specified pattern given to the engine first goes through a compilation process, where it gets transformed into a bytecode program, along with a bunch of lookup tables and auxiliary data-structures. Bytecode implies a VM, not unlike, say, the Java VM, but far simpler and more specific. In fact, there are two of them in `std.regex`, one that evaluates execution of threads in a backtracking manner and another one which evaluates all threads in lock-step, resolving any duplicates along the way.

Now, running a full blown VM, even a tiny one, on each character of input doesn't sound all that high-performance. That's why there is an extra trick, a kickstart engine (it should probably be called a "sidekick engine"), which is a dumb approximation of the full engine. It is run over the input first. When it spots something that looks like a match, the full engine is run to check it. The only requirement is that it can have no false negatives. That is, it has to detect as positive all matches of the regex pattern. This kickstart engine is the central piece of today's post.

Historically, I intended for there to be a lot of different kickstart engines, ranging from a simple 'memchr the first byte of the pattern' to a [Boyer-Moore](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string_search_algorithm) search on the prefix of a pattern. But during the gory days of GSOC 2011, a simple solution came first and out-shadowed all others: the [Shift Or](http://www-igm.univ-mlv.fr/~lecroq/string/node6.html) algorithm.

Basically, it is an NFA ([Nondeterministic Finite Automation](https://en.wikipedia.org/wiki/Nondeterministic_finite_automaton)), where each state is a bit in a word. Shifting this word advances all the states. Masking removes those that don't match the current character. Importantly, shifting also places `0` as the first bit, indicating the active state.

With these two insights, the whole process of searching for a string becomes shifting + OR-masking the bits. The last point is checking for the successful match - one of the bits is in the finish state.

Looking at this marvelous construction, it's tempting to try and overcome its limitation - the straight-forward execution of states. So let's introduce some control flow by denoting some bits as jumps. To carry out a jump, we just need to map every combination of jump bits to the mask of the resulting positions. A basic hashmap could serve us well in this regard. Then the cycle becomes:



 	
  1. Shift the word

 	
  2. Capture control flow bits

 	
  3. Lookup control flow table

 	
  4. Mask AND with control flow bits

 	
  5. Check for finish state(s) bits

 	
  6. Mask OR with match filter table


In the end, we execute the whole engine with nothing more than a hash-map lookup, a table lookup, and a bit of bitwise operations. This is the essence of what I call the Bit-NFA engine.

Of course, there are some tricky bits, such as properly mapping bytecodes to bits. Then there comes Unicode.. oh gosh. The trick to Unicode, though, is having a fast path for ASCII < 0x80 and the rest. For ASCII, we just go with a simple table. Unicode is a two-staged variation of it. Two-staging the table let's us coalesce identical pages, saving space for the whole 21 bits of the code point range.

Overall, the picture really is worth a thousand words. Here is how the new kickstart engine stacks up.
__

![](https://cloud.githubusercontent.com/assets/281770/15093873/e40eeb62-14a4-11e6-825e-39c0d7d0836e.png)

This now only leaves us to optimize the VMs further. The proven technique is JIT-ing the bytecode and is what top engines are doing. Still, I'm glad there are notable tricks to speed up regex execution in general without pulling out this heavy handed weapon.
