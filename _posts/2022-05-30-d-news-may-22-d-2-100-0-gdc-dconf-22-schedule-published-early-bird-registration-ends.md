---
author: DBlogAdmin
comments: false
date: 2022-05-30 13:01:29+00:00
layout: post
link: https://dlang.org/blog/2022/05/30/d-news-may-22-d-2-100-0-gdc-dconf-22-schedule-published-early-bird-registration-ends/
slug: d-news-may-22-d-2-100-0-gdc-dconf-22-schedule-published-early-bird-registration-ends
title: 'D News May ''22: D 2.100.0; GDC & LDC Releases; DConf ''22 Schedule Published
  & Early-Bird Registration Ends'
wordpress_id: 3089
categories:
- Compilers &amp; Tools
- DConf
- DMD Releases
- GDC Releases
- LDC Releases
- News
permalink: /d-news-may-22-d-2-100-0-gdc-dconf-22-schedule-published-early-bird-registration-ends/
redirect_from: /2022/05/30/d-news-may-22-d-2-100-0-gdc-dconf-22-schedule-published-early-bird-registration-ends/
---

![]({{ '/assets/images/d-news-may-22-d-2-100-0-gdc-dconf-22-schedule-published-early-bird-registration-ends/rocket-200x200.png' | relative_url }})

