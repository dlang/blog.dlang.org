---
author: VictorPorton
comments: false
date: 2019-03-30 12:40:45+00:00
layout: post
link: https://dlang.org/blog/2019/03/30/memoization-in-the-d-programming-language/
slug: memoization-in-the-d-programming-language
title: Memoization in the D Programming Language
wordpress_id: 2018
categories:
- Code
- Guest Posts
permalink: /memoization-in-the-d-programming-language/
redirect_from: /2019/03/30/memoization-in-the-d-programming-language/
---

![](http://dlang.org/blog/wp-content/uploads/2019/03/brain02.png)The D programming language provides advanced facilities for structuring programs logically, almost like Python or Ruby, but with high performance and the higher reliability of static typing and contract programming.

In this article, I will describe how to use D templates and mixins for _memoization_, that is, to automatically remember a function (or property) result.


### `std.functional.memoize` from the standard library


The first way is built straight into [Phobos, the D standard library](https://dlang.org/phobos/index.html), and is very easy to use:

```d
import std.functional;
import std.stdio;

float doCalculations() {
    writeln("Doing calculations.");
    return -1.7; // the value of the calculations
}

// Apply the template “memoize” to the function doCalculations():
alias doCalculationsOnce = memoize!doCalculations;
```
Now the alias `doCalculationsOnce()` does the same as `doCalculations()`, but the calculations are done only once (if we call `doCalculationsOnce()` then “Doing calculations.” would be printed only once). This is useful for long slow calculations and in some other situations (like to create only a single window on the screen). That is memoization.

It’s even possible to memoize a function with arguments:

```d
float doCalculations2(float arg1, float arg2) {
    writeln("Doing calculations.");
    return arg1 + arg2; // the value of the calculations
}

// Apply the template “memoize” to the function  doCalculations2():
alias doCalculationsOnce2 = memoize!doCalculations2;

void main(string[] args)
{
    writeln(doCalculationsOnce2(1.0, 1.2));
    writeln(doCalculationsOnce2(1.0, 1.3));
    writeln(doCalculationsOnce2(1.0, 1.3));
}
```
This outputs:

    Doing calculations.
    2.2
    Doing calculations.
    2.3
    2.3
You see that the calculations are not repeated again when the argument values are the same.


### Memoizing `struct` or `class` properties


I’ve found another way to memoize in D. It involves caching a _property_ of a `struct` or `class`. Properties are zero-argument (for reading) or one-argument (for writing) member functions (or sometimes two-argument non-member functions) and differ only in syntax. I deemed it more elegant to cache a property of a struct or class rather than to cache a member function’s return value. My code can easily be changed to memoize a member function instead of a property, but you can always convert zero-argument member functions into a property, so why bother?

Here is the source (you can also install it from [https://github.com/vporton/memoize-dlang](https://github.com/vporton/memoize-dlang) for use in your programs):

```d
module memoize;

mixin template CachedProperty(string name, string baseName = '_' ~ name) {
    mixin("private typeof(" ~ baseName ~ ") " ~ name ~ "Cache;");
    mixin("private bool " ~ name ~ "IsCached = false;");
    mixin("@property typeof(" ~ baseName ~ ") " ~ name ~ "() {\n" ~
          "if (" ~ name ~ "IsCached" ~ ") return " ~ name ~ "Cache;\n" ~
          name ~ "IsCached = true;\n" ~
          "return " ~ name ~ "Cache = " ~ baseName ~ ";\n" ~
          '}');
}
```
It is used like this (with either structs or classes at your choice):

```d
import memoize;

struct S {
    @property float _x() { return 1.5; }
    mixin CachedProperty!"x";
}
```
Then just:

    S s;
    assert(s.x == 1.5);
Or you can specify the explicit name of the cached property:

```d
import memoize;

struct S {
    @property float _x() { return 1.5; }
    mixin CachedProperty!("x", "_x");
}
```
`CachedProperty` is [a template mixin](https://dlang.org/spec/template-mixin.html). Template mixins insert the declarations of the template body directly into the current context. In this case, the template body is composed [of string mixins](https://tour.dlang.org/tour/en/gems/string-mixins). As you can guess, a string mixin generates code at compile time from strings. So,

```d
struct S {
    @property float _x() { return 1.5; }
    mixin CachedProperty!"x";
}
```
turns into

```python
struct S {
    @property float _x() { return 1.5; }
    private typeof(_x) xCache;
    private bool xIsCached = false;
    @property typeof(_x) x() {
        if (xIsCached) return xCache;
        xIsCached = true;
        return xCache = _x;
    }
}
```
That is, it sets `xCache` to `_x` unless `xIsCached` and also sets `xIsCached` to `true` when retrieving `x`.

The idea originates from the following Python code:

```python
class cached_property(object):
    """A version of @property which caches the value.  On access, it calls the
    underlying function and sets the value in `__dict__` so future accesses
    will not re-call the property.
    """
    def __init__(self, f):
        self._fname = f.__name__
        self._f = f

    def __get__(self, obj, owner):
        assert obj is not None, 'call {} on an instance'.format(self._fname)
        ret = obj.__dict__[self._fname] = self._f(obj)
        return ret
```
My way of memoization does not (yet) involve caching return values of functions with arguments.



* * *



_Victor Porton is an open source developer, a math researcher, and a Christian writer. He earns his living as a programmer._
