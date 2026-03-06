---
author: NickSab
comments: false
date: 2018-05-21 14:29:49+00:00
excerpt: Nick Sabalausky presents a simple and effective D tip for improving error
  messages involving aliased types.
layout: post
link: https://dlang.org/blog/2018/05/21/complicated-types-prefer-alias-this-over-alias-for-easier-to-read-error-messages/
slug: complicated-types-prefer-alias-this-over-alias-for-easier-to-read-error-messages
title: 'Complicated Types: Prefer "alias this" Over "alias" For Easier-To-Read Error
  Messages'
wordpress_id: 1534
categories:
- Code
- Tutorials
permalink: /complicated-types-prefer-alias-this-over-alias-for-easier-to-read-error-messages/
redirect_from: /2018/05/21/complicated-types-prefer-alias-this-over-alias-for-easier-to-read-error-messages/
---

_Nick Sabalausky is a long-time D user and contributor. He is the maintainer of [mysql-native](https://github.com/mysql-d/mysql-native) and [Scriptlike](https://github.com/Abscissa/scriptlike). In this post, he presents a way to use a specific D language feature to improve error messages involving aliased types._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)In [the D programming language](http://dlang.org), `alias` is a common and handy feature that can be used to provide a simple name for a complex and verbose templated type.

As an example, consider the case of an algebraic type or tagged union:

    // A type that can be either an int or a string
    Algebraic!(int, string) someVariable;
That’s a fairly simple example. Much more complicated type names are common in D. This sort of thing can be a pain to repeat everywhere it’s used, and can make code difficult to read. `alias` is often used in situations like this to create a simpler shorthand name:

    // A type that can be either an int or a string
    alias MyType = Algebraic!(int, string);
    
    // Ahh, much nicer!
    MyType someVariable;
There’s one problem this still doesn’t solve. Anytime a compiler error message shows a type name, it shows the _full_ original name, not the convenient easy-to-read alias. Instead of errors saying `MyType`, they’ll still say `Algebraic!(int, string)`. This can be especially unfriendly if `MyType` is in the public API of a library and happens to be constructed using some private, internal-only template.

That can be fixed, and error messages forced to provide the customized name, by creating `MyType` as a separate type on its own, rather than an alias. But how? If this was C or C++, `typedef` would do the job nicely. There is a D equivalent, `std.typecons.Typedef!T`, which will create a separate type. But naming the type still involves `alias`, which just leads back to the same problem.

Luckily, D has another feature which can help simulate a C-style `typedef`: `alias this`. Used inside a struct (or class), `alias this` allows the struct to be implicitly converted to and behave just like any one of its members.

Incidentally, although `alias` and `alias this` are separate features of the language, they do have a shared history as their names suggest. Originally, `alias` was intended to be a variation on C’s `typedef`, one which would result in two names for the same type instead of two separate types. At the time, D had `typedef` as well, but it was eventually dropped as a language feature in favor of a standard library solution (the aforementioned `std.typecons.Typedef` template). As a variant of `typedef`, `alias` used the same syntax (`alias TypeName AliasName;`). Later, `alias` spawned the `alias this` feature, which was given a similar syntax: `alias memberName this`. When `alias` gained its modern syntax (`alias AliasName = TypeName`), a lengthy debate resulted in keeping the existing syntax for `alias this`.

Here is how `alias this` can be used to solve our problem:

```d
// A type that can be either an int or a string
struct MyType {
    private Algebraic!(int, string) _data;
    alias _data this;
}

// Ahh, much nicer! And now error messages say "MyType"!
MyType someVariable;
```
There’s an important difference to be aware of, though. Before, when using `alias`, `MyType` and `Algebraic!(int, string)` were considered the same type. Now, they’re not. Is that a problem? What does that imply? Mainly, it means two things:



 	
  1. 

 	
```d
1. Although this doesn’t affect any actual code, it _can_ mean the compiler generates extra, duplicate template instantiations. If `MyType` is passed to one template, and somewhere else `Algebraic!(int, string)` is passed to the same template, the compiler will now generate two separate template instantiations instead of just one.
```
In practice though, this shouldn’t be a problem unless you’re already in a genuine template-bloat situation and are trying to reduce template instantiations. Usually, this won’t be an issue.

 	
    2. Although the `alias this` means `MyType` can still be implicitly converted to `Algebraic!(int, string)`, the other way around no longer works. An `Algebraic!(int, string)` can no longer be implicitly converted to a `MyType`.
Arguably, this can be considered a good thing if you believe, as I do, in using [domain-specific types](http://programmer.97things.oreilly.com/wiki/index.php/Prefer_Domain-Specific_Types_to_Primitive_Types). But in any case, you can still manually convert the original type to your `MyType` with the basic built-in struct constructor:

```d
Algebraic!(int, string) algebVar;
auto myVar = MyType(algebVar);
```
So when you’re aliasing a large, complicated type name to a simpler name, consider using a struct and `alias this` instead, especially if it’s a type on offer in a library. There’s little downside, and it will greatly improve the readability of error messages for both yourself and your library’s users.
