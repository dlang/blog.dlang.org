---
author: AteEskola
comments: false
date: 2022-06-21 15:00:14+00:00
layout: post
link: https://dlang.org/blog/2022/06/21/dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/
slug: dip1000-memory-safety-in-a-modern-system-programming-language-pt-1
title: Memory Safety in a Modern Systems Programming Language Part 1
wordpress_id: 3095
categories:
- Code
- DIPs
- Guest Posts
- The Language
- Tutorials
permalink: /dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/
redirect_from: /2022/06/21/dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/
---

## Memory safety needs no checks



![]({{ '/assets/images/dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/brain02.png' | relative_url }})

D is both a garbage-collected programming language and an efficient raw memory access language. Modern high-level languages like D are memory safe, preventing users from accidently reading or writing to unused memory or breaking the type system of the language.

As a systems programming language, not all of D can give such guarantees, but it does have a memory-safe subset that uses the garbage collector to take care of memory management much like Java, C#, or Go. A D codebase, even in a systems programming project, should aim to remain within that memory-safe subset where practical. D provides the `@safe` function attribute to verify that a function uses only memory-safe features of the language. For instance, try this.


```d
@safe string getBeginning(immutable(char)* cString)
{
    return cString[0..3];
}
```
The compiler will refuse to compile this code. There's no way to know what will result from the three-character slice of `cString`, which could be referring to an empty string (i.e., `cString[0]` is `\0`), a string with a length of `1`, or even one or two characters without the terminating NUL. The result in those cases would be a memory violation.



### `@safe` does not mean slow



Note that I said above that even a low-level systems programming project should use `@safe` where practical. How is that possible, given that such projects sometimes cannot use the garbage collector, a major tool used in D to guarantee memory safety?

Indeed, such projects must resort to memory-unsafe constructs every now and then. Even higher-level projects often have reasons to do so, as they want to create interfaces to C or C++ libraries, or avoid the garbage collector when indicated by runtime performance. But still, surprisingly large parts of code can be made `@safe` without using the garbage collector at all.

D can do this because the memory safe subset does not prevent raw memory access per se.


```d
@safe void add(int* a, int* b, int* sum)
{
    *sum = *a + *b;
}
```
This compiles and is fully memory safe, despite dereferencing those pointers in the same completely unchecked way they are dereferenced in C. This is memory safe because `@safe` D does not allow creating an `int*` that points to unallocated memory areas, or to a `float**`, for instance. `int*` can point to the null address, but this is generally not a memory safety problem because the null address is protected by the operating system. Any attempt to dereference it would crash the program before any memory corruption can happen. The garbage collector isn't involved, because D's GC can only run if more memory is requestend from it, or if the collection is explicitly called.

D slices are similar. When indexed at runtime, they will check at runtime that the index is less than their length and that's it. They will do no checking whatsoever on whether they are referring to a legal memory area. Memory safety is achieved by preventing creation of slices that could refer to illegal memory in the first place, as demonstrated in the first example of this article. And again, there's no GC involved.

This enables many patterns that are memory-safe, efficient, and independent of the garbage collector.


```d
struct Struct
{
    int[] slice;
    int* pointer;
    int[10] staticArray;
}

@safe @nogc Struct examples(Struct arg)
{
    arg.slice[5] = *arg.pointer;
    arg.staticArray[0..5] = arg.slice[5..10];
    arg.pointer = &arg.slice[8];
    return arg;
}
```
As demonstrated, D liberally lets one do unchecked memory handling in `@safe` code. The memory referred to by `arg.slice` and `arg.pointer` may be on the garbage collected heap, or it may be in the static program memory. There is no reason the language needs to care. The program will probably need to either call the garbage collector or do some unsafe memory management to allocate memory for the pointer and the slice, but handling already allocated memory does not need to do either. If this function needed the garbage collector, it would fail to compile because of the `@nogc` attribute.



### However…



There's a historical design flaw here in that the memory may also be on the stack. Consider what happens if we change our function a bit.


```python
@safe @nogc Struct examples(Struct arg)
{
    arg.pointer = &arg.staticArray[8];
    arg.slice = arg.staticArray[0..8];
    return arg;
}
```
`Struct arg` is a value type. Its contents are copied to the stack when `examples` is called and can be ovewritten after the function returns. `staticArray` is also a value type. It's copied along with the rest of the struct just as if there were ten integers in the struct instead. When we return `arg`, the contents of `staticArray` are copied to the return value, but `ptr` and `slice` continue to point to `arg`, not the returned copy!

