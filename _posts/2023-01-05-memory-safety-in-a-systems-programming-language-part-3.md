---
author: AteEskola
comments: false
date: 2023-01-05 12:03:51+00:00
layout: post
link: https://dlang.org/blog/2023/01/05/memory-safety-in-a-systems-programming-language-part-3/
slug: memory-safety-in-a-systems-programming-language-part-3
title: Memory Safety in a Systems Programming Language Part 3
wordpress_id: 3128
categories:
- Code
- DIPs
- Guest Posts
- The Language
- Tutorials
permalink: /memory-safety-in-a-systems-programming-language-part-3/
redirect_from: /2023/01/05/memory-safety-in-a-systems-programming-language-part-3/
---

![]({{ '/assets/images/memory-safety-in-a-systems-programming-language-part-3/brain02.png' | relative_url }})

[The first entry in this series](https://dlang.org/blog/2022/06/21/dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/) shows how to use the new DIP1000 rules to have slices and pointers refer to the stack, all while being memory safe. [The second entry in this series](https://dlang.org/blog/2022/10/08/dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/) teaches about the `ref` storage class and how DIP1000 works with aggregate types (classes, structs, and unions).

So far the series has deliberately avoided templates and `auto` functions. This kept the first two posts simpler in that they did not have to deal with function attribute inference, which I have referred to as "attribute auto inference" in earlier posts. However, both `auto` functions and templates are very common in D code, so a series on DIP1000 can't be complete without explaining how those features work with the language changes. Function attribute inference is our most important tool in avoiding so-called "attribute soup", where a function is decorated with several attributes, which arguably decreases readability.

We will also dig deeper into unsafe code. The previous two posts in this series focused on the `scope` attribute, but this post is more focused on attributes and memory safety in general. Since DIP1000 is ultimately about memory safety, we can't get around discussing those topics.



## Avoiding repetition with attributes



[Function attribute inference](https://dlang.org/spec/function.html#function-attribute-inference) means that the language will analyze the body of a function and will automatically add the `@safe`, `pure`, `nothrow`, and `@nogc` attributes where applicable. It will also attempt to add `scope` or `return scope` attributes to parameters and `return ref` to `ref` parameters that can't otherwise be compiled. Some attributes are never inferred. For instance, the compiler will not insert any `ref`, `lazy`, `out` or `@trusted` attributes, because very likely they are explicitly not wanted where they are left out.

There are many ways to turn on function attribute inference. One is by omitting the return type in the function signature. Note that the `auto` keyword is not required for this. `auto` is a placeholder keyword used when no return type, storage class, or attribute is specified. For example, the declaration `half(int x) { return x/2; }` does not parse, so we use `auto half(int x) { return x/2; }` instead. But we could just as well write `@safe half(int x) { return x/2; }` and the rest of the attributes (`pure`, `nothrow`, and `@nogc`) will be inferred just as they are with the `auto` keyword.

The second way to enable attribute inference is to templatize the function. With our `half` example, it can be done this way:


```d
int divide(int denominator)(int x) { return x/denominator; }
alias half = divide!2;
```
The D spec does not say that a template must have any parameters. An empty parameter list can be used to turn attribute inference on: `int half()(int x) { return x/2; }`. Calling this function doesn't even require the template instantiation syntax at the call site, e.g., `half!()(12)` is not required as `half(12)` will compile.

Another means to turn on attribute inference is to store the function inside another function. [These are called nested functions](https://dlang.org/spec/function.html#nested). Inference is enabled not only on functions nested directly inside another function but also on most things nested in a type or a template inside the function. Example:


```d
@safe void parentFun()
{
    // This is auto-inferred.
    int half(int x){ return x/2; }

    class NestedType
    {
        // This is auto inferred
        final int half1(int x) { return x/2; }

        // This is not auto inferred; it's a
        // virtual function and the compiler
        // can't know if it has an unsafe override
        // in a derived class.
        int half2(int x) { return x/2; }
    }

    int a = half(12); // Works. Inferred as @safe.
    auto cl = new NestedType;
    int b = cl.half1(18); // Works. Inferred as @safe.
    int c = cl.half2(26); // Error.
}
```
A downside of nested functions is that they can only be used in lexical order (the call site must be below the function declaration) unless both the nested function and the call are inside the same struct, class, union, or template that is in turn inside the parent function. Another downside is that they don't work with [Uniform Function Call Syntax](https://dlang.org/spec/function.html#pseudo-member).

Finally, attribute inference is always enabled for [function literals](https://dlang.org/spec/expression.html#FunctionLiteral) (a.k.a. lambda functions). The halving function would be defined as `enum half = (int x) => x/2;` and called exactly as normal. However, the language does not consider this declaration a function. It considers it a function pointer. This means that in global scope it's important to use `enum` or `immutable` instead of `auto`. Otherwise, the lambda can be changed to something else from anywhere in the program and cannot be accessed from `pure` functions. In rare cases, such mutability can be desirable, but most often it is an antipattern (like global variables in general).



### Limits of inference



Aiming for minimal manual typing isn't always wise. Neither is aiming for maximal attribute bloat.

The primary problem of auto inference is that subtle changes in the code can lead to inferred attributes turning on and off in an uncontrolled manner. To see when it matters, we need to have an idea of what will be inferred and what will not.

The compiler in general will go to great lengths to infer `@safe`, `pure`, `nothrow`, and `@nogc` attributes. If your function _can_ have those, it almost always will. The specification says that recursion is an exception: a function calling itself should not be `@safe`, `pure`, or `nothrow` unless explicitly specified as such. But in my testing, I found those attributes actually are inferred for recursive functions. It turns out, there is an ongoing effort to get recursive attribute inference working, and it partially works already.

Inference of `scope` and `return` on function parameters is less reliable. In the most mundane cases, it'll work, but the compiler gives up pretty quickly. The smarter the inference engine is, the more time it takes to compile, so the current design decision is to infer those attributes in only the simplest of cases.



### Where to let the compiler infer?



A D programmer should get into the habit of asking, "What will happen if I mistakenly do something that makes this function unsafe, impure, throwing, garbage-collecting, or escaping?" If the answer is "immediate compiler error", auto inference is probably fine. On the other hand, the answer could be "user code will break when updating this library I'm maintaining". In that case, annotate manually.

In addition to the potential of losing attributes the author intends to apply, there is also another risk:


```d
@safe pure nothrow @nogc firstNewline(string from)
{
    foreach(i; 0 .. from.length) switch(from[i])
    {
        case '\r':
        if(from.length > i+1 && from[i+1] == '\n')
        {
            return "\r\n";
        }
        else return "\r";

        case '\n': return "\n";

        default: break;
    }

    return "";
}
```
You might think that since the author is manually specifying the attributes, there's no problem. Unfortunately, that's wrong. Suppose the author decides to rewrite the function such that all the return values are slices of the `from` parameter rather than string literals:


```d
@safe pure nothrow @nogc firstNewline(string from)
{
    foreach(i; 0 .. from.length) switch(from[i])
    {
        case '\r':
        if (from.length > i + 1 && from[i + 1] == '\n')
        {
            return from[i .. i + 2];
        }
        else return from[i .. i + 1];

        case '\n': return from[i .. i + 1];

        default: break;
    }

    return "";
}
```
Surprise! The parameter `from` was previously inferred as `scope`, and a library user was relying on that, but now it's inferred as `return scope` instead, breaking client code.

Still, for internal functions, auto inference is a great way to save both our fingers when writing and our eyes when reading. Note that it's perfectly fine to rely on auto inference of the `@safe` attribute as long as the function is used in explicitly in `@safe` functions or unit tests. If something potentially unsafe is done inside the auto-inferred function, it gets inferred as `@system`, not `@trusted`. Calling a `@system` function from a `@safe` function results in a compiler error, meaning auto inference is safe to rely on in this case.

It still sometimes makes sense to manually apply attributes to internal functions, because the error messages generated when they are violated tend to be better with manual attributes.



### What about templates?



Auto inference is always enabled for templated functions. What if a library interface needs to expose one? There is a way to block the inference, albeit an ugly one:


```d
private template FunContainer(T)
{
    // Not auto inferred
    // (only eponymous template functions are)
    @safe T fun(T arg){return arg + 3;}
}

// Auto inferred per se, but since the function it calls
// is not, only @safe is inferred.
auto addThree(T)(T arg){return FunContainer!T.fun(arg);}
```
However, which attributes a template should have often depends on its compile-time parameters. It would be possible to use metaprogramming to designate attributes depending on the template parameters, but that would be a lot of work, hard to read, and easily as error-prone as relying on auto inference.

It's more practical to just test that the function template infers the wanted attributes. Such testing doesn't have to, and probably shouldn't, be done manually each time the function is changed. Instead:


```d
float multiplyResults(alias fun)(float[] arr)
    if (is(typeof(fun(new float)) : float))
{
    float result = 1.0f;
    foreach (ref e; arr) result *= fun(&e);
    return result;
}

@safe pure nothrow unittest
{
    float fun(float* x){return *x+1;}
    // Using a static array makes sure
    // arr argument is inferred as scope or
    // return scope
    float[5] elements = [1.0f, 2.0f, 3.0f, 4.0f, 5.0f];

    // No need to actually do anything with
    // the result. The idea is that since this
    // compiles, multiplyResults is proven @safe
    // pure nothrow, and its argument is scope or
    // return scope.
    multiplyResults!fun(elements);
}
```
Thanks to D's compile-time introspection powers, testing against unwanted attributes is also covered:


```d
@safe unittest
{
    import std.traits : attr = FunctionAttribute,
        functionAttributes, isSafe;

    float fun(float* x)
    {
        // Makes the function both throwing
        // and garbage collector dependant.
        if (*x > 5) throw new Exception("");
        static float* impureVar;

        // Makes the function impure.
        auto result = impureVar? *impureVar: 5;

        // Makes the argument unscoped.
        impureVar = x;
        return result;
    }

    enum attrs = functionAttributes!(multiplyResults!fun);

    assert(!(attrs & attr.nothrow_));
    assert(!(attrs & attr.nogc));

    // Checks against accepting scope arguments.
    // Note that this check does not work with
    // @system functions.
    assert(!isSafe!(
    {
        float[5] stackFloats;
        multiplyResults!fun(stackFloats[]);
    }));

    // It's a good idea to do positive tests with
    // similar methods to make sure the tests above
    // would fail if the tested function had the
    // wrong attributes.
    assert(attrs & attr.safe);
    assert(isSafe!(
    {
        float[] heapFloats;
        multiplyResults!fun(heapFloats[]);
    }));
}
```
If assertion failures are wanted at compile time before the unit tests are run, adding the `static` keyword before each of those `assert`s will get the job done. Those compiler errors can even be had in non-unittest builds by converting that unit test to a regular function, e.g., by replacing `@safe unittest` with, say, `private @safe testAttrs()`.



## The live fire exercise: @system



Let's not forget that D is a systems programming language. As this series has shown, in most D code the programmer is well protected from memory errors, but D would not be D if it didn't allow going low-level and bypassing the type system in the same manner as C or C++: bit arithmetic on pointers, writing and reading directly to hardware ports, executing a `struct` destructor on a raw byte blob… D is designed to do all of that.

The difference is that in C and C++ it takes only one mistake to break the type system and [cause undefined behavior](https://blog.regehr.org/archives/213) anywhere in the code. A D programmer is only at risk when not in a `@safe` function, or when using dangerous compiler switches such as `-release` or `-check=assert=off` (failing a disabled assertion is undefined behavior), and even then the semantics tend to be less UB-prone. For example:


```d
float cube(float arg)
{
    float result;
    result *= arg;
    result *= arg;
    return result;
}
```
This is a language-agnostic function that compiles in C, C++, and D. Someone intended to calculate the cube of `arg` but forgot to initialize `result` with `arg`. In D, nothing dangerous happens despite this being a `@system` function. No initialization value means `result` is default initialized to `NaN` (not-a-number), which leads to the result also being `NaN`, which is a glaringly obvious "error" value when using this function the first time.

However, in C and C++, not initializing a local variable means reading it is (sans a few narrow exceptions) _undefined behavior_. This function does not even handle pointers, yet according to the standard, calling this function could just as well have `*(int*) rand() = 0XDEADBEEF;` in it, all due to a trivial mistake. While many compilers with enabled warnings will catch this one, not all do, and these languages are full of similar examples where even warnings don't help.

In D, even if you explicitly requested no default initialization with `float result = void`, it'd just mean the return value of the function is undefined, not anything and everything that happens if the function is called. Consequently, that function could be annotated `@safe` even with such an initializer.

Still, for anyone who cares about memory safety, as they probably should for anything intended for a wide audience, it's a bad idea to assume that D `@system` code is safe enough to be the default mode. Two examples will demonstrate what can happen.



### What undefined behavior can do



Some people assume that "Undefined Behavior" simply means "erroneous behavior" or crashing at runtime. While that is often what ultimately happens, undefined behavior is far more dangerous than, say, an uncaught exception or an infinite loop. The difference is that with undefined behavior, you have no guarantees at all about what happens. This might not sound any worse than an infinite loop, but an accidental infinite loop is discovered the first time it's entered. Code with undefined behavior, on the other hand, might do what was intended when it's tested, but then do something completely different in production. Even if the code is tested with the same flags it's compiled with in production, the behavior may change from one compiler version to another, or when making completely unrelated changes to the code. Time for an example:


```d
// return whether the exception itself is in the array
bool replaceExceptions(Object[] arr, ref Exception e)
{
    bool result;
    foreach (ref o; arr)
    {
        if (&o is &e) result = true;
        if (cast(Exception) o) o = e;
    }

    return result;
}
```
The idea here is that the function replaces all exceptions in the array with `e`. If `e` itself is in the array, it returns `true`, otherwise `false`. And indeed, testing confirms it works. The function is used like this:


```d
auto arr = [new Exception("a"), null, null, new Exception("c")];
auto result = replaceExceptions
(
    cast(Object[]) arr,
    arr[3]
);
```
This cast is not a problem, right? Object references are always of the same size regardless of their type, and we're casting the exceptions to the parent type, `Object`. It's not like the array contains anything other than object references.

Unfortunately, that's not how [the D specification views it](https://dlang.org/spec/expression.html#AssignExpression). Having two class references (or any references, for that matter) in the same memory location but with different types, and then assigning one of them to the other, is undefined behavior. That's exactly what happens in


```d
if (cast(Exception) o) o = e;
```
if the array does contain the `e` argument. Since `true` can only be returned when undefined behavior is triggered, it means that any compiler would be free to optimize `replaceExceptions` to always return `false`. This is a dormant bug no amount of testing will find, but that might, years later, completely mess up the application when compiled with the powerful optimizations of an advanced compiler.

It may seem that requiring a cast to use a function is an obvious warning sign that a good D programmer would not ignore. I wouldn't be so sure. Casts aren't that rare even in fine high-level code. Even if you disagree, other cases are provably bad enough to bite anyone. Last summer, this case [appeared in the D forums](https://forum.dlang.org/thread/t7qd45$1lrb$1@digitalmars.com):


```d
string foo(in string s)
{
    return s;
}

void main()
{
    import std.stdio;
    string[] result;
    foreach(c; "hello")
    {
        result ~= foo([c]);
    }
    writeln(result);
}
```
This problem was encountered by Steven Schveighoffer, a long-time D veteran who has himself [lectured about `@safe` and `@system`](https://dlang.org/blog/2016/09/28/how-to-write-trusted-code-in-d/) on [more than one occasion](https://dconf.org/2020/online/#steven). Anything that can burn him can burn any of us.

Normally, this works just as one would think and is fine according to the spec. However, if one enables [another soon-to-be-default language feature](https://dlang.org/spec/function.html#in-params) with the `-preview=in` compiler switch along with DIP1000, the program starts malfunctioning. The old semantics for `in` are the same as `const`, but the new semantics make it `const scope`.

Since the argument of `foo` is `scope`, the compiler assumes that `foo` will copy `[c]` before returning it, or return something else, and therefore it allocates `[c]` on the same stack position for each of the "hello" letters. The result is that the program prints `["o", "o, "o", "o", "o"]`. At least for me, it's already somewhat hard to understand what's happening in this simple example. Hunting down this sort of bug in a complex codebase could be a nightmare.

(With my nightly DMD version somewhere between 2.100 and 2.101 a compile-time error is printed instead. With 2.100.2, the example runs as described above.)

The fundamental problem in both of these examples is the same: `@safe` is not used. Had it been, both of these undefined behaviors would have resulted in compilation errors (the `replaceExceptions` function itself can be `@safe`, but the cast at the usage site cannot). By now it should be clear that `@system` code should be used sparingly.



### When to proceed anyway



Sooner or later, though, the time comes when the guard rail has to be temporarily lowered. Here's an example of a good use case:


```d
/// Undefined behavior: Passing a non-null pointer
/// to a standalone character other than '\0', or
/// to an array without '\0' at or after the
/// pointed character, as utf8Stringz
extern(C) @system pure
bool phobosValidateUTF8(const char* utf8Stringz)
{
    import std.string, std.utf;

    try utf8Stringz.fromStringz.validate();
    catch (UTFException) return false;

    return true;
}
```
This function lets code written in another language validate a UTF-8 string using Phobos. C being C, it tends to use zero-terminated strings, so the function accepts a pointer to one as the argument instead of a D array. This is why the function has to be unsafe. There is no way to safely check that `utf8Stringz` is pointing to either `null` or a valid C string. If the character being pointed to is not `'\0'`, meaning the next character has to be read, the function has no way of knowing whether that character belongs to the memory allocated for the string. It can only trust that the calling code got it right.

Still, this function is a good use of the `@system` attribute. First, it is presumably called primarily from C or C++. Those languages do not get any safety guarantees anyway. Even a `@safe` function is safe only if it gets only those parameters that can be created in `@safe` D code. Passing `cast(const char*) 0xFE0DA1` as an argument to a function is unsafe no matter what the attribute says, and nothing in C or C++ verifies what arguments are passed.

Second, the function clearly documents the cases that would trigger undefined behavior. However, it does not mention that passing an invalid pointer, such as the aforementioned `cast(const char*) 0xFE0DA1`, is UB, because UB is always the default assumption with `@system`-only values unless it can be shown otherwise.

Third, the function is small and easy to review manually. No function should be needlessly big, but it's many times more important than usual to keep `@system` and `@trusted` functions small and simple to review. `@safe` functions can be debugged to pretty good shape by testing, but as we saw earlier, undefined behavior can be immune to testing. Analyzing the code is the only general answer to UB.

There is a reason why the parameter does not have a `scope` attribute. It could have it, no pointers to the string are escaped. However, it would not provide many benefits. Any code calling the function has to be `@system`, `@trusted`, or in a foreign language, meaning they can pass a pointer to the stack in any case. `scope` could potentially improve the performance of D client code in exchange for the increased potential for undefined behavior if this function is erroneously refactored. Such a tradeoff is unwanted in general unless it can be shown that the attribute helps with a performance problem. On the other hand, the attribute would make it clearer for the reader that the string is not supposed to escape. It's a difficult judgment call whether `scope` would be a wise addition here.



### Further improvements



It should be documented why a `@system` function is `@system` when it's not obvious. Often there is a safer alternative--our example function could have taken a D array or the CString struct from the previous post in this series. Why was an alternative not taken? In our case, we could write that the ABI would be different for either of those options, complicating matters on the C side, and the intended client (C code) is unsafe anyway.

`@trusted` functions are like `@system` functions, except they can be called from `@safe` functions, whereas `@system` functions cannot. When something is declared `@trusted`, it means the authors have verified that it's just as safe to use as an actual `@safe` function with any arguments that can be created within safe code. They need to be just as carefully reviewed, if not more so, as `@system` functions.

In these situations, it should be documented (for other developers, not users) how the function was deemed to be safe in all situations. Or, if the function is not fully safe to use and the attribute is just a temporary hack, it should have a big ugly warning about that.

![]({{ '/assets/images/memory-safety-in-a-systems-programming-language-part-3/unsafe-safe.jpg' | relative_url }})

Such greenwashing is of course highly discouraged, but if there's a codebase full of `@system` code that's just too difficult to make `@safe` otherwise, it's better than giving up. Even as we often talk about the dangers of UB and memory corruption, in our actual work our attitudes tend to be much more carefree, meaning such codebases are unfortunately common.

It might be tempting to define a small `@trusted` function inside a bigger `@safe` function to do something unsafe without disabling checks for the whole function:


```d
extern(C) @safe pure
bool phobosValidateUTF8(const char* utf8Stringz)
{
    import std.string, std.utf;

    try (() @trusted => utf8Stringz.fromStringz)()
      .validate();
    catch (UTFException) return false;

    return true;
}
```
Keep in mind though, that the parent function needs to be documented and reviewed like an overt `@trusted` function because the encapsulated `@trusted` function can let the parent function do anything. In addition, since the function is marked `@safe`, it isn't obvious on a first look that it's a function that needs special care. Thus, a visible warning comment is needed if you elect to use `@trusted` like this.

Most importantly, don't trust yourself! Just like any codebase of non-trivial size has bugs, more than a handful of `@system` functions will include latent UB at some point. The remaining hardening features of D, meaning asserts, contracts, invariants, and bounds checking should be used aggressively and kept enabled in production. This is recommended even if the program is fully `@safe`. In addition, a project with a considerable amount of unsafe code should use external tools like LLVM address sanitizer and Valgrind to at least some extent.

Note that the idea in many of these hardening tools, both those in the language and those of the external tools, is to crash as soon as any fault is detected. It decreases the chance of any surprise from undefined behavior doing more serious damage.

This requires that the program is designed to accept a crash at any moment. The program must never hold such amounts of unsaved data that there would be any hesitation in crashing it. If it controls anything important, it must be able to regain control after being restarted by a user or another process, or it must have another backup program. Any program that "can't afford" to run potentially crashing checks is in no business to be trusted with systems programming either.



## Conclusion



That concludes this blog series on DIP1000. There are some topics related to DIP1000 we have left up to the readers to experiment with themselves, such as associative arrays. Still, this should be enough to get them going.

Though we have uncovered some practical tips in addition to language rules, there surely is a lot more that could be said. Tell us your memory safety tips [in the D forums](https://forum.dlang.org/)!

_Thanks to Walter Bright and Dennis Korpel for providing feedback on this article._
