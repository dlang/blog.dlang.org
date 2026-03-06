---
author: StevenSchveighoffer
comments: false
date: 2016-09-28 13:59:14+00:00
layout: post
link: https://dlang.org/blog/2016/09/28/how-to-write-trusted-code-in-d/
slug: how-to-write-trusted-code-in-d
title: How to Write @trusted Code in D
wordpress_id: 283
categories:
- Guest Posts
- The Language
permalink: /how-to-write-trusted-code-in-d/
redirect_from: /2016/09/28/how-to-write-trusted-code-in-d/
---

_Steven Schveighoffer is the creator and maintainer of the [dcollections](https://github.com/schveiguy/dcollections) and [iopipe](https://github.com/schveiguy/iopipe) libraries. He was the primary instigator of D's [inout](https://dlang.org/spec/function.html#inout-functions) feature and the architect of a major rewrite of the language's built-in arrays. He also authored the oft-recommended [introductory article](https://dlang.org/d-array-article.html) on the latter.
_



* * *



![d6](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)In computer programming, there is a concept of [**memory-safe** code](https://en.wikipedia.org/wiki/Memory_safety), which is guaranteed at some level not to cause memory corruption issues. The ultimate holy grail of memory safety is to be able to mechanically verify you will not corrupt memory no matter what. This would provide immunity from attacks via buffer overflows and the like. The D language provides a definition of memory safety that allows quite a bit of useful code, but conservatively forbids things that are sketchy. In practice, the compiler is not omnipotent, and it lacks the context that we humans are so good at seeing (most of the time), so there is often the need to allow otherwise risky behavior. Because the compiler is very rigid on memory safety, we need the equivalent of a cast to say "yes, I know this is normally forbidden, but I'm guaranteeing that it is fine". That tool is called `@trusted`.

Because it's very difficult to explain why `@trusted` code might be incorrect without first discussing memory safety and D's `@safe` mechanism, I'll go over that first.


### What is Memory Safe Code?


The easiest way to explain what is safe, is to examine what results in unsafe code. There are generally 3 main ways to create a safety violation in a statically-typed language:



 	
  1. Write or read from a buffer outside the valid segment of memory that you have access to.

 	
  2. Cast some value to a type that allows you to treat a piece of memory that is not a pointer as a pointer.

 	
  3. Use a pointer that is dangling, or no longer valid.


The first item is quite simple to achieve in D:

```d
auto buf = new int[1]; 
buf[2] = 1;
```
With default bounds checks on, this results in an exception at runtime, even in code that is not checked for safety. But D allows circumventing this by accessing the pointer of the array:

    buf.ptr[2] = 1;
For an example of the second, all that is needed is a cast:

    *cast(int*)(0xdeadbeef) = 5;
And the third is relatively simple as well:

```d
auto buf = new int[1];
auto buf2 = buf;
delete buf;  // sets buf to null
buf2[0] = 5; // but not buf2.
```
Dangling pointers also frequently manifest by pointing at stack data that is no longer in use (or is being used for a different reason). It's very simple to achieve:

```d
int[] foo()
{
    int[4] buf;
    int[] result = buf[];
    return result;
}
```
So simply put, safe code avoids doing things that could potentially result in memory corruption. To that end, we must follow some rules that prohibit such behavior.

Note: dereferencing a `null` pointer in user-space is _not_ considered a memory safety issue in D. Why not? Because this triggers a hardware exception, and generally does not leave the program in an undefined state with corrupted memory. It simply aborts the program. This may seem undesirable to the user or the programmer, but it's perfectly fine in terms of preventing exploits. There are potential memory issues possible with `null` pointers, if one has a `null` pointer to a very large memory space. But for safe D, this requires an unusually large struct to even begin to worry about it. In the eyes of the D language, instrumenting all pointer dereferences to check for `null` is not worth the performance degradation for these rare situations.


### D's @safe rules


D provides the [**`@safe`** attribute](http://dlang.org/spec/function.html#safe-functions) that tags a function to be mechanically checked by the compiler to follow rules that should prevent all possible memory safety problems. Of course, there are cases where developers need to make exceptions in order to get some meaningful work done.

The following rules are geared to prevent issues like the ones discussed above (listed in the spec [here](http://dlang.org/spec/function.html#function-safety)).



 	
  1. Changing a raw pointer value is not allowed. If `@safe` D code has a pointer, it has access _only_ to the value pointed at, no others. This includes indexing a pointer.

 	
  2. Casting pointers to any type other than `void*` is not allowed. Casting from any non-pointer type to a pointer type is not allowed. All other casts are OK (e.g. casting from `float` to `int`) as long as they are valid. Casting a dynamic array to a `void[]` is also allowed.

 	
  3. Unions that have pointer types that overlap other types cannot be accessed. This is similar to rules 1 and 2 above.

 	
  4. Accessing an element in or taking a slice from a dynamic array _must_ be either proven safe by the compiler, or incur a bounds check during runtime. This even happens in release mode, when bounds checks are normally omitted (note: dmd's option **-boundscheck=off** will override this, so use with extreme caution).

 	
  5. In normal D, you can create a dynamic array from a pointer by [slicing](http://dlang.org/spec/expression.html#SliceExpression) the pointer. In `@safe` D, this is not allowed, since the compiler has no idea how much space you actually have available via that pointer.

 	
  6. Taking a pointer to a local variable or function parameter (variables that are stored on the stack) or taking a pointer to a reference parameter are forbidden. An exception is slicing a local static array, including the function `foo` above. This is a [known issue](https://issues.dlang.org/show_bug.cgi?id=8838).

 	
  7. Explicit casting between immutable and mutable types that are or contain references is not allowed. Casting value-types between immutable and mutable can be done implicitly and is perfectly fine.

 	
  8. Explicit casting between thread-local and shared types that are or contain references is not allowed. Again, casting value-types is fine (and can be done implicitly).

 	
  9. The [inline assembler](http://dlang.org/spec/iasm.html) feature of D is not allowed in `@safe` code.

 	
  10. Catching thrown objects that are not derived from `class Exception` is not allowed.

 	
  11. In D, all variables are default initialized. However, this can be changed to uninitialized by using a [void initializer](http://dlang.org/spec/declaration.html#void_init):

    int *s = void;
Such usage is not allowed in `@safe` D. The above pointer would point to random memory and create an obvious dangling pointer.

 	
  12. `__gshared` variables are static variables that are not properly typed as `shared`, but are still in global space. Often these are used when interfacing with C code. Accessing such variables is not allowed in `@safe` D.

 	
  13. Using the `ptr` property of a dynamic array is forbidden (a new rule that will be released in version 2.072 of the compiler).

 	
  14. Writing to `void[]` data by means of slice-assigning from another `void[]` is not allowed (this rule is also new, and will be released in 2.072).

 	
  15. Only `@safe` functions or those inferred to be `@safe` can be called.




### The need for @trusted


The above rules work well to prevent memory corruption, but they prevent a lot of valid, and actually safe, code. For example, consider a function that wants to use the system call [`read`](http://pubs.opengroup.org/onlinepubs/009695399/functions/read.html), which is prototyped like this:

    ssize_t read(int fd, void* ptr, size_t nBytes);
For those unfamiliar with this function, it reads data from the given file descriptor, and puts it into the buffer pointed at by `ptr` and expected to be `nBytes` bytes long. It returns the number of bytes actually read, or a negative value if an error occurs.

Using this function to read data into a stack-allocated buffer might look like this:

```d
ubyte[128] buf;
auto nread = read(fd, buf.ptr, buf.length);
```
How is this done inside a `@safe` function? The main issue with using `read` in `@safe` code is that pointers can only pass a single value, in this case a single `ubyte`. `read` expects to store more bytes of the buffer. In D, we would normally pass the data to be read as a dynamic array. However, `read` is not D code, and uses a common C idiom of passing the buffer and length separately, so it cannot be marked `@safe`. Consider the following call from `@safe` code:

```d
auto nread = read(fd, buf.ptr, 10_000);
```
This call is definitely _not_ safe. What is safe in the above `read` example is only the one call, where the understanding of the `read` function and calling context assures memory outside the buffer will not be written.

To solve this situation, D provides the [`@trusted`  attribute](http://dlang.org/spec/function.html#trusted-functions), which tells the compiler that the code inside the function is assumed to be `@safe`, but will not be mechanically checked. It's on you, the developer, to make sure the code is actually `@safe`.

A function that solves the problem might look like this in D:

```d
auto safeRead(int fd, ubyte[] buf) @trusted
{
    return read(fd, buf.ptr, buf.length);
}
```
Whenever marking an entire function `@trusted`, consider if code could call this function from _any context_ that would compromise memory safety. If so, this function should not be marked `@trusted` **under any circumstances**. Even if the intention is to only call it in safe ways, the compiler will not prevent unsafe usage by others. `safeRead` should be fine to call from any `@safe` context, so it's a great candidate to mark `@trusted`.

A more liberal API for the `safeRead` function might take a `void[]` array as the buffer. However, recall that in `@safe` code, one can cast any dynamic array to a `void[]` array -- including an array of pointers. Reading file data into an array of pointers could result in an array of dangling pointers. This is why `ubyte[]` is used instead.


### @trusted escapes


A `@trusted` escape is a single expression that allows `@system` (the unsafe default in D) calls such as `read` without exposing the potentially unsafe call to any other part of the program. Instead of writing the `safeRead` function, the same feat can be accomplished inline within a `@safe` function:

```d
auto nread = ( () @trusted => read(fd, buf.ptr, buf.length) )();
```
Let's take a closer look at this escape to see what is actually happening. D allows declaring a [lambda function](https://en.wikipedia.org/wiki/Anonymous_function) that evaluates and returns a single expression, with the [`() => expr` syntax](http://dlang.org/spec/expression.html#Lambda). In order to call the lambda function, parentheses are appended to the lambda. However, operator precedence will apply those parentheses to the expression and not the lambda, so the entire lambda must be wrapped in parentheses to clarify the call. And finally, the lambda can be tagged `@trusted` as shown, so the call is now usable from the `@safe` context that contains it.

In addition to simple lambdas, whole [nested functions](http://dlang.org/spec/function.html#nested) or multi-statement lambdas can be used. However, remember that adding a trusted nested function or saving a lambda to a variable exposes the rest of the function to potential safety concerns! Take care not to expose the escape too much because this risks having to manually verify code that should just be mechanically checked.


### Rules of Thumb for @trusted


The previous examples show that tagging something as `@trusted` has huge implications. If you are disabling memory safety checks, but allowing any `@safe` code to call it, then you must be sure that it cannot result in memory corruption. These rules should give guidance on where to put `@trusted` marks and avoid getting into trouble:


#### Keep @trusted code small


`@trusted` code is never mechanically checked for safety, so every line must be reviewed for correctness. For this reason, it's always advisable to keep the code that is `@trusted` as small as possible.


#### Apply @trusted to entire functions when the unsafe calls are leaky


Code that modifies or uses data that `@safe` code also uses creates the potential for unsafe calls to leak into the mechanically checked portion of a `@safe` function. This means that portion of the code must be manually reviewed for safety issues. It's better to mark the whole thing `@trusted`, as that's more in line with the truth. This is not a hard and fast rule; for example, the `read` call from the earlier example is perfectly safe, even though it will affect data that is used later by the function in `@safe` mode.

A pointer allocated with C's `malloc` in the beginning of the function, and `free`'d later, could have been copied somewhere in between. In this case, the dangling pointer may violate `@safe`, even in the mechanically checked part. Instead, try wrapping the entire portion that uses the pointer as `@trusted`, or even the entire function. Alternatively, use [scope guards](http://dlang.org/spec/statement.html#scope-guard-statement) to guarantee the lifetime of the data until the end of the function.


#### Never use @trusted on template functions that accept arbitrary types


D is smart enough to [infer](http://dlang.org/spec/function.html#function-attribute-inference) `@safe` for template functions that follow the rules. This includes member functions of templated types. Just let the compiler do its job here. To ensure the function is actually `@safe` in the right contexts, create an `@safe unittest`  to call it. Marking the function `@trusted` allows any operator overloads or members that might violate memory safety to be ignored by the safety checker! Some tricky ones to remember are [postblit](http://dlang.org/spec/struct.html#struct-postblit) and [opCast](http://dlang.org/spec/operatoroverloading.html#cast).

It's still OK to use `@trusted` escapes here, but be very careful. Consider especially possible types that contain pointers when thinking about how such a function could be abused. A common mistake is to mark a range function or range usage `@trusted`. Remember that most ranges are templates, and can be easily inferred as `@system` when the type being iterated has a `@system` postblit or constructor/destructor, or is generated from a user-provided lambda.


#### Use @safe to find the parts you need to mark as @trusted


Sometimes, a template intended to be `@safe` may not be inferred `@safe`, and it's not clear why. In this case, try temporarily marking the template function `@safe` to see where the compiler complains. That's where `@trusted` escapes should be inserted if appropriate.

In some cases, a template is used pervasively, and tagging it as `@safe` may make too many parts break. Make a copy of the template under a different name that you mark `@safe`, and change the calls that are to be checked so that they call the alternative template instead.


#### Consider how the function may be edited in the future


When writing a trusted function, always think about how it could be called with the given API, and ensure that it should be `@safe`. A good example from above is making sure `safeRead` cannot accept an array of pointers. However, another possibility for unsafe code to creep in is when someone edits a part of the function later, invalidating the previous verification, and the whole function needs to be rechecked. Insert comments to explain the danger of changing something that would then violate safety. Remember, pull request diffs don't always show the entire context, including that a long function being edited is `@trusted`!


#### Use types to encapsulate @trusted operations with defined lifetimes


Sometimes, a resource is only dangerous to create and/or destroy, but not to use during its lifetime. The dangerous operations can be encapsulated into a type's constructor and destructor, marked `@trusted`, which allows `@safe` code to use the resource in between those calls. This takes a lot of planning and care. At no time can you allow `@safe` code to ferret out the actual resource so that it can keep a copy past the lifetime of the managing struct! It is essential to make sure the resource is alive as long as `@safe` code has a reference to it.

For example, a reference-counted type can be perfectly safe, as long as a raw pointer to the payload data is never available. D's [`std.typecons.RefCounted`](https://dlang.org/phobos/std_typecons.html#.RefCounted) cannot be marked `@safe`, since it uses [`alias this`](https://dlang.org/spec/class.html#alias-this) to devolve to the protected allocated struct in order to function, and any calls into this struct are unaware of the reference counting. One copy of that payload pointer, and then when the struct is `free`'d, a dangling pointer is present.


### This can't be @safe!


Sometimes, the compiler allows a function to be `@safe`, or is inferred `@safe`, and it's obvious that shouldn't be allowed. This is caused by one of two things: either a function that is called by the `@safe` function (or some deeper function) is marked `@trusted` but allows unsafe calls, or there is a bug or hole in the `@safe` system. Most of the time, it is the former. `@trusted` is a very tricky attribute to get correct, as is shown by most of this post. Frequently, developers will mark a function `@trusted` only thinking of some uses of their function, not realizing the dangers it allows. Even core D developers make this mistake! There can be template functions that are inferred safe because of this, and sometimes it's difficult to even find the culprit. Even after the root cause is discovered, it's often difficult to remove the `@trusted` tag as it will break many users of the function. However, it's better to break code that is expecting a promise of memory safety than subject it to possible memory exploits. The sooner you can deprecate and remove the tag, the better. Then insert trusted escapes for cases that can be proven safe.

If it does happen to be a hole in the system, please [report the issue](https://issues.dlang.org/enter_bug.cgi), or ask questions on the [D forums](http://forum.dlang.org/group/learn). The D community is generally happy to help, and memory safety is a particular focus for Walter Bright, the creator of the language.
