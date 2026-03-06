---
author: WalterBright
comments: false
date: 2019-07-15 14:51:39+00:00
layout: post
link: https://dlang.org/blog/2019/07/15/ownership-and-borrowing-in-d/
slug: ownership-and-borrowing-in-d
title: Ownership and Borrowing in D
wordpress_id: 2136
categories:
- Code
- Core Team
- The Language
permalink: /ownership-and-borrowing-in-d/
redirect_from: /2019/07/15/ownership-and-borrowing-in-d/
---

![Digital Mars logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)Nearly all non-trivial programs allocate and manage memory. Getting it right is becoming increasingly important, as programs get ever more complex and mistakes get ever more costly. The usual problems are:



 	
  1. memory leaks (failure to free memory when no longer in use)

 	
  2. double frees (freeing memory more than once)

 	
  3. use-after-free (continuing to refer to memory already freed)


The challenge is in keeping track of which pointers are responsible for freeing the memory (i.e. owning the memory), which pointers are merely referring to the memory, where they are, and which are active (in scope).

The common solutions are:

 	
  1. Garbage Collection - The GC owns the memory and periodically scans memory looking for any pointers to that memory. If none are found, the memory is released. This scheme is reliable and in common use in languages like Go and Java. It tends to use much more memory than strictly necessary, have pauses, and slow down code because of inserted write gates.

 	
  2. Reference Counting - The RC object owns the memory and keeps a count of how many pointers point to it. When that count goes to zero, the memory is released. This is also reliable and is commonly used in languages like C++ and ObjectiveC. RC is memory efficient, needing only a slot for the count. The downside of RC is the expense of maintaining the count, building an exception handler to ensure the decrement is done, and the locking for all this needed for objects shared between threads. To regain efficiency, sometimes the programmer will cheat and temporarily refer to the RC object without dealing with the count, engendering a risk that this is not done correctly.

 	
  3. Manual - Manual memory management is exemplified by C’s `malloc` and `free`. It is fast and memory efficient, but there’s no language help at all in using them correctly. It’s entirely up to the programmer’s skill and diligence in using it. I’ve been using `malloc` and `free` for 35 years, and through bitter and endless experience rarely make a mistake with them anymore. But that’s not the sort of thing a programming shop can rely on, and note I said “rarely” and not “never”.


Solutions 2 and 3 more or less rely on faith in the programmer to do it right. Faith-based systems do not scale well, and memory management issues have proven to be very difficult to audit (so difficult that some coding standards prohibit use of memory allocation).

But there is a fourth way - Ownership and Borrowing. It’s memory efficient, as performant as manual management, and mechanically auditable. It has been recently popularized by the Rust programming language. It has its downsides, too, in the form of a reputation for having to rethink how one composes algorithms and data structures.

The downsides are manageable, and the rest of this article is an outline of how the ownership/borrowing (OB) system works, and how we propose to fit it into D. I had originally thought this would be impossible, but after spending a lot of time thinking about it I’ve found a way to fit it in, much like we’ve fit functional programming into D (with transitive immutability and function purity).


## Ownership


The solution to who owns the memory object is ridiculously simple—there is only one pointer to it, so that pointer must be the owner. It is responsible for releasing the memory, after which it will cease to be valid. It follows that any pointers in the memory object are the owners of what they point to, there are no other pointers into the data structure, and the data structure therefore forms a tree.

It also follows that pointers are not copied, they are moved:

    T* f();
    void g(T*);
    T* p = f();
    T* q = p; // value of p is moved to q, not copied
    g(p);     // error, p has invalid value
Moving a pointer out of a data structure is not allowed:

```d
struct S { T* p; }
S* f();
S* s = f();
T* q = s.p; // error, can't have two pointers to s.p
```
Why not just mark `s.p` as being invalid? The trouble there is one would need to do so with a runtime mark, and this is supposed to be a compile-time solution, so attempting it is simply flagged as an error.

Having an owning pointer fall out of scope is also an error:

```d
void h() {
  T* p = f();
} // error, forgot to release p?
```
It’s necessary to move the pointer somewhere else:

```d
void g(T*);
void h() {
  T* p = f();
  g(p);  // move to g(), it's now g()'s problem
}
```
This neatly solves memory leaks and use-after-free problems. (Hint: to make it clearer, replace `f()` with `malloc()`, and `g()` with `free()`.)

