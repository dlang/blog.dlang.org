---
author: DBlogAdmin
comments: false
date: 2018-11-06 15:07:53+00:00
excerpt: D programmers come from a variety of programming backgrounds, C-family languages
  perhaps being the most common among them. Understanding the differences and how
  familiar features are tailored to D can open the door to more possibilities for
  organizing a code base, and designing and implementing an API. This article is the
  first of a few that will examine D features that can be overlooked or misunderstood
  by those experienced in similar languages.
layout: post
link: https://dlang.org/blog/2018/11/06/lost-in-translation-encapsulation/
slug: lost-in-translation-encapsulation
title: 'Lost in Translation: Encapsulation'
wordpress_id: 1772
categories:
- Code
- The Language
permalink: /lost-in-translation-encapsulation/
redirect_from: /2018/11/06/lost-in-translation-encapsulation/
---

I first learned programming in BASIC. Outgrew it, and switched to Fortran. Amusingly, my early Fortran code looked just like BASIC. My early C code looked like Fortran. My early C++ code looked like C. – [Walter Bright, the creator of D](http://walterbright.com/)


Programming in a language is not the same as _thinking_ in that language. A natural side effect of experience with one programming language is that we view other languages through the prism of its features and idioms. Languages in the same family may look and feel similar, but there are guaranteed to be subtle differences that, when not accounted for, can lead to compiler errors, bugs, and missed opportunities. Even when good docs, books, and other materials are available, most misunderstandings are only going to be solved through trial-and-error.

D programmers come from a variety of programming backgrounds, C-family languages perhaps being the most common among them. Understanding the differences and how familiar features are tailored to D can open the door to more possibilities for organizing a code base, and designing and implementing an API. This article is the first of a few that will examine D features that can be overlooked or misunderstood by those experienced in similar languages.

We’re starting with a look at a particular feature that’s common among languages that support Object-Oriented Programming (OOP). There's one aspect in particular of the D implementation that experienced programmers are sure they already fully understand and are often surprised to later learn they don't.


## Encapsulation


Most readers will already be familiar with the concept of encapsulation, but I want to make sure we’re on the same page. For the purpose of this article, I’m talking about encapsulation in the form of separating interface from implementation. Some people tend to think of it strictly as it relates to object-oriented programming, but it’s a concept that’s more broad than that. Consider this C code:

```c
#include <stdio.h>
static size_t s_count;

void print_message(const char* msg) {
    puts(msg);
    s_count++;
}

size_t num_prints() { return s_count; }
```
In C, functions and global variables decorated with `static` become private to the _translation unit_ (i.e. the source file along with any headers brought in via `#include`) in which they are declared. Non-static declarations are publicly accessible, usually provided in header files that lay out the public API for clients to use. Static functions and variables are used to hide implementation details from the public API.

Encapsulation in C is a minimal approach. C++ supports the same feature, but it also has anonymous namespaces that can encapsulate type definitions in addition to declarations. Like Java, C#, and other languages that support OOP, C++ also has _access modifiers_ (alternatively known as access specifiers, protection attributes, visibility attributes) which can be applied to `class` and `struct` member declarations.

C++ supports the following three access modifiers, common among OOP languages:



 	
  * `public` - accessible to the world

 	
  * `private` - accessible only within the class

 	
  * `protected` - accessible only within the class and its derived classes


An experienced Java programmer might raise a hand to say, “Um, excuse me. That’s not a complete definition of `protected`.” That’s because in Java, it looks like this:



 	
  * `protected` - accessible within the class, its derived classes, and classes in the same package.


Every class in Java belongs to a package, so it makes sense to factor packages into the equation. Then there’s this:

 	
  * _package-private_ (not a keyword) - accessible within the class and classes in the same package.


This is the default access level in Java when no access modifier is specified. This combined with `protected` make packages a tool for encapsulation beyond classes in Java.

Similarly, C# has assemblies, [which MSDN defines as](https://msdn.microsoft.com/en-us/library/ms973231.aspx) “a collection of types and resources that forms a logical unit of functionality”. In C#, the meaning of `protected` is identical to that of C++, but the language has two additional forms of protection that relate to assemblies and that are analogous to Java’s `protected` and package-private.



 	
  * `internal` - accessible within the class and classes in the same assembly.

 	
  * `protected internal` - accessible within the class, its derived classes, and classes in the same assembly.


Examining encapsulation in other programming languages will continue to turn up similarities and differences. Common encapsulation idioms are generally adapted to language-specific features. The fundamental concept remains the same, but the scope and implementation vary. So it should come as no surprise that D also approaches encapsulation in its own, language-specific manner.


## Modules


The foundation of D’s approach to encapsulation [is the module](https://dlang.org/spec/module.html). Consider this D version of the C snippet from above:

```d
module mymod;

private size_t _count;

void printMessage(string msg) {
    import std.stdio : writeln;

    writeln(msg);
    _count++;
}

size_t numPrints() { return _count; }
```
In D, access modifiers can apply to module-scope declarations, not just `class` and `struct` members. `_count` is `private`, meaning it is not visible outside of the module. `printMessage` and `numPrints` have no access modifiers; they are `public` by default, making them visible and accessible outside of the module. Both functions could have been annotated with the keyword `public`.

_Note that imports in module scope are `private` by default, meaning the symbols in the imported modules are not visible outside the module, and local imports, as in the example, are never visible outside of their parent scope._

Alternative syntaxes are supported, giving more flexibility to the layout of a module. For example, there’s C++ style:

```d
module mymod;

// Everything below this is private until either another
// protection attribute or the end of file is encountered.
private:
    size_t _count;

// Turn public back on
public:
    void printMessage(string msg) {
        import std.stdio : writeln;

        writeln(msg);
        _count++;
    }

    size_t numPrints() { return _count; }
```
And this:

```d
module mymod;

private {
    // Everything declared within these braces is private.
    size_t _count;
}

// The functions are still public by default
void printMessage(string msg) {
    import std.stdio : writeln;

    writeln(msg);
    _count++;
}

size_t numPrints() { return _count; }
```
Modules can belong to packages. A package is a way to group related modules together. In practice, the source files corresponding to each module should be grouped together in the same directory on disk. Then, in the source file, each directory becomes part of the module declaration:

    // mypack/amodule.d
    mypack.amodule;
    
    // mypack/subpack/anothermodule.d
    mypack.subpack.anothermodule;
_Note that it’s possible to have package names that don’t correspond to directories and module names that don’t correspond to files, but it’s bad practice to do so. A deep dive into packages and modules will have to wait for a future post._

`mymod` does not belong to a package, as no packages were included in the module declaration. Inside `printMessage`, the function `writeln` is imported from the `stdio` module, which belongs to the `std` package. Packages have no special properties in D and primarily serve as namespaces, but they are a common part of the codescape.

In addition to `public` and `private`, the `package` access modifier can be applied to module-scope declarations to make them visible only within modules in the same package.

Consider the following example. There are three modules in three files (only one module per file is allowed), each belonging to the same root package.

```d
// src/rootpack/subpack1/mod2.d
module rootpack.subpack1.mod2;
import std.stdio;

package void sayHello() {
    writeln("Hello!");
}

// src/rootpack/subpack1/mod1.d
module rootpack.subpack1.mod1;
import rootpack.subpack1.mod2;

class Speaker {
    this() { sayHello(); }
}

// src/rootpack/app.d
module rootpack.app;
import rootpack.subpack1.mod1;

void main() {
    auto speaker = new Speaker;
}
```
Compile this with the following command line:

```bash
cd src
dmd -i rootpack/app.d
```
_The `-i` switch tells the compiler to automatically compile and link imported modules (excluding those in the standard library namespaces `core` and `std`). Without it, each module would have to be passed on the command line, else they wouldn’t be compiled and linked._

The class `Speaker` has access to `sayHello` because they belong to modules that are in the same package. Now imagine we do a refactor and we decide that it could be useful to have access to `sayHello` throughout the `rootpack` package. D provides the means to make that happen by allowing the `package` attribute to be parameterized with the fully-qualified name (FQN) of a package. So we can change the declaration of `sayHello` like so:

```d
package(rootpack) void sayHello() {
    writeln("Hello!");
}
```
Now all modules in `rootpack` and _all modules in packages that descend from `rootpack`_ will have access to `sayHello`. Don’t overlook that last part. A parameter to the `package` attribute is saying that a package and all of its descendants can access this symbol. It may sound overly broad, but it isn’t.

For one thing, only a package that is a direct ancestor of the module’s parent package can be used as a parameter. Consider a module `rootpack.subpack.subsub.mymod`. That name contains all of the packages that are legal parameters to the `package` attribute in `mymod.d`, namely `rootpack`, `subpack`, and `subsub`. So we can say the following about symbols declared in `mymod`:



 	
  * `package` - visible only to modules in the parent package of `mymod`, i.e. the `subsub` package.

 	
  * `package(subsub)` - visible to modules in the `subsub` package and modules in all packages descending from `subsub`.

 	
  * `package(subpack)` - visible to modules in the `subpack` package and modules in all packages descending from `subpack`.

 	
  * `package(rootpack`) - visible to modules in the `rootpack` package and modules in all packages descending from `rootpack`.


This feature makes packages another tool for encapsulation, allowing symbols to be hidden from the outside world but visible and accessible in specific subtrees of a package hierarchy. In practice, there are probably few cases where expanding access to a broad range of packages in an entire subtree is desirable.

It’s common to see parameterized package protection in situations where a package exposes a common public interface and hides implementations in one or more subpackages, such as a `graphics` package with subpackages containing implementations for DirectX, Metal, OpenGL, and Vulkan. Here, D’s access modifiers allow for three levels of encapsulation:



 	
  * the `graphics` package as a whole

 	
  * each subpackage containing the implementations

 	
  * individual modules in each package


Notice that I didn’t include `class` or `struct` types as a fourth level. The next section explains why.


## Classes and structs


Now we come to the motivation for this article. I can’t recall ever seeing anyone [come to the D forums](https://forum.dlang.org/) professing surprise about package protection, but the behavior of access modifiers in classes and structs is something that pops up now and then, largely because of expectations derived from experience in other languages.

Classes and structs use the same access modifiers as modules: `public`, `package`, `package(some.pack)`, and `private`. The `protected` attribute can only be used in classes, as inheritance is not supported for structs (nor for modules, which aren’t even objects). `public`, `package`, and `package(some.pack)` behave exactly as they do at the module level. The thing that surprises some people is that `private` also behaves the same way.

```d
import std.stdio;

class C {
    private int x;
}

void main() {
    C c = new C();
    c.x = 10;
    writeln(c.x);
}
```
_[Run this example online](https://run.dlang.io/is/L7geN6)_

Snippets like this are posted in the forums now and again by people exploring D, accompanying a question along the lines of, “Why does this compile?” (and sometimes, “I think I’ve found a bug!”). This is an example of where experience can cloud expectations. Everyone knows what `private` means, so it’s not something most people bother to look up in the language docs. However, [those who do would find this](https://dlang.org/spec/attribute.html#visibility_attributes):


Symbols with private visibility can only be accessed from within the same module.


`private` in D always means _private to the module_. The module is the lowest level of encapsulation. It’s easy to understand why some experience an initial resistance to this, that it breaks encapsulation, but the intent behind the design is to _strengthen_ encapsulation. It’s inspired by the C++ `friend` feature.

Having implemented and maintained a C++ compiler for many years, Walter understood the need for a feature like `friend`, but felt that it wasn’t the best way to go about it.


Being able to declare a “friend” that is somewhere in some other file runs against notions of encapsulation.


An alternative is to take a Java-like approach of one class per module, but he felt that was too restrictive.


One may desire a set of closely interrelated classes that encapsulate a concept, and those should go into a module.


So the way to view a module in D is not just as a single source file, but as a unit of encapsulation. It can contain free functions, classes, and structs, all operating on the same data declared in module scope and class scope. The public interface is still protected from changes to the private implementation inside the module. Along those same lines, `protected` class members are accessible not just in derived classes, but also in the module.

Sometimes though, there really is a benefit to denying access to private members in a module. The bigger a module becomes, the more of a burden it is to maintain, especially when it’s being maintained by a team. Every place a `private` member of a class is accessed in a module means more places to update when a change is made, thereby increasing the maintenance burden. The language provides the means to alleviate the burden in the form of [the special _package module_](https://dlang.org/spec/module.html#package-module).

In some cases, we don’t want to require the user to import multiple modules individually. Splitting a large module into smaller ones is one of those cases. Consider the following file tree:

    -- mypack
    ---- mod1.d
    ---- mod2.d
We have two modules in a package called `mypack`. Let’s say that `mod1.d` has grown extremely large and we’re starting to worry about maintaining it. For one, we want to ensure that private members aren’t manipulated outside of class declarations with hundreds or thousands of lines in between. We want to split the module into smaller ones, but at the same time we don’t want to break user code. Currently, users can get at the module’s symbols by importing it with `import mypack.mod1`. We want that to continue to work. Here’s how we do it:

    -- mypack
    ---- mod1
    ------ package.d
    ------ split1.d
    ------ split2.d
    ---- mod2.d
We’ve split `mod1.d` into two new modules and put them in a package named `mod1`. We’ve also created a special `package.d` file, which looks like this:

    module mypack.mod1;
    
    public import mypack.mod1.split1,
                  mypack.mod1.split2;
When the compiler sees `package.d`, it knows to treat it specially. Users will be able to continue using `import mypack.mod1` without ever caring that it’s now split into two modules in a new package. The key is the module declaration at the top of `package.d`. It’s telling the compiler to treat this package as the module `mod1`. And instead of automatically importing all modules in the package, the requirement to list them as public imports in `package.d` allows more freedom in implementing the package. Sometimes, you might want to require the user to explicitly import a module even when a `package.d` is present.

Now users will continue seeing `mod1` as a single module and can continue to import it as such. Meanwhile, encapsulation is now more stringently enforced internally. Because `split1` and `split2` are now separate modules, they can’t touch each other’s private parts. Any part of the API that needs to be shared by both modules can be annotated with `package` protection. Despite the internal transformation, the public interface remains unchanged, and encapsulation is maintained.


## Wrapping up


The full list of access modifiers in D can be defined as such:



 	
  * `public` - accessible everywhere.

 	
  * `package` - accessible to modules in the same package.

 	
  * `package(some.pack)` - accessible to modules in the package `some.pack` and to the modules in all of its descendant packages.

 	
  * `private` - accessible only in the module.

 	
  * `protected` (classes only) - accessible in the module and in derived classes.


Hopefully, this article has provided you with the perspective to think in D instead of your “native” language when thinking about encapsulation in D.

_Thanks to Ali Çehreli, Joakim Noah, and Nicholas Wilson for reviewing and providing feedback on this article._
