---
author: DanielNielsen
comments: false
date: 2017-02-13 14:19:04+00:00
layout: post
link: https://dlang.org/blog/2017/02/13/a-new-import-idiom/
slug: a-new-import-idiom
title: A New Import Idiom
wordpress_id: 628
categories:
- Guest Posts
- The Language
permalink: /a-new-import-idiom/
redirect_from: /2017/02/13/a-new-import-idiom/
---

_Daniel Nielsen is an Embedded Software Engineer. He is currently using D in his spare time for an unpublished Roguelike and warns that he "may produce bursts of D Evangelism"._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)I remember one day in my youth, before the dawn of Internet, telling my teachers about "my" new algorithm, only to learn it had been discovered by the ancient Greeks in ~300 BC. This is the story of my life and probably of many who are reading this. It is easy to "invent" something; being the first, not so much!

Anyway, this is what all the fuss is about this time:

```d
template from(string moduleName)
{
  mixin("import from = " ~ moduleName ~ ";");
}
```
The TL;DR version: A new idiom to achieve even lazier imports.

Before the C programmers start running for the hills, please forget you ever got burned by C++ templates. The above snippet doesn't look that complicated, now does it? If you enjoy inventing new abstractions, take my advice and [give D a try](http://ddili.org/ders/d.en/index.html). Powerful, yet an ideal beginner's language. No need to be a template archwizard.

Before we proceed further, I'd like to call out Andrei Alexandrescu for identifying that there is a problem which needs solving. Please see his in depth motivation [in DIP 1005](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1005.md). Many thanks also to Dominikus Dittes Scherkl, who helped trigger the magic spark by making his own counter proposal and questioning if there really is a need to change the language specification in order to obtain Dependency-Carrying Declarations (DIP 1005).

