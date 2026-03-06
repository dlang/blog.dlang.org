---
author: MaxHaughton
comments: false
date: 2022-03-24 14:46:03+00:00
layout: post
link: https://dlang.org/blog/2022/03/24/reducing-template-compile-times/
slug: reducing-template-compile-times
title: Reducing Template Compile Times
wordpress_id: 3068
categories:
- Code
- Tutorials
permalink: /reducing-template-compile-times/
redirect_from: /2022/03/24/reducing-template-compile-times/
---

![]({{ '/assets/images/reducing-template-compile-times/cropped-rocket-200x200.png' | relative_url }})

Templates have been enormously profitable for the D programming language. They allow the programmer to generate efficient and correct code at compile time. Long gone are the days of preprocessor macros or handwritten, per-type data structures. D templates, though designed in the shadow of C++ templates, were not made in their image. D makes templates cleaner and more expressive, and also enables patterns like "[Design by Introspection](https://www.youtube.com/watch?v=HdzwvY8Mo-w)".

Here is a simple example of a template that would require the use of preprocessor macros in C or C++:


```d
template sizeOfTypeByName(string name)
{
  enum sizeOfTypeByName = mixin(name, ".sizeof");
}
unittest
{
  assert(sizeOfTypeByName!"int" == 4);
}
```
D's templates are powerful tools but should not be used unthinkingly. Carelessness could result in long compile times or excessive code generation.

In this blog post, I introduce some simple concepts that can help in writing templates that minimize resource usage. Deeper intuition can also lead to the discovery of new abstractions or increased confidence in existing ones.



## Read the memo



The D compiler memoizes template instantiations: if I instantiate `MyTemplate!int` once, the compiler produces an AST for that instantiation; if I instantiate that exact template again, the previous computation is reused.

As a demonstration, let's write a generic addition function and use `pragma(msg, ...)` to print the number of instantiations at compile time. I'm going to use it twice with integers and twice with floating point numbers.


```d
auto genericAdd(T)(const T x, const T y)
{
  pragma(msg, "genericAdd instantiated with ", T);
  return x + y;
}

// Instantiate with int
writeln(genericAdd!int(4, 5));
writeln(genericAdd!int(6, 1));

// Now for the float type
writeln(genericAdd!float(24.0, 32.0));
writeln(genericAdd!float(0.0, float.nan));
```
Now let's compile the code and look at what our `pragma(msg, )` says about template instantiations in the compiler.


```bash
dmd -c generic_add.d
```
This yields the following output during compilation:


    genericAdd instantiated with int
    genericAdd instantiated with float

We can see `int` and `float` as we expected, but notice that each is only mentioned once. Newcomers to languages with templates or generics can sometimes mistakenly think that using a template requires a potentially expensive instantiation on every use in the source code. For the benefit of those new users of D, the above is categoric proof that this is not the case; you _cannot_ pay twice for templates you have already asked the compiler to instantiate. (You can, however, convince yourself that you _are_ asking the compiler to do something it's already done when you in fact are not. We'll go over contrived and real-world examples of this later in this article.)

The benefit of this feature should be obvious, but what may not be obvious is how it can be employed in writing templates. Within the bounds of our desire for ergonomics, we should design the _interfaces_ of our templates to maximize the number of identical instantiations.



## What's in a name?



The following example, adapted from a real-world change to a large D project, yielded a reduction in compile time of a few percent for unit-test builds.

Let's say we have an expensive template whose behavior we want to test over a simple type. Our type might be:


```d
struct Vector3
{
  float x;
  float y;
  float z;
}
```
To demonstrate the phenomenon, we don't have to do anything fancy, so we'll just declare a stub called `send`.

```d
// Let's say this sends a value of type T to a database.
void send(T)(T x);
```
**A note on syntax:** Given a variable `val` of type `int`, this template could be explicitly instantiated as `send!(int)(val)`. However, the compiler can infer the type `T`, so we can instantiate it as if it were a normal function call as `send(val)`. Using D's [Uniform Function Call Syntax](https://dlang.org/spec/function.html#pseudo-member), we could alternatively call it like a property or member, as `val.send()` (the approach used in the following example), or even `val.send`, since parentheses are optional in function calls when there are no arguments.

Our test might then be something like:


