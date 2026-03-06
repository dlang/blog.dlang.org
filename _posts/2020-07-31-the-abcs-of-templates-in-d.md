---
author: DBlogAdmin
comments: false
date: 2020-07-31 13:35:32+00:00
excerpt: An introductory tutorial on templates in D. This post lays the foundation
  for future articles on more advanced template topics.
layout: post
link: https://dlang.org/blog/2020/07/31/the-abcs-of-templates-in-d/
slug: the-abcs-of-templates-in-d
title: The ABC's of Templates in D
wordpress_id: 2627
categories:
- Code
- The Language
- Tutorials
permalink: /the-abcs-of-templates-in-d/
redirect_from: /2020/07/31/the-abcs-of-templates-in-d/
---

D was not supposed to have templates.

Several months before Walter Bright released the first alpha version of the DMD compiler in December 2001, he completed a draft language specification in which he wrote:




Templates. A great idea in theory, but in practice leads to numbingly    complex code to implement trivial things like a "next" pointer in a singly linked list.




The (freely available) HOPL IV paper, [Origins of the D Programming Language](https://dl.acm.org/doi/abs/10.1145/3386323), expands on this:




[Walter] initially objected to templates on the grounds that they added disproportionate complexity to the front end, they could lead to overly complex user code, and, as he had heard many C++ users complain, they had a syntax that was difficult to understand. He would later be convinced to change his mind.




It did take some convincing. As activity grew in the (then singular) D newsgroup, requests that templates be added to the language became more frequent. Eventually, Walter started mulling over how to approach templates in a way that was less confusing for both the programmer [and the compiler](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1757.html). Then he had an epiphany: the difference between template parameters and function parameters is one of compile time vs. run time.

From this perspective, there's no need to introduce a special template syntax (like the C++ style `<T>`) when there's already a syntax for parameter lists in the form of `(T)`. So a template declaration in D could look like this:


```d
template foo(T, U) {
    // template members here
}
```
From there, the basic features fell into place and [were expanded and enhanced over time](https://dlang.org/spec/template.html).

In this article, we're going to lay the foundation for future articles by introducing the most basic concepts and terminology of D's template implementation. If you've never used templates before in any language, this may be confusing. That's not unexpected. Even though many find D's templates easier to understand than other implementations, the concept itself can still be confusing. You'll find links at the end to some tutorial resources to help build a better understanding.



## Template declarations



Inside a template declaration, one can nest any other valid D declaration:


```d
template foo(T, U) {
    int x;
    T y;

    struct Bar {
        U thing;
    }

    void doSomething(T t, U u) {
        ...
    }
}
```
In the above example, the parameters `T` and `U` are [template type parameters](https://dlang.org/spec/template.html#template_type_parameters), meaning they are generic substitutes for concrete types, which might be built-in types like `int`, `float`, or any type the programmer might implement with a `class` or `struct` declaration. By declaring a template, it's possible to, for example, write a single implementation of a function like `doSomething` that can accept multiple types for the same parameters. The compiler will generate as many copies of the function as it needs to accomodate the concrete types used in each unique template instantiation.

Other kinds of parameters are supported: [value parameters](https://dlang.org/spec/template.html#template_value_parameter), [alias parameters](https://dlang.org/spec/template.html#aliasparameters), [sequence (or variadic) parameters](https://dlang.org/spec/template.html#variadic-templates), and [this parameters](https://dlang.org/spec/template.html#template_this_parameter), all of which we'll explore in future blog posts.



### One name to rule them all



In practice, it's not very common to implement templates with multiple members. By far, the most common form of template declaration is the single-member eponymous template. Consider the following:


```d
template max(T) {
    T max(T a, T b) { ... }
}
```
An eponymous template can have multiple members that share the template name, but when there is only one, D provides us with an alternate template declaration syntax. In this example, we can opt for a normal function declaration that has the template parameter list in front of the function parameter list:


```d
T max(T)(T a, T b) { ... }
```
The same holds for eponymous templates that declare an aggregate type:


```d
// Instead of the longhand template declaration...
/* 
template MyStruct(T, U) {
    struct MyStruct { 
        T t;
        U u;
    }
}
*/

// ...just declare a struct with a type parameter list
struct MyStruct(T, U) {
    T t;
    U u;
}
```
Eponymous templates also offer a shortcut for instantiation, as we'll see in the next section.



## Instantiating templates



In relation to templates, the term _instantiate_ means essentially the same as it does in relation to classes and structs: create an instance of something. A template instance is, essentially, a concrete implementation of the template in which the template parameters have been replaced by the arguments provided in the instantiation. For a template function, that means a new copy of the function is generated, just as if the programmer had written it. For a type declaration, a new copy of the declaration is generated, just as if the programmer had written it.

We'll see an example, but first we need to see the syntax.



### Explicit instantiations



An _explicit instantiation_ is a template instance created by the programmer using the template instantiation syntax. To easily disambiguate template instantiations from function calls, D requires the template instantiation operator, `!`, to be present in explicit instantiations. If the template has multiple members, they can be accessed in the same manner that members of aggregates are accessed: using dot notation.


```d
import std;

template Temp(T, U) {
    T x;
    struct Pair {
        T t;
        U u;
    }
}

void main()
{
    Temp!(int, float).x = 10;
    Temp!(int, float).Pair p;
    p.t = 4;
    p.u = 3.2;
    writeln(Temp!(int, float).x);
    writeln(p);            
}
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/9BkV2d)

There is one template instantiation in this example: `Temp!(int, float)`. Although it appears three times, it refers to the same instance of `Temp`, one in which `T == int` and `U == float`. The compiler will generate declarations of `x` and the `Pair` type as if the programmer had written the following:


```d
int x;
struct Pair {
    int t;
    float u;
}
```
However, we can't just refer to `x` and `Pair` by themselves. We might have other instantiations of the same template, like `Temp!(double, long)`, or `Temp(MyStruct, short)`. To avoid conflict, the template members must be accessed through a namespace unique to each instantiation. In that regard, `Temp!(int, float)` is like a class or struct with static members; just as you would access a static `x` member variable in a struct `Foo` using the struct name, `Foo.x`, you access a template member using the template name, `Temp!(int, float).x`.

There is only ever one instance of the variable `x` for the instantiation `Temp!(int float)`, so no matter where you use it in a code base, you will always be reading and writing the same `x`. Hence, the first line of `main` isn't declaring and initializing the variable `x`, but is assigning to the already declared variable. `Temp!(int, float).Pair` is a struct type, so that after the declaration `Temp!(int, float).Pair p`, we can refer to `p` by itself. Unlike `x`, `p` is not a member of the template. The type `Pair` _is_ a member, so we can't refer to it without the prefix.



### Aliased instantiations



It's possible to simplify the syntax by using [an `alias` declaration](https://dlang.org/spec/declaration.html#alias) for the instantiation:


```d
import std;

template Temp(T, U) {
    T x;
    struct Pair {
        T t;
        U u;
    }
}
alias TempIF = Temp!(int, float);

void main()
{
    TempIF.x = 10;
    TempIF.Pair p = TempIF.Pair(4, 3.2);
    writeln(TempIF.x);
    writeln(p);            
}
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/nal7dV)

Since we no longer need to type the template argument list, using a struct literal to initialize `p`, in this case `TempIF.Pair(3, 3.2)`, looks cleaner than it would with the template arguments. So I opted to use that here rather than first declare `p` and then initialize its members. We can trim it down still more [using D's `auto` attribute](https://dlang.org/spec/attribute.html#auto), but whether this is cleaner is a matter of preference:


```d
auto p = TempIF.Pair(4, 3.2);
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/cLeE7X)



### Instantiating eponymous templates



Not only do eponymous templates have a shorthand declaration syntax, they also allow for a shorthand instantiation syntax. Let's take the `x` out of `Temp` and rename the template to `Pair`. We're left with a `Pair` template that provides a declaration `struct Pair`. Then we can take advantage of both the shorthand declaration and instantiation syntaxes:


```d
import std;

struct Pair(T, U) {
    T t;
    U u;
}

// We can instantiate Pair without the dot operator, but still use
// the alias to avoid writing the argument list every time
alias PairIF = Pair!(int, float);

void main()
{
    PairIF p = PairIF(4, 3.2);
    writeln(p);            
}
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/2X4DiW)

The shorthand instantiation syntax means we don't have to use the dot operator to access the `Pair` type.



### Even shorter syntax



When a template instantiation is passed only one argument, and the argument's symbol is a single token (e.g., `int` as opposed to `int[]` or `int*`), the parentheses can be dropped from the template argument list. Take [the standard library template function `std.conv.to`](https://dlang.org/phobos/std_conv.html#to) as an example:


```d
void main() {
    import std.stdio : writeln;
    import std.conv : to;
    writeln(to!(int)("42"));
}
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/uezRpo)

`std.conv.to` is an eponymous template, so we can use the shortened instantiation syntax. But the fact that we've instantiated it as an argument to the `writeln` function means we've got three pairs of parentheses in close proximity. That sort of thing can impair readability if it pops up too often. We could move it out and store it in a variable if we really care about it, but since we've only got one template argument, this is a good place to drop the parentheses from the template argument list.


```d
writeln(to!int("42"));
```
Whether that looks better is another case where it's up to preference, but it's fairly idiomatic these days to drop the parentheses for a single template argument no matter where the instantiation appears.



### Not done with the short stuff yet



`std.conv.to` is an interesting example because it's an eponymous template with multiple members that share the template name. That means that it must be declared using the longform syntax (as you can see [in the source code](https://github.com/dlang/phobos/blob/9f84d4ff2174c034c3dec4fb487d143ea7b326ba/std/conv.d#L218)), but we can still instantiate it without the dot notation. It's also interesting because, even though it accepts two template arguments, it is generally only instantiated with one. This is because the second template argument can be deduced by the compiler based on the function argument.

For a somewhat simpler example, [take a look at `std.utf.toUTF8`](https://dlang.org/phobos/std_utf.html#toUTF8):


```d
void main()
{    
    import std.stdio : writeln;
    import std.utf : toUTF8;
    string s1 = toUTF8("This was a UTF-16 string."w);
    string s2 = toUTF8("This was a UTF-32 string."d);
    writeln(s1);
    writeln(s2);
}
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/wQ3N4J)

Unlike `std.conv.to`, `toUTF8` takes exactly one parameter. The signature of the declaration looks like this:

```d
string toUTF8(S)(S s)
```
But in the example, we aren't passing a type as a template argument. Just as the compiler was able to deduce `to`'s second argument, it's able to deduce `toUTF8`'s sole argument.

`toUTF8` is an eponymous function template with a template parameter `S` and a function parameter `s` of type `S`. There are two things we can say about this: 1) the return type is independent of the template parameter and 2) the template parameter is the type of the function parameter. Because of 2), the compiler has all the information it needs from the function call itself and has no need for the template argument in the instantiation.