D, like many modern languages, has [a fully fledged module system](http://dlang.org/spec/module.html) where symbols are directly imported (unlike the infamous C `#include`). This has ultimately resulted in the widespread use of local imports, limiting the scope as much as possible, in preference to the somewhat slower and less maintainable module-level imports:

```d
// A module-level import
import std.datetime;
  
void fun(SysTime time)
{
  import std.stdio; // A local import
  ...
}
```
Similar lazy import idioms are possible in other languages, for instance Python.

The observant among you might notice that because `SysTime` is used as the type of a function parameter, `std.datetime` must be imported at module level. Which brings us to the point of this blog post (and DIP 1005). How can we get around that?

```d
void fun(from!"std.datetime".SysTime time)
{
  import std.stdio;
  ...
}
```
There you have it, the Scherkl-Nielsen self-important lookup.

In order to fully understand what's going on, you may need to learn some D-isms. Let's break it down.



 	
  1. When [instantiating a template](http://dlang.org/spec/template.html) (via the `!` operator), if the **TemplateArgument** is one token long, the parentheses can be omitted from the template parameters. So `from!"std.datetime"` is the same as `from!("std.datetime")`. It may seem trivial, but you'd be surprised how much readability is improved by avoiding ubiquitous punctuation noise.

 	
  2. [Eponymous templates](https://dlang.org/spec/template.html#implicit_template_properties). The declaration of a template looks like this:

```d
template y() {
    int x;
}
```
With that, you have to type `y!().x` in order to reach the int. Oh, ze horror! Is that a smiley? Give me `x` already! That's exactly what eponymous templates accomplish:

```d
template x() {
    int x;
}
```
Now that the template and its only member have the same name, `x!().x` can be shortened to simply `x`.

 	
  3. [Renamed imports](https://dlang.org/spec/module.html#renamed_imports) allow accessing an imported module via a user-specified namespace. Here, `std.stdio` is imported normally:

```d
void printSomething(string s) {
    import std.stdio;
    writeln(s);           // The normal way
    std.stdio.writeln(s)  // An alternative using the fully qualified 
                          // symbol name, for disambiguation
}
```
Now it's imported and renamed as `io`:

```d
void printSomething(string s) {
    import io = std.stdio;
    io.writeln(s);         // Must be accessed like this.
    writeln(s);            // Error
    std.stdio.writeln(s);  // Error
}
```
Combining what we have so far:

```d
template dt() {
    import dt = std.datetime; 
}
void fun(dt!().SysTime time) {}
```
It works perfectly fine. The only thing which remains is to make it generic.

 	
  4. String concatenation is achieved with the `~` operator.

    string hey = "Hello," ~ " World!";
    assert(hey == "Hello, World!");
  5. [String mixins](https://dlang.org/mixin.html) put the power of a compiler writer at your fingertips. Let's generate code at compile time, then compile it. This is typically used for domain-specific languages (see [Pegged](https://github.com/PhilippeSigaud/Pegged) for one prominent use of a DSL in D), but in our simple case we only need to generate one single statement based on the name of the module we want to import. Putting it all together, we get the final form, allowing us to import any symbol from any module inline:

```d
template from(string moduleName)
{
  mixin("import from = " ~ moduleName ~ ";");
}
```
In the end, is it all really worth the effort? Using one comparison made by Jack Stouffer:

```d
import std.datetime;
import std.traits;

void func(T)(SysTime a, T value) if (isIntegral!T)
{
    import std.stdio : writeln;
    writeln(a, value);
}
```
Versus:

```d
void func(T)(from!"std.datetime".SysTime a, T value)
    if (from!"std.traits".isIntegral!T)
{
    import std.stdio : writeln;
    writeln(a, value);
}
```
In this particular case, the total compilation time dropped to ~30% of the original, while the binary size dropped to ~41% of the original.

What about the linker, I hear you cry? Sure, it can remove unused code. But it's not always as easy as it sounds, in particular due to module constructors (think `__attribute__((constructor))`). In either case, it's always more efficient to avoid generating unused code in the first place rather than removing it afterwards.

So this combination of D features was waiting there to be used, but somehow no one had stumbled on it before. I agreed with the need Andrei identified for Dependency-Carrying Declarations, yet I wanted even more. I wanted Dependency-Carrying _Expressions_. My primary motivation comes from being exposed to way too much legacy C89 code.

```c
void foo(void)
{
#ifdef XXX /* needed to silence unused variable warnings */
  int x;
#endif
... lots of code ...
#ifdef XXX
  x = bar();
#endif
}
```
Variables or modules, in the end they're all just symbols. For the same reason C99 allowed declaring variables in the middle of functions, one should be allowed to import modules where they are first used. D already allows importing anywhere in a scope, but not in declarations or expressions. It was with this mindset that I saw [Dominikus Dittes Scherkl's snippet](http://forum.dlang.org/thread/tzqzmqhankrkbrfsrmbo@forum.dlang.org?page=1):

```d
fun.ST fun()
{
   import someModule.SomeType;
   alias ST = SomeType;
   ...
}
```
Clever, yet for one thing it doesn't adhere to the DRY principle. Still, it was that tiny dot in `fun.ST` which caused the spark. There it was again, the Dependency-Carrying Expression of my dreams.

Criteria:



 	
  * It must not require repeating `fun`, since that causes problems when refactoring

 	
  * It must be lazy

 	
  * It must be possible today with no compiler updates


Templates are the poster children of lazy constructs; they don't generate any code until instantiated. So that seemed a good place to start.

Typically when using eponymous templates, you would have the template turn into a function, type, variable or alias. But why make the distinction? Once again, they're all just symbols in the end. We could have used an alias to the desired module (see Scherkl's snippet above); using the renamed imports feature is just a short-cut for import and alias. Maybe it was this simplified view of modules that made me see more clearly.

Now then, is this the only solution? No. As a challenge to the reader, try to figure out what this does and, more importantly, its flaw. Can you fix it?

    
    static struct STD
    {
      template opDispatch(string moduleName)
      {
        mixin("import opDispatch = std." ~ moduleName ~ ";");
      }
    }
    
    
    
