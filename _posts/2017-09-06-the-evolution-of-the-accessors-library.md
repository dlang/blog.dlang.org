---
author: RonnySpiegel
comments: false
date: 2017-09-06 13:22:17+00:00
excerpt: We've always used UML tools to visualize the internal structure and document
  details of software. That's true for me not only at Funkwerk, but also in the companies
  I worked before I joined the team here in Karlsfeld. One of the major issues of
  documentation is that at some point in time it will diverge from the actual implementation
  and become outdated. Additionally, if you have to support old versions of your components
  you will have to take care of old versions of your documentation as well.
layout: post
link: https://dlang.org/blog/2017/09/06/the-evolution-of-the-accessors-library/
slug: the-evolution-of-the-accessors-library
title: The Evolution of the accessors Library
wordpress_id: 1081
categories:
- Code
- Companies
- Guest Posts
permalink: /the-evolution-of-the-accessors-library/
redirect_from: /2017/09/06/the-evolution-of-the-accessors-library/
---

_[![](http://www.funkwerk.com/wp-content/themes/funkwerk/library/images/logo.png)](http://www.funkwerk.com)Ronny Spiegel is a developer at [Funkwerk AG](http://www.funkwerk.com/), a German company whose passenger information system is developed in D and was [recently highlighted on this blog](https://dlang.org/blog/2017/07/28/project-highlight-funkwerk/). In this post, Ronny tells the story of the company's open source [accessors library](https://github.com/funkwerk/accessors), which provides a mechanism for users to automatically generate property getters and setters using D's robust compile-time features._



* * *





### A little bit of history.


We've always used UML tools to visualize the internal structure and document details of software. That's true for me not only at Funkwerk, but also in the companies I worked before I joined the team here in Karlsfeld. One of the major issues of documentation is that at some point in time it will diverge from the actual implementation and become outdated. Additionally, if you have to support old versions of your components you will have to take care of old versions of your documentation as well.

The first approach to connecting code and model is to generate code from the model, which requires the model to reflect the current implementation. When I joined Funkwerk we were using [ArgoUML](http://argouml.tigris.org/) to manage class diagrams which were used as input to generate code. Not only class or struct skeletons were generated (existing code was kept), but also methods to access members which were not even part of the model. In order to control whether a member should be accessible, annotations, similar to UDAs ([User-Defined Attributes](https://dlang.org/spec/attribute.html#uda)), were used as part of the member documentation. So it was very common for us to annotate a member with `@Read` or `@Write` even though it was only in the documentation. The tool which we used to generate code was powerful enough to create the implementation of these field accessor methods supported by annotations to synchronize access, or to automatically use invariants for pre- and post-conditions as well.

Anyway, the approach of using the model as a base for code generation always suffers from the same problem: it is very hard to merge models!

So we reversed the whole thing and decided to create documentation from code. We could still use code which had been generated before, but all the new classes had to be supplied with accessor functions. You can imagine that this was very annoying.

```d
public class Journey
{
    private Leg[] legs_;

    public Leg[] legs()
    {
	return this.legs_.dup;
    }

...
}
```
(Yes, we’ve been writing Java and compiling as D.)

Code which was generated before still had these `@Read` and `@Write` annotations next to the fields. So I thought, "These look like UDAs. Why not just use those to generate the methods automatically?" I'd always wanted to use mixins and compile-time introspection in order to move forward with a more D-like development approach.


### A first draft...


The very first version of the accessors library was able to generate basic read- and write-accessor methods using the [`allMembers` trait](https://dlang.org/spec/traits.html#allMembers), filtering by UDAs, and generating some basic code like:

```d
public final Leg[] legs() { return this.legs_.dup; }
```
It works... Yes, it does.

We did not replace all existing accessor methods at once, but working on a large project at that time we introduced many of them. The automated generation of accessor methods was really a simplification for us.


### ...always has some issues.


The first implementation looked so simple - there must have been issues. And yes, there were. I cannot list all of them because I do not remember anymore, but some of these issues were:


#### Explicitly defined properties suppressed generated ones


We ran into a situation where we explicitly defined a setter method (e.g. because it had to notify an observer) but wanted to use the generated getter method. The result was that the defined setter method could be used but accessing the generated getter method (with the same name) was impossible.

According to [the specification](https://dlang.org/spec/template-mixin.html), the compiler places mixins in a nested scope and then imports them into the surrounding scope. If a function with the same name already exists in the surrounding scope, then this function overwrites the function from the mixin. So if there is a field with a `@Read` annotation and another explicitly defined mutating field accessor, then the `@Read` accessor is overwritten by the defined one.

The solution to this issue was rather simple. We had to use a [string mixin](https://dlang.org/mixin.html) to import the generated code into the class where it shall be used.


#### Flags


We have a guideline to avoid magic `bool`s wherever possible and use much more verbose flags instead. So a simple attribute like:

    private bool isExtraJourney_;
Becomes:

    private Flag!”isExtraJourney” isExtraJourney_;
This approach has two advantages. Providing a value with `Yes.isExtraJourney` is much more verbose than just a `true`, and it creates a type. When there are two or more flags as part of a method signature, you cannot change the order of the flags (by accident) as you could with `bool`s.

To generate the type of the return value (or in case of mutable access of the parameter) we used `T.stringof`, where `T` is the type of the field. Unfortunately, this does not work as expected for Flags.

    Flag!”foo” fooFlag;
    
    static assert(`Flag!”foo”`, typeof(fooFlag).stringof); // Fails!
    static assert(`Flag`, typeof(fooFlag).stringof); // Succeeds!
#### Unit Tests


When using the mixin in private types defined in unit tests, a similar issue arose. Classes defined in unittest blocks have a prefix like `__unittestL526_8`. It was necessary to strip this prefix from the used type string.


#### Private Classes


While iterating over members of private classes, we stumbled across the issue that the `allMembers` (or [`derivedMembers`](https://dlang.org/spec/traits.html#derivedMembers)) trait returns, in addition to __ctor, an unaccessible member called `this`. This issue [remains unsolved](https://issues.dlang.org/show_bug.cgi?id=14740).


### The current implementation...


The [currently released version](https://github.com/funkwerk/accessors) covers the aforementioned issues, although there is still room for new features.

An example might be to provide a predicate which is then used for synchronizing access to the field. That was possible using the old version of the code generator by annotating it with `@GuardedBy(“this”)`. Fortunately, we've advanced in our D coding skills and have moved away from Java code compiled with DMD to a more D-like style by using structs wherever we need value semantics (and we don’t have to deal with thousands of copies of that value). So at the moment, this doesn’t really hurt that much.

Another interesting (and [still open issue](https://github.com/funkwerk/accessors/issues/11)) is to create accessors for aliased imported types. The generated code still refers to the real name of the type, which is then unknown to the compile unit where the code is mixed in.


### ...has room for improvement!


If you’re interested in dealing with this kind of problem and want to dive into CTFE and compile-time introspection, we welcome contributions!