This can all be tracked at compile time through a function by using [Data Flow Analysis (DFA)](https://en.wikipedia.org/wiki/Data-flow_analysis) techniques, like those used to compute [Common Subexpressions](https://en.wikipedia.org/wiki/Common_subexpression_elimination). DFA can unravel whatever rat’s nest of `goto`s happen to be there.


## Borrowing


The ownership system described above is sound, but it is a little too restrictive. Consider:

```d
struct S { void car(); void bar(); }
struct S* f();
S* s = f();
s.car();  // s is moved to car()
s.bar();  // error, s is now invalid
```
To make it work, `s.car()` would have to have some way of moving the pointer value back into `s` when `s.car()` returns.

In a way, this is how borrowing works. `s.car()` borrows a copy of `s` for the duration of the execution of `s.car()`. `s` is invalid during that execution and becomes valid again when `s.car()` returns.

In D, struct member functions take the `this` by reference, so we can accommodate borrowing through an enhancement: taking an argument by ref borrows it.

D also supports scope pointers, which are also a natural fit for borrowing:

```d
void g(scope T*);
T* f();
T* p = f();
g(p);      // g() borrows p
g(p);      // we can use p again after g() returns
```
(When functions take arguments by ref, or pointers by scope, they are not allowed to escape the ref or the pointer. This fits right in with borrow semantics.)

Borrowing in this way fulfills the promise that only one pointer to the memory object exists at any one time, so it works.

Borrowing can be enhanced further with a little insight that the ownership system is still safe if there are multiple const pointers to it, as long as there are no mutable pointers. (Const pointers can neither release their memory nor mutate it.) That means multiple const pointers can be borrowed from the owning mutable pointer, as long as the owning mutable pointer cannot be used while the const pointers are active.

For example:

```d
T* f();
void g(T*);
T* p = f();  // p becomes owner
{
  scope const T* q = p; // borrow const pointer
  scope const T* r = p; // borrow another one
  g(p); // error, p is invalid while q and r are in scope
}
g(p); // ok
```
## Principles


The above can be distilled into the notion that a memory object behaves as if it is in one of two states:



 	
  1. there exists exactly one mutable pointer to it

 	
  2. there exist one or more const pointers to it


The careful reader will notice something peculiar in what I wrote: “as if”. What do I mean by that weasel wording? Is there some skullduggery going on? Why yes, there is. Computer languages are full of “as if” dirty deeds under the hood, like the money you deposit in your bank account isn’t actually there (I apologize if this is a rude shock to anyone), and this isn’t any different. Read on!

But first, a bit more necessary exposition.


## Folding Ownership/Borrowing into D


Isn’t this scheme incompatible with the way people normally write D code, and won’t it break pretty much every D program in existence? And not break them in an easily fixed way, but break them so badly they’ll have to redesign their algorithms from the ground up?

Yup, it sure is. Except that D has a (not so) secret weapon: function attributes. It turns out that the semantics for the Ownership/Borrowing (aka OB) system can be run on a per-function basis after the usual semantic pass has been run. The careful reader may have noticed that no new syntax is added, just restrictions on existing code. D has a history of using function attributes to alter the semantics of a function--for example, adding the `pure` attribute causes a function to behave as if it were pure. To enable OB semantics for a function, an attribute `@live` is added.

This means that OB can be added to D code incrementally, as needed, and as time and resources permit. It becomes possible to add OB while, and this is critical, keeping your project in a fully functioning, tested, and releasable state. It’s mechanically auditable how much of the project is memory safe in this manner. It adds to the list of D’s many other memory-safe guarantees (such as no pointers to the stack escaping).


## As If


Some necessary things cannot be done with strict OB, such as reference counted memory objects. After all, the whole point of an RC object is to have multiple pointers to it. Since RC objects are memory safe (if built correctly), they can work with OB without negatively impinging on memory safety. They just cannot be built with OB. The solution is that D has other attributes for functions, like `@system`. `@system` is where much of the safety checking is turned off. Of course, OB will also be turned off in `@system` code. It’s there that the RC object’s implementation hides from the OB checker.

But in OB code, the RC object looks to the OB checker like it is obeying the rules, so no problemo!

A number of such library types will be needed to successfully use OB.


## Conclusion


This article is a basic overview of OB. I am working on a much more comprehensive specification. It’s always possible I’ve missed something and that there’s a hole below the waterline, but so far it’s looking good. It’s a very exciting development for D and I’m looking forward to getting it implemented.

_For further discussion and comments from Walter, see the discussion threads on [the /r/programming subreddit](https://www.reddit.com/r/programming/comments/cdifbu/ownership_and_borrowing_in_d/) and [at Hacker News](https://news.ycombinator.com/item?id=20441519)._