May was a busy month in D land. Early on, a major milestone release of GDC, the GCC-based D compiler, hit the virtual shelves. It was followed in middle of the month by the release of D 2.100.0 along with a DMD release, the reference D compiler, of the same version. That was immediately follwed by a beta release of the LLVM-based D compiler, LDC, version 1.30.0. Finally, the latter half of the month saw [the publication of the DConf '22 schedule](https://dconf.org/2022/index.html#schedule), we found a sponsor for the DConf tradition of BeerConf, and May 31st marks [the final day of DConf '22 early-bird registration](https://dconf.org/2022/index.html#register).

_A video version of this blog post is available on [the D Language Foundation YouTube channel](https://youtu.be/qTH1S6z-n-s)._



## D 2.100.0



[This latest release of DMD](https://dlang.org/changelog/2.100.0.html) comes to us courtesy of 41 contributors who brought us 22 major changes and 179 fixed Bugzilla issues. Although the community attached a bit of significance to the 2.100.0 version number, there isn't anything overly exciting in the changelog. This is largely a house-cleaning release--a number of deprecation periods that should have already ended have been terminated-- but there are a couple of interesting additions to the language.



### D1-style operator overloading



One of these is [the deprecation of D1-style operator overloads](https://dlang.org/changelog/2.100.0.html#d1_style_operators). Originally, these were designed to make their purpose clear. Want to overload the addition operator? Then implement `opAdd`. What to overload the multiplication operator? Then implement `opMul`. Walter took this approach with operator overloading because of one of the major complaints about the feature in C++: people often overload an operator to do something different from what it is expected to do. An example: overloading the `+` operator to append rather than perform addition. Walter's reasoning was that if the intent of the operator is included in the name of the function, then anyone overloading it to do something different is essentially violating its contract. Perhaps it would encourage people to stick to the intent.

No one can say for sure if Walter's approach worked like he hoped, but [a more generic design was implemented in D2](https://dlang.org/spec/operatoroverloading.html), and this is the approach all D code must use today. The D1 operators were kept around largely to ease porting D1 code to D2, with the intention that they would one day be deprecated. [It finally happened in D 2.088.0](https://dlang.org/changelog/2.088.0.html), which was released in the fall of 2019. [Following the deprecation process](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1013.md), the deprecation period should have ended with 2.098.0 (the first release after 10 non-patch releases including the deprecation).



### delete



The `delete` keyword was another D1 feature that was ultimately axed in D2. It [was deprecated in D 2.079.0](https://dlang.org/changelog/2.079.0.html), which was released in the spring of 2018. This was something that had long been planned ([see the deprecation page for the rationale](https://dlang.org/deprecate.html#delete)), and its use had been discouraged for some time.

N`delete` would both destroy an object instance (call its destructors) and release the memory allocated for it by the GC. Now, we use the `destroy` function from the `object` module which is imported by default in all D programs. This will call the destructor on an instance and optionally reset the instance to its default `.init` state. The GC will then free the memory allocated for the instance when necessary, or the programmer can do it manually via `GC.free` static member function in `core.memory`.



### @mustuse



[Paul Backus took DIP 1038](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1038.md) through the review process from beginning to end. Initially, it introduced an `@nodiscard` attribute for functions and types. During the Formal Assessment after the review rounds were completed, Walter and Átila were willing to approve it with changes. The final version renamed the attribute to `@mustUse` and restricted its application to structs and unions.

[The feature was implemented in D 2.100.0](https://dlang.org/changelog/2.100.0.html#mustUse) as `@mustuse`, and is now available to use in your D code. When a type marked with the attribute is the result of an expression, the result cannot be ignored.



### .tupleof for static arrays



Many D programmers are familiar with the `.tupleof` property of structs, which is particularly useful when interfacing with C libraries:


```d
struct Circle {
    float x, y;
    float radius;
    ubyte r, g, b, a;
}

@nogc nothrow
extern(C) void draw_circle (
    float cx, float cy, float radius,
    ubyte r, ubyte g, ubyte b, ubyte a
);

void foo() {
    Circle c = makeCircle();
    draw_circle(c.tupleof);
}
```
Now we can [do the same thing with static arrays](https://dlang.org/changelog/2.100.0.html#static_array_tupleof):


```python
void foo(int, int, int) { /* ... */ }

int[3] ia = [1, 2, 3];
foo(ia.tupleof); // same as `foo(1, 2, 3);

float[3] fa;
//fa = ia; // error
fa.tupleof = ia.tupleof;
assert(fa == [1F, 2F, 3F]);
```
## DConf '22



DConf '22 is happening in London, August 1 4. If you haven't registered yet and you're reading this on or before May 31st, [then register now to take advantage of the 15% early-bird discount](https://dconf.org/2022/index.html#register). The schedule is online and BeerConf is a go!



### DConf '22 schedule



We love to see and hear first-time speakers at DConf, whether it's their first conference talk ever or their first DConf talk. This year, we have 11 first-time DConf speakers, 12 if you include our invited keynote speaker Roberto Ierusalimschy ([the head designer of the Lua programming language](http://www.inf.puc-rio.br/en/teacher/@roberto-ierusalimschy)). This is awesome!

[The DConf '22 schedule](https://dconf.org/2022/index.html#schedule) is set up as follows:





  * three keynotes: two from the language maintainers, one from our guest speaker


  * two panels: the traditional DConf Ask Us Anything involving the language maintainers, and a panel on Programming Language Design


  * a Lightning Talks session


  * 15 presentations (11 of which are from first-time DConf speakers)



We're limiting the talks to 45 minutes this year so that we'll have more time to mingle between sessions. One of the talks on Day 3 is slated for 25-30 minutes, so we've slotted it such that we have a longer lunch that day.

The schedule (excluding the keynotes, as the details of those haven't yet been provided) has a loose theme. It's not perfect, but it'll do:



  * Day One is mostly status reports and tutorials


  * Day Two is largely intermediate to advanced and heavy on the tech


  * Day Three is about the D ecosystem



All of the talks will be livestreamed and recorded, so they'll be available on our YouTube channel at some point after the conference has ended. Still, DConf is about more than just the talks, [as Razvan Nitu and Dennis Korpel noted in an interview](https://youtu.be/nvo7wzjVDQc). It's about getting to know in person the people we encounter online in our regular D community interactions. As Razvan said and I can attest, your perspective will surely change after you can match the internet handles with living, breathing, human beings with whom you've interacted in person.

[So register](https://dconf.org/2022/index.html#register)!



### Early-bird registration ends



[May 31st is the last day of early-bird registration](https://dconf.org/2022/index.html#register). With the 15% discount and 20% VAT, the total is $423.30 USD. We also show the GBP equivalent on the site, based on the HMRC exchange rate for the current month, and accept payments in GBP through PayPal. On June 1st, the general registration rate of $498.00 USD (including 20% VAT) kicks in.

If you are a student, there's a flat rate of $120.00 USD (including 20% VAT). Email [social@dlang.org](mailto:social@dlang.org) to take advantge of it.

We also offer a flat rate of $240.00 USD (including 20% VAT) for major open source contributors. The keyword here is _major_. It's not something for which we can set specific criteria, and we don't really want to provide examples that may discourage inquiries. If you would like to see if you qualify for this discount, please email [social@dlang.org](mailto:social@dlang.org), and we'll let you know.

Finally, we also offer a hardship rate. If you would like to attend DConf but can't afford the registration, just email [social@dlang.org](mailto:social@dlang.org) and we'll see about helping you out. We can't help you with transportation, just the registration.



### BeerConf



BeerConf is a DConf tradition going back to the very beginning, though we didn't call it that back then. Every year, we would designate an "official" hotel somewhere in the vicinity of the venue. This would be our gathering spot in the evenings, usually in the hotel lobby or bar. Typically, would people break off into groups for dinner, then several of them would wander over to the gathering spot to hang out and chat, usually over beers. At DConf 2017, Ethan Watson branded this gathering BeerConf and the name has stuck.

At DConf 2019 in London, we couldn't find a suitable hotel to select as the site of BeerConf. Instead, we hired out the upper floor of a pub close to the venue, thanks to the sponsorship of Mercedes Benz Research and Development North America. For DConf '22, we're back in the same general area, and so we again have to hire out a pub.

The 2019 pub was a bit crowded for us, and is a bit too far of a walk from our '22 venue, so we've got our eyes on another pub within walking distance of the venue and near [some of the budget hotels listed at dconf.org](https://dconf.org/2022/index.html#venue). What we've been missing is funding.

That has changed, [thanks to Funkwerk](https://funkwerk.com/en/?noredirect=en-US)! With their sponsorship, we're able to cover the minimum spend the pub asks for the each of the evenings of August 1 3. This means that DConf attendees dropping by this pub on those nights can order food and drinks (alcoholic and or otherwise) for free until the DConf tab runs out. We'll have a separate tab for each night so that we don't blow it all in one go.

Unfortunately, I can't announce the specifics about the pub just yet. [Our DConf host, Symmetry Investments](https://symmetryinvestments.com/), is handling the arrangements for us since they're in London and we aren't. Once I receive confirmation that the deal is set, I'll announce all of the details in the forums, here on the blog, and at dconf.org. So keep your ears open!

Thanks again to Funkwerk for helping us out.



## Next time



The next big news roundup will come in late August or early September, but I'll keep the blog updated with announcements before DConf as they come. If you are planning to attend DConf, then I'm looking forward to seeing you in London. And if you aren't, then change your plans!
