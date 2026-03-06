---
author: AteEskola
comments: false
date: 2022-10-08 15:19:06+00:00
layout: post
link: https://dlang.org/blog/2022/10/08/dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/
slug: dip1000-memory-safety-in-a-modern-systems-programming-language-part-2
title: Memory Safety in a Modern Systems Programming Language Part 2
wordpress_id: 3116
categories:
- Code
- DIPs
- Guest Posts
- The Language
- Tutorials
permalink: /dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/
redirect_from: /2022/10/08/dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/
---

# DIP1000: Memory Safety in a Modern System Programming Language Pt. 2



![]({{ '/assets/images/dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/brain02.png' | relative_url }})

[The previous entry in this series](https://dlang.org/blog/2022/06/21/dip1000-memory-safety-in-a-modern-system-programming-language-pt-1/) shows how to use the new DIP1000 rules to have slices and pointers refer to the stack, all while being memory safe. But D can refer to the stack in other ways, too, and that's the topic of this article.



## Object-oriented instances are the easiest case



In Part 1, I said that if you understand how DIP1000 works with pointers, then you understand how it works with classes. An example is worth more than mere words:


```d
@safe Object ifNull(return scope Object a, return scope Object b)
{
    return a? a: b;
}
```
The `return scope` in the above example works exactly as it does in the following:


```d
@safe int* ifNull(return scope int* a, return scope int* b)
{
    return a? a: b;
}
```
The principle is: if the `scope` or `return scope` storage class is applied to an object in a parameter list, the address of the object instance is protected just as if the parameter were a pointer to the instance. From the perspective of machine code, it **is** a pointer to the instance.

From the point of view of regular functions, that's all there is to it. What about member functions of a class or an interface? This is how it's done:


```d
interface Talkative
{
    @safe const(char)[] saySomething() scope;
}

class Duck : Talkative
{
    char[8] favoriteWord;
    @safe const(char)[] saySomething() scope
    {
        import std.random : dice;

        // This wouldn't work
        // return favoriteWord[];

        // This does
        return favoriteWord[].dup;

        // Also returning something totally
        // different works. This
        // returns the first entry 40% of the time,
        // The second entry 40% of the time, and
        // the third entry the rest of the time.
        return
        [
            "quack!",
            "Quack!!",
            "QUAAACK!!!"
        ][dice(2,2,1)];
    }
}
```
`scope` positioned either before or after the member function name marks the `this` reference as `scope`, preventing it from leaking out of the function. Because the address of the instance is protected, nothing that refers directly to the address of the fields is allowed to escape either. That's why `return favoriteWord[]` is disallowed; it's a static array stored inside the class instance, so the returned slice would refer directly to it. `favoriteWord[].dup` on the other hand returns a copy of the data that isn't located in the class instance, which is why it's okay.

Alternatively one could replace the `scope` attributes of both `Talkative.saySomething` and `Duck.saySomething` with `return scope`, allowing the return of `favoriteWord` without duplication.



### DIP1000 and Liskov Substitution Principle



[The Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle) states, in simplified terms, that an inherited function can give the caller more guarantees than its parent function, but never fewer. DIP1000-related attributes fall in that category. The rule works like this:





  * if a parameter (including the implicit `this` reference) in the parent functions has no DIP1000 attributes, the child function may designate it `scope` or `return scope`


  * if a parameter is designated `scope` in the parent, it must be designated `scope` in the child


  * if a parameter is `return scope` in the parent, it must be either `scope` or `return scope` in the child



If there is no attribute, the caller can not assume anything; the function might store the address of the argument somewhere. If `return scope` is present, the caller can assume the address of the argument is not stored other than in the return value. With `scope`, the guarantee is that the address is not stored anywhere, which is an even stronger guarantee. Example:


```d
class C1
{   double*[] incomeLog;
    @safe double* imposeTax(double* pIncome)
    {
        incomeLog ~= pIncome;
        return new double(*pIncome * .15);
    }
}

class C2 : C1
{
    // Okay from language perspective (but maybe not fair
    // for the taxpayer)
    override @safe double* imposeTax
        (return scope double* pIncome)
    {
        return pIncome;
    }
}

class C3 : C2
{
    // Also okay.
    override @safe double* imposeTax
        (scope double* pIncome)
    {
        return new double(*pIncome * .18);
    }
}

class C4: C3
{
    // Not okay. The pIncome parameter of C3.imposeTax
    // is scope, and this tries to relax the restriction.
    override @safe double* imposeTax
        (double* pIncome)
    {
        incomeLog ~= pIncome;
        return new double(*pIncome * .16);
    }
}
```
## The special pointer, `ref`



We still have not uncovered how to use `struct`s and `union`s with DIP1000. Well, obviously we've uncovered pointers and arrays. When referring to a `struct` or a `union`, they work the same as they do when referring to any other type. But pointers and arrays are not the canonical way to use structs in D. They are most often passed around by value, or by reference when [bound to `ref` parameters](https://dlang.org/spec/function.html#ref-params). Now is a good time to explain how `ref` works with DIP1000.

They don't work like just any pointer. Once you understand `ref`, you can use DIP1000 in many ways you otherwise could not.



### A simple `ref int` parameter



The simplest possible way to use `ref` is probably this:


```d
@safe void fun(ref int arg) {
    arg = 5;
}
```
What does this mean? `ref` is internally a pointer--think `int* pArg`--but is used like a value in the source code. `arg = 5` works internally like `*pArg = 5`. Also, the client calls the function as if the argument were passed by value:


```d
auto anArray = [1,2];
fun(anArray[1]); // or, via UFCS: anArray[1].fun;
// anArray is now [1, 5]
```
instead of `fun(&anArray[1])`. [Unlike C++ references](https://en.wikipedia.org/wiki/Reference_(C%2B%2B)), D `ref`erences can be `null`, but the application will instantly terminate with a segmentation fault if a null `ref` is used for something other than reading the address with the `&` operator. So this:

```d
int* ptr = null;
fun(*ptr);
```
…compiles, but crashes at runtime because the assignment inside `fun` lands at the null address.

The address of a `ref` variable is always guarded against escape. In this sense `@safe void fun(ref int arg){arg = 5;}` is like `@safe void fun(scope int* pArg){*pArg = 5;}`. For example, `@safe int* fun(ref int arg){return &arg;}` will not compile, just like `@safe int* fun(scope int* pArg){return pArg;}` will not.

There is a `return ref` storage class, however, that allows returning the address of the parameter but no other form of escape, just like `return scope`. This means that `@safe int* fun(return ref int arg){return &arg;}` works.



### `ref`erence to a reference



`ref`erence to an `int` or similar type already allows much nicer syntax than one can get with pointers. But the real power of `ref` shows when it refers to a type that is a reference itself--a pointer or a class, for instance. `scope` or `return scope` can be applied to a reference that is referenced to by `ref`. For example:


```d
@safe float[] mergeSort(ref return scope float[] arr)
{
    import std.algorithm: merge;
    import std.array : Appender;

    if(arr.length < 2) return arr;

    auto firstHalf = arr[0 .. $/2];
    auto secondHalf = arr[$/2 .. $];

    Appender!(float[]) output;
    output.reserve(arr.length);

    foreach
    (
        el;
        firstHalf.mergeSort
        .merge!floatLess(secondHalf.mergeSort)
    )   output ~= el;

    arr = output[];
    return arr;
}

@safe bool floatLess(float a, float b)
{
    import std.math: isNaN;

    return a.isNaN? false:
          b.isNaN? true:
          a<b;
}
```
`mergeSort` here guarantees it won't leak the address of the `float`s in `arr` except in the return value. This is the same guarantee that would be had from a `return scope float[] arr` parameter. But at the same time, because `arr` is a `ref` parameter, `mergeSort` can mutate the array passed to it. Then the client can write:

```d
float[] values = [5, 1.5, 0, 19, 1.5, 1];
values.mergeSort;
```
With a non-`ref` argument, the client would have to write `values = values.sort` instead (not using `ref` would be a perfectly reasonable API in this case, because we do not always want to mutate the original array). This is something that cannot be accomplished with pointers, because `return scope float[]* arr` would protect the address of the array's metadata (the `length` and `ptr` fields of the array), not the address of it's contents.

It is also possible to have a returnable `ref` argument to a `scope` reference. Since this example has a unit test, remember to use the `-unittest` compile flag to include it in the compiled binary.


```d
@safe ref Exception nullify(return ref scope Exception obj)
{
    obj = null;
    return obj;
}

@safe unittest
{
    scope obj = new Exception("Error!");
    assert(obj.msg == "Error!");
    obj.nullify;
    assert(obj is null);
    // Since nullify returns by ref, we can assign
    // to it's return value.
    obj.nullify = new Exception("Fail!");
    assert(obj.msg == "Fail!");
}
```
Here we return the address of the argument passed to `nullify`, but still guard both the address of the object pointer and the address of the class instance against being leaked by other channels.

`return` is a free keyword that does not mandate `ref` or `scope` to follow it. What does `void* fun(ref scope return int*)` mean then? [The spec states](https://dlang.org/spec/function.html#ref-return-scope-parameters) that `return` without a trailing `scope` is always treated as `ref return`. This example thus is equivalent to `void* fun(return ref scope int*)`. However, this only applies if there is `ref`erence to bind to. Writing `void* fun(scope return int*)` means `void* fun(return scope int*)`. It's even possible to write `void* fun(return int*)` with the latter meaning, but I leave it up to you to decide whether this qualifies as conciseness or obfuscation.



## Member functions and `ref`



`ref` and `return ref` often require careful consideration to keep track of which address is protected and what can be returned. It takes some experience to get confortable with them. But once you do, understanding how `struct`s and `union`s work with DIP1000 is pretty straightforward.

The major difference to classes is that where the `this` reference is just a regular class reference in class member functions, `this` in a struct or union member function is `ref StructOrUnionName`.


```d
union Uni
{
    int asInt;
    char[4] asCharArr;

    // Return value contains a reference to
    // this union, won't escape references
    // to it via any other channel
    @safe char[] latterHalf() return
    {
        return asCharArr[2 .. $];
    }

    // This argument is implicitly ref, so the
    // following means the return value does
    // not refer to this union, and also that
    // we don't leak it in any other way.
    @safe char[] latterHalfCopy()
    {
        return latterHalf.dup;
    }
}
```
Note that `return ref` should not be used with the `this` argument. `char[] latterHalf() return ref` fails to parse. The language already has to understand what `ref char[] latterHalf() return` means: the return value is a `ref`erence. The "ref" in `return ref` would be redundant anyway.

Note that we did not use the `scope` keyword here. `scope` would be meaningless with this union, because it does not contain references to anything. Just like it is meaningless to have a `scope ref int`, or a `scope int` function argument. `scope` makes sense only for types that refer to memory elsewhere.

`scope` in a `struct` or `union` means the same thing as it means in a static array. It means that the memory its members refer to cannot be escaped. Example:


```d
struct CString
{
    // We need to put the pointer in an anonymous
    // union with a dummy member, otherwise @safe user
    // code could assign ptr to point to a character
    // not in a C string.
    union
    {
        // Empty string literals get optimised to null pointers by D
        // compiler, we have to do this for the .init value to really point to
        // a '\0'.
        immutable(char)* ptr = &nullChar;
        size_t dummy;
    }

    // In constructors, the "return value" is the
    // constructed data object. Thus, the return scope
    // here makes sure this struct won't live longer
    // than the memory in arr.
    @trusted this(return scope string arr)
    {
        // Note: Normal assert would not do! They may be
        // removed from release builds, but this assert
        // is necessary for memory safety so we need
        // to use assert(0) instead which never gets
        // removed.
        if(arr[$-1] != '\0') assert(0, "not a C string!");
        ptr = arr.ptr;
    }

    // The return value refers to the same memory as the
    // members in this struct, but we don't leak references
    // to it via any other way, so return scope.
    @trusted ref immutable(char) front() return scope
    {
        return *ptr;
    }

    // No references to the pointed-to array passed
    // anywhere.
    @trusted void popFront() scope
    {
        // Otherwise the user could pop past the
        // end of the string and then read it!
        if(empty) assert(0, "out of bounds!");
        ptr++;
    }

    // Same.
    @safe bool empty() scope
    {
        return front == '\0';
    }
}

immutable nullChar = '\0';

@safe unittest
{
    import std.array : staticArray;

    auto localStr = "hello world!\0".staticArray;
    auto localCStr = localStr.CString;
    assert(localCStr.front == 'h');

    static immutable(char)* staticPtr;

    // Error, escaping reference to local.
    // staticPtr = &localCStr.front();

    // Fine.
    staticPtr = &CString("global\0").front();

    localCStr.popFront;
    assert(localCStr.front == 'e');
    assert(!localCStr.empty);
}
```
Part One said that `@trusted` is a terrible footgun with DIP1000. This example demonstrates why. Imagine how easy it'd be to use a regular assert or forget about them totally, or overlook the need to use the anonymous union. I _think_ this struct is safe to use, but it's entirely possible I overlooked something.



## Finally



We almost know all there is to know about using structs, unions, and classes with DIP1000. We have two final things to learn today.

But before that, a short digression regarding the `scope` keyword. It is not used for just annotating parameters and local variables as illustrated. It is also used for [scope classes](https://dlang.org/spec/class.html#auto) and [scope guard statements](https://dlang.org/spec/statement.html#scope-guard-statement). This guide won't be discussing those, because the former feature is deprecated, and the latter is not related to DIP1000 or control of variable lifetimes. The point of mentioning them is to dispel a potential misconception that `scope` always means limiting the lifetime of something. Learning about scope guard statements is still a good idea, as it's a useful feature.

Back to the topic. The first thing is not really specific to structs or classes. We discussed what `return`, `return ref`, and `return scope` usually mean, but there's an alternative meaning to them. Consider:


```d
@safe void getFirstSpace
(
    ref scope string result,
    return scope string where
)
{
    //...
}
```
The usual meaning of the `return` attribute makes no sense here, as the function has a `void` return type. A special rule applies in this case: if the return type is `void`, and the first argument is `ref` or `out`, any subsequent `return` [`ref`/`scope`] is assumed to be escaped by assigning to the first argument. With struct member functions, they are assumed to be assigned to the struct itself.


```d
@safe unittest
{
    static string output;
    immutable(char)[8] input = "on stack";
    //Trying to assign stack contents to a static
    //variable. Won't compile.
    getFirstSpace(output, input);
}
```
Since `out` came up, it should be said it would be a better choice for `result` here than `ref`. `out` works like `ref`, with the one difference that the referenced data is automatically default-initialized at the beginning of the function, meaning any data to which the `out` parameter refers is guaranteed to not affect the function.

The second thing to learn is that `scope` is used by the compiler to optimize class allocations inside function bodies. If a `new` class is used to initialize a `scope` variable, the compiler can put it on the stack. Example:


```d
class C{int a, b, c;}
@safe @nogc unittest
{
    // Since this unittest is @nogc, this wouldn't
    // compile without the scope optimization.
    scope C c = new C();
}
```
This feature requires using the `scope` keyword explicitly. Inference of `scope` does not work, because initializing a class this way does not normally (meaning, without the `@nogc` attribute) mandate limiting the lifetime of `c`. The feature currently works only with classes, but there is no reason it couldn't work with `new`ed struct pointers and array literals too.



### Until next time



This is pretty much all that there is to manual DIP1000 usage. But this blog series shall not be over yet! DIP1000 is not intended to always be used explicitly--it works with attribute inference. That's what the next post will cover.

It will also cover some considerations when daring to use `@trusted` and `@system` code. The need for dangerous systems programming exists and is part of the D language domain. But even systems programming is a responsible affair when people do what they can to minimize risks. We will see that even there it's possible to do a lot.

_Thanks to Walter Bright and Dennis Korpel for reviewing this article_