Take the first call to the `toUTF8` function in the declaration of `s1`. In long form, it would be `toUTF8!(wstring)("blah"w)`. The `w` at the end of the string literal indicates it is of type `wstring`, with UTF-16 encoding, as opposed to `string`, with UTF-8 encoding (the default for string literals). In this situation, having to specify `!(wstring)` in the template instantiation is completely redundant. The compiler already knows that the argument to the function is a `wstring`. It doesn't need the programmer to tell it that. So we can drop the template instantiation operator and the template argument list, leaving what looks like a simple function call. The compiler knows that `toUTF8` is a template, knows that the template is declared with one type parameter, and knows that the type should be `wstring`.

Similarly, the `d` suffix on the literal in the initialization of `s2` indicates a UTF-32 encoded `dstring`, and the compiler knows all it needs to know for that instantiation. So in this case also, we drop the template argument and make the instantiation appear as a function call.

It does seem silly to convert a `wstring` or `dstring` literal to a `string` when we could just drop the `w` and `d` prefixes and have `string` literals that we can directly assign to `s1` and `s2`. Contrived examples and all that. But the syntax the examples are demonstrating really shines when we work with variables.


```d
wstring ws = "This is a UTF-16 string"w;
string s = ws.toUTF8;
writeln(s);
```
[_Run it online at run.dlang.io_](https://run.dlang.io/is/8Z2CZ9)

Take a close look at the initialization of `s`. This combines the shorthand template instantiation syntax with Uniform Function Call Syntax (UFCS) and D's shorthand function call syntax. We've already seen the template syntax in this post. As for the other two:





  * [UFCS allows using dot notation](https://dlang.org/spec/function.html#pseudo-member) on the first argument of any function call so that it appears as if a member function is being called. By itself, it doesn't offer much benefit aside from, some believe, readability. Generally, it's a matter of preference. But this feature can seriously simplify the implementation of generic templates that operate on aggregate types and built-in types.


  * [The parentheses in a function call can be dropped](https://dlang.org/spec/function.html#optional-parenthesis) when the function takes no arguments, so that `foo()` becomes `foo`. In this case, the function takes one argument, but we've taken it out of the argument list using UFCS, so the argument list is now empty and the parentheses can be dropped. (The compiler will lower this to a normal function call, `toUTF8(ws)`--it's purely programmer convenience.) When and whether to do this in the general case is a matter of preference. The big win, again, is in the simplification of template implementations: a template can be implemented to accept a type `T` with a member variable `foo` or a member function `foo`, or a free function `foo` for which the first argument is of type `T`.



All of this shorthand syntax is employed to great effect [in D's range API](https://dlang.org/phobos/std_range.html), which allows chained function calls on types that are completely hidden from the public-facing API (aka [Voldemort types](https://wiki.dlang.org/Voldemort_types)).



## More to come



In future articles, we'll explore the different kinds of template parameters, introduce template constraints, see inside a template instantiation, and take a look at some of the ways people combine templates with D's powerful compile-time features in the real world. In the meantime, here are some template tutorial resources to keep you busy:





  * Ali Çehreli's [Programming in D](http://ddili.org/ders/d.en/) is an excellent introduction to D in general, suitable even for those with little programming experience. The two chapters on templates ([the first called 'Templates'](http://ddili.org/ders/d.en/templates.html) and [the second 'More Templates'](http://ddili.org/ders/d.en/templates_more.html)) provide a great introduction. (The online version of the book is free, but if you find it useful, please consider throwing Ali some love by buying the ebook or print version linked [at the top of the TOC](http://ddili.org/ders/d.en/index.html).)


  * More experienced programmers may find Phillipe Sigaud's ['D Template Tutorial'](https://github.com/PhilippeSigaud/D-templates-tutorial) a good read. It's almost a decade old, but still relevant (and still free!). This tutorial goes beyond the basics into some of the more advanced template features. It includes a look at D's compile-time features, provides a number of examples, and sports an appendix detailing [the `is` expression](https://dlang.org/spec/expression.html#is_expression) (a key component of template constraints). It can also serve as a great reference when reading open source D code until you learn your way around D templates.



There are other resources, though other than [my book 'Learning D'](https://amzn.to/37GpL2K) (this is a referral link that will benefit the D Language Foundation), I'm not aware of any that provide as much detail as the above. (And my book is currently not freely available). Eventually, we'll be able to add this blog series to the list.

_Thanks to Stefan Koch for reviewing this article._