```d
struct Vector3
{
  float x;
  float y;
  float z;
}

unittest
{
  Vector3 value;
  value.send();
}
```
This is reasonable so far. However, an issue arises when we start to write more than one test. Should we want to test different behaviors of a fancy template, but instantiate it with the same type, then we end up spending more time in compilation than we would have expected. A lot more time. And we see large growth in the number of symbols emitted in the resulting binary, resulting in a larger file size than one would expect. Why is that?

Despite our intuition that the compiler should consider multiple declarations of a type like `Vector3` in multiple `unittest` blocks as identical, it actually does not. We can demonstrate this effect with an extremely simple example. We'll provide an implementation of `send` that prints at compile time the type of each instantiation. Then [we'll use `static foreach`](https://dlang.org/spec/version.html#staticforeach) to generate five distinct implementations of a single unit test.


```d
void send(T)(T x)
{
  pragma(msg, T); 
}

// Generate 5 unittest blocks
static foreach(_; 0..5)
{
  unittest
  {
    struct JustInt
    {
      int x;
    }
    JustInt value;
    value.send;
  }
}
```
This results in the following output from the compiler:


    JustInt
    JustInt
    JustInt
    JustInt
    JustInt

Huh? Doesn't this violate our "you can't pay twice" rule? If you were to take this output from the compiler as gospel, then yes, but there's a more subtle truth here.



## Fully qualified names



The name of a type as you would write it in your editor is not the complete name of a type. Let's amend the implementation of `send` to print the return value of a template called `fullyQualifiedName` rather than printing `T` directly. The rest of the example remains the same.


```d
void send(T)(T x)
{
  import std.traits : fullyQualifiedName;
  pragma(msg, fullyQualifiedName!T); 
}
```
Assuming the module is named `example`, this yields something like:


```d
example.__unittest_L13_C3_1.JustInt
example.__unittest_L13_C3_2.JustInt
example.__unittest_L13_C3_3.JustInt
example.__unittest_L13_C3_4.JustInt
example.__unittest_L13_C3_5.JustInt
```
This explains our previous conundrum. By declaring the type _locally_ in each test, we have actually declared a new type per test, each of which results in a new instantiation.

A type's fully qualified name includes the name of its enclosing scope (`{package-name.}module-name.{scope-name(s).}TypeName`). The compiler rewrites each `unittest` as a unique function with a generated name. We have five unique functions, each with its own local, distinct declaration of a `JustInt` type. And so we end up with five distinct types.

We want to ensure that one instantiation is reused across `unittest` blocks. We do that by moving the declaration of `JustInt` to module scope, outside of the unit tests.


```d
struct JustInt
{
  int x;
}

static foreach(_; 0..5)
{
  unittest
  {
    JustInt value;
    value.send;
  }
}
```
The `send` template now prints:


    example.JustInt
Much better.



### Some hard data



To collect some anecdata about the usefulness of these changes, we'll look at compilation times and the size of compiled binaries. Since this template is _very_ trivial, let's generate a hundred copies of the same `unittest` rather than five so we can see a trend.

On my system, timing the compilation of our programs shows the locally declared types took **243ms** to compile, but the version with a single global type declaration took **159ms** to compile. A difference of **84ms** is not all that much, sure, but in a large codebase, there may be _a lot_ of these speedups waiting to be found. Any reduction in compile times is to be embraced, especially when it's cumulative.

As for binary size, I saw a savings of `69K` on disk. The quantity of machine code generated by the compiler is worth keeping a close eye on. Larger binaries mean more work for the linker, which in turn means more time waiting for builds to complete. The easiest job is the one you don't have to do.



## A more complex example



The following example demonstrates a very simple but fundamental change to a template that yields an enormous improvement in compile times and other metrics.

Let's say we have a fairly simple interpreter, and we want to expose functions in our D code to the scripts executed by our interpreter. We can do that with some sort of registration function, which we'll call `register`.



### The signature of the register function



To prove the point I'm discussing, we don't need to implement this function--its _interface_ is what can cause a big slow down.

Let's say our `register` function looks like this:

```d
// Context is something our hypothetical interpreter works with
void register(alias func, string registeredName)(Context x);
```
It's pretty reasonable, right? It takes a template alias parameter that specifies the function to call (a common idiom in D) and a template value parameter of type `string` that represents the name of the function as it is exposed to scripts. The implementation of `register` will presumably map the value of `registeredName` to the `func` alias, and then scripts can call the function using that value. Functions can be registered with, e.g., the following:


```d
context = createAContext();
context.register!(writeln!string, "writeln")();
```
The scripts can call the Phobos `writeln` function template using the name `writeln`.



### The compile-time performance of the register template



The interface for `register` looks harmless, but it turns out that it has a significant impact on compile time. We can test this by registering some random functions. The actual contents of the functions don't matter--this article is about template compile times, so we just want a baseline figure for roughly how much time the infrastructure templates take to compile rather than the code they are hooking together.

Although we will pull the functions out of a hat, the thing that will drive our intuition is to realize that a small number of interfaces will likely be reused many times. We could start with a basket of interface stubs like this:


```d
int stub1(string) { return 1; }

int stub2(string) { return 2; }

int stub3(string) { return 3; }

// etc.
```
More broadly, with a bunch of functions that have identical signatures and a bunch of functions with random parameter lists and return types, we can get a rough baseline. With the set of stubs I used, compile times ended up at roughly **5** seconds.

So what happens if we move the compile-time parameters to run time? Since `registeredName` is a template value parameter, we can just move it into the function parameter list with no change. We have to handle the `func` parameter differently. Almost any symbol can bind at compile time to a template alias parameter, but symbols can't bind at run time to function parameters. We have to use a function pointer instead. In that case, we can use the type of the referenced function as a template parameter.


```d
void register(FuncType)(Context x, FuncType ptr, string registeredName);
```
With this signature, the compile time drops to roughly **1** second.



### What's going on?



D is a fairly fast language to compile. Good decisions have been made over the lifetime of the language to make that possible. It is also the case that one can happily write slow-to-compile D code. Although we are choosing to ignore the compilation speeds of the non-infrastructure code to simplify the point being made, this can actually (in a certain sense) be the case in real projects, too. As such it is worthwhile to pay attention not to the quantity of metaprogramming being done _semantically_ but rather the quantity of metaprogramming being performed _by the compiler_.

In this case, with the first interface used for `register`, the compiler had no opportunity to reuse any instantiations. Because it accepted the registered functions as symbols, each instantiation was unique. By shifting instead to take the _type_ of the registered function as a template parameter and a pointer to the function as a function parameter, the compiler could reuse instantiations. `stub1`, `stub2`, and `stub3` are distinct symbols, but they each have the same type of (`int function(string)`).

To be clear, this is not an indictment of template alias parameters. There are good use cases for them (the Phobos algorithms API is an example). The point of this example is to show how the compile-time costs of unique template instantiations can be hidden. A decision about the trade-offs between compile-time and run-time performance can only be made if the programmer is aware there is a decision to make. So when implementing a template, consider how it will be used. If it's going to end up creating many unique instantiations, then you can weigh the benefits of keeping that interface versus redesigning it to maximize reuse.



## A false friend



In linguistics, [a false friend](https://en.wikipedia.org/wiki/False_friend) is a pair of words from two different languages that look the same but have different meanings. I'm going to abuse this term by using it to refer to a pair of programming patterns that actually result in the same program behavior, but via different routes through the language implementation, i.e., one of these patterns has, say, worse performance or compile times than the other.



### A simple example:



Let's say we have a library that exposes a template as part of its API, like this one that prints a string when a module is loaded during run-time initialization:


```d
template FunTemplate(string op)
{
  shared static this()
  {
    import std.stdio;
    writeln(op);
  }
}

// Use like this
mixin FunTemplate!"Hello from DLang";
```
Now let's say we want to refactor the library in some way that it's desirable to distinguish the name `FunTemplate` and its implementation. How would you go about doing that?

One way would be to tack `Impl` onto the implementation name, then declare an eponymous template that aliases the shortened name to the implementation name and forwards the template argument like this:


```d
template FunTemplate(string op)
{
  alias FunTemplate = FunTemplateImpl!op;
}
```
This does the job, but it also creates an additional instantiation for each different value of `op`, one instance of `FunTemplate`, and one of `FunTemplateImpl`. So if we instantiate it with, e.g., five different values, we end up with ten unique instantiations. Now imagine doing that with a template that's heavily used throughout a program.

Since we only want to provide an alternate name for the implementation and aren't doing anything to the parameter list, we can achieve the same result without adding another template into the mix: just use `alias` by itself.


```d
alias FunTemplate = FunTemplateImpl; 
```
Since `FunTemplate` is no longer a template, `FunTemplate!"Foo"` only creates the one instance of `FunTemplateImpl`.



## Normalization of template arguments



Once we know what we want a template to look like, and we're satisfied with the interface we want it to have, there are sometimes subtle ways to separate the interface and implementation of a template such that we can minimize the total amount of work the compiler has to do.

The definition of "work" in this context can be important to consider, as we can find ways to balance a tradeoff between compile times and the amount of object code generated for each instantiation. One technique to reduce these costs is by normalizing a given list of template arguments into something called a _canonical form_.



### Canonical Forms



A canonical form, resulting from a process called canonicalization, is a mathematical structure that is intended to reduce multiple different-looking but identical objects into _one_ form that we can then manipulate as we see fit. Using an automatic code formatter is an example of transforming input (in this case, source code) into a canonical form.



### Application to templates



Consider a template like this one:


```d
template Expensive(Args...)
{
  /* Some kind of expensive metaprogramming or code generation */
}
```
If we can think of a useful canonical form that isn't too hard to compute, we can then write a second template `Reduce` to implement it, then inject it like in the following example.


```d
template Expensive(Args...)
{
  // Reduce to some kind of canonical form
  alias reduced = Reduce!(Args);

  // Where ExpensiveImpl is the same as above but renamed
  alias Expensive = ExpensiveImpl!(reduced);
}
```
To be worth doing, `ExpensiveImpl` must be _significantly_ more expensive than the reduction operation (pay attention to differences in `sys` time when measuring this), where "significant" is meant statistically rather than informally, i.e., any win is good as long as you can rigorously prove it's real.



### An example: sorting template arguments



Take a templated aggregate like this:


```d
struct ExposeMethods(Types...)
{
  /* Some kind of internal state dependant on Types but not their order */

  static foreach(Type; Types) {
    bool test(Type x) { 
      /* Something slow to compile depending on Type */
    }
  }
}
```
If it's instantiated with, e.g., five different types across a large codebase, we could spend a lot of time redoing semantically identical compilation. If all possible types are used as input many times, we could end up with a few permutations, and if not we will probably get a few identical subsets.

A canonical form that might come to mind (i.e., a potential definition of `Reduce`) is simply sorting the arguments by their names. This can be achieved [via the use of `staticSort`](https://dlang.org/library/std/meta/static_sort.html).



## Conclusion



D has powerful metaprogramming and code generation features. But like anything in programming, their use isn't free. If you want to avoid the situation where you find yourself making coffee while your project builds, then it's imperative to be aware of the cost vs. benefits of the metaprogramming features you use. Then you can make intelligent decisions about your compile-time interfaces and implementations.



## Appendix - Tracing the D compiler to count template instantiations



Here's a simple lesson in Linux userspace tracing: you can use a tool like `bpftrace` or `DTrace` to spy on the D compiler compiling other things, so we can get basic figures about the compilation of other D programs without either hacking the compiler or changing their build process.

You'll need a `bpftrace` file like the following (saved as e.g., `main.bt`):


```
BEGIN
{
  printf("Tracing a D file\n");
}
uprobe:/home/mhh/dlang/dmd-2.097.0/linux/bin64/dmd:_Dmain
{
  printf("This is the main\n");
}
uprobe:/home/mhh/dlang/dmd-2.097.0/linux/bin64/dmd:_D3dmd10dsymbolsem24templateInstanceSemanticFCQBs9dtemplate16TemplateInstancePSQCz6dscope5ScopePSQDr4root5array__T5ArrayTCQEq10expression10ExpressionZQBkZv
{
  //We do nothing with the knowledge here but if you write some code you can get info about the templates relatively easily
  printf("Instantiating a template\n");
}
```
What am I hooking here? That big mangled name in the middle of the script is `templateInstanceSemantic` in the `dsymbolsem` module in the DMD source. By hooking it, we can get a rough idea of when a template is being worked on.

Running it with `sudo bpftrace main.bt` (eBPF tracing currently requires root) when building DMD, for example, I see there are about 50,800 template hits.

You can use a more complicated script in a system like `bcc` to reconstruct the compiler's internal data structures. With that, we can get output a bit more like `09:05:17 69310 b'/home/mhh/d_dev/dmd/src/dmd/errors.d:85'` and actually reconstruct the source/line info (alongside a timestamp and PID).