But we have a fix. It allows one to write code just as performant in `@safe` functions as before, including references to the stack. It even enables a few formerly `@system` (the opposite of `@safe`) tricks to be written in a safe way. That fix is DIP1000. It's the reason why this example already causes a deprecation warning by default if it's compiled with the latest nightly dmd.



## Born first, dead last



DIP1000 is a set of enhancements to the language rules regarding pointers, slices, and other references. The name stands for [D Improvement Proposal](https://github.com/dlang/DIPs) number 1000, [as that document is what](https://github.com/dlang/DIPs/blob/master/DIPs/other/DIP1000.md) the new rules were initially based on. One can enable the new rules with the preview compiler switch, `-preview=dip1000`. Existing code may need some changes to work with the new rules, which is why the switch is not enabled by default. It's going to be the default in the future, so it's best to enable it where possible and work to make code compatible with it where not.

The basic idea is to let people limit the lifetime of a reference (an array or pointer, for example). A pointer to the stack is not dangerous if it does not exist longer than the stack variable it is pointing to. Regular references continue to exist, but they can refer only to data with an unlimited lifetime--that is, garbage collected memory, or `static` or global variables.



### Let's get started



The simplest way to construct limited lifetime references is to assign to it something with a limited lifetime.


```d
@safe int* test(int arg1, int arg2)
{
    int* notScope = new int(5);
    int* thisIsScope = &arg1;
    int* alsoScope; // Not initially scope...
    alsoScope = thisIsScope; // ...but this makes it so.

    // Error! The variable declared earlier is
    // considered to have a longer lifetime,
    // so disallowed.
    thisIsScope = alsoScope;

    return notScope; // ok
    return thisIsScope; // error
    return alsoScope; // error
}
```
When testing these examples, remember to use the compiler switch `-preview=dip1000` and to mark the function `@safe`. The checks are not done for non-`@safe` functions.

Alternatively, [the `scope` keyword](https://dlang.org/spec/attribute.html#scope) can be explicitly used to limit the lifetime of a reference.


```d
@safe int[] test()
{
    int[] normalRef;
    scope int[] limitedRef;

    if(true)
    {
        int[5] stackData = [-1, -2, -3, -4, -5];

        // Lifetime of stackData ends
        // before limitedRef, so this is
        // disallowed.
        limitedRef = stackData[];

        //This is how you do it
        scope int[] evenMoreLimited
            = stackData[];
    }

    return normalRef; // Okay.
    return limitedRef; // Forbidden.
}
```
If we can't return limited lifetime references, how they are used at all? Easy. Remember, only the address of the data is protected, not the data itself. It means that we have many ways to pass scoped data out of the function.


```d
@safe int[] fun()
{
    scope int[] dontReturnMe = [1,2,3];

    int[] result = new int[](dontReturnMe.length);
    // This copies the data, instead of having
    // result refer to protected memory.
    result[] = dontReturnMe[];
    return result;

    // Shorthand way of doing the same as above
    return dontReturnMe.dup;

    // Also you are not always interested
    // in the contents as a whole; you
    // might want to calculate something else
    // from them
    return
    [
        dontReturnMe[0] * dontReturnMe[1],
        cast(int) dontReturnMe.length
    ];
}
```
### Getting interprocedural



With the tricks discussed so far, DIP1000 would be restricted to language primitives when handling limited lifetime references, but the `scope` storage class can be applied to function parameters, too. Because this guarantees the memory won't be used after the function exits, local data references can be used as arguments to `scope` parameters.


```d
@safe double average(scope int[] data)
{
    double result = 0;
    foreach(el; data) result += el;
    return result / data.length;
}

@safe double use()
{
    int[10] data = [1,2,3,4,5,6,7,8,9,10];
    return data[].average; // works!
}
```
Initially, it's probably best to keep [attribute auto inference](https://dlang.org/spec/function.html#function-attribute-inference) off. Auto inference in general is a good tool, but it silently adds `scope` attributes to all parameters it can, meaning it's easy to lose track of what's happening. That makes the learning process a lot harder. Avoid this by always explicitly specifying the return type (or lack thereof with `void` or `noreturn`): `@safe const(char[]) fun(int* val)` as opposed to `@safe auto fun(int* val)` or `@safe const fun(int* val)`. The function also must not be a template or inside a template. We'll dig deeper on `scope` auto inference in a future post.

`scope` allows handling pointers and arrays that point to the stack, but forbids returning them. What if that's the goal? Enter the `return scope` attribute:


```d
//Being character arrays, strings also work with DIP1000.
@safe string latterHalf(return scope string arg)
{
    return arg[$/2 .. $];
}

@safe string test()
{
    // allocated in static program memory
    auto hello1 = "Hello world!";
    // allocated on the stack, copied from hello1
    immutable(char)[12] hello2 = hello1;

    auto result1 = hello1.latterHalf; // ok
    return result1; // ok

    auto result2 = hello2[].latterHalf; // ok
    // Nice try! result2 is scope and can't
    // be returned.
    return result2;
}
```
`return scope` parameters work by checking if any of the arguments passed to them are `scope`. If so, the return value is treated as a `scope` value that may not outlive any of the `return scope` arguments. If none are `scope`, the return value is treated as a global reference that can be copied freely. Like `scope`, `return scope` is conservative. Even if one does not actually return the address protected by `return scope`, the compiler will still perform the call site lifetime checks just as if one did.



### `scope` is shallow




```d
@safe void test()
{
    scope a = "first";
    scope b = "second";
    string[] arr = [a, b];
}
```
In `test`, initializing `arr` does not compile. This may be surprising given that the language automatically adds `scope` to a variable on initialization if needed.

However, consider what the `scope` on `scope string[] arr` would protect. There are two things it could potentially protect: the addresses of the strings in the array, or the addresses of the characters in the strings. For this assignment to be safe, `scope` would have to protect the characters in the strings, but it only protects the top-level reference, i.e., the strings in the array. Thus, the example does not work. Now change `arr` so that it's a static array:


```d
@safe void test()
{
    scope a = "first";
    scope b = "second";
    string[2] arr = [a, b];
}
```
This works because static arrays are not references. Memory for all of their elements is allocated in place on the stack (i.e., they _contain_ their elements), as opposed to dynamic arrays which contain a reference to elements stored elsewhere. When a static array is `scope`, its elements are treated as `scope`. And since the example would not compile were `arr` not `scope`, it follows that `scope` is inferred.



## Some practical tips



Let's face it, the DIP1000 rules take time to understand, and many would rather spend that time coding something useful. The first and most important tip is: avoid non-`@safe` code like the plague if doable. Of course, this advice is not new, but it appears even more important with DIP1000. In a nutshell, the language does not check the validity of `scope` and `return scope` in a non-`@safe` function, _but_ when calling those functions the compiler assumes that the attributes are respected.

This makes `scope` and `return scope` terrible footguns in unsafe code. But by resisting the temptation to mark code `@trusted` to avoid thinking, a D coder can hardly do damage. Misusing DIP1000 in `@safe` code can cause needless compilation errors, but it won't corrupt memory and is unlikely to cause other bugs either.

A second important point worth mentioning is that there is no need for `scope` and `return scope` for function attributes if they receive only static or GC-allocated data. Many langauges do not let coders refer to the stack at all; just because D programmers can do so does not mean they must. This way, they don't have to spend any more time solving compiler errors than they did before DIP1000. And if a desire to work with the stack arises after all, the authors can then return to annotate the functions. Most likely they will accomplish this without breaking the interface.



## What's next?



This concludes today's blog post. This is enough to know how to use arrays and pointers with DIP1000. In principle, it also enables readers to use DIP1000 with classes and interfaces. The only thing to learn is that a class reference, including the `this` pointer in member functions, works with DIP1000 just like a pointer would. Still, it's hard to grasp what that means from one sentence, so later posts shall illustrate the subject.

In any case, there is more to know. DIP1000 has some features for `ref` function parameters, structs, and unions that we didn't cover here. We'll also dig deeper on how DIP1000 plays with non-`@safe` functions and attribute auto inference. Currently, the plan is to do two more posts for this series.

Do let us know in the comment section or the D forums if you have any useful DIP1000 tips that were not covered!

_Thanks to Walter Bright for reviewing this article._
