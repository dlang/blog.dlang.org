---
author: DBlogAdmin
comments: false
date: 2020-04-28 14:22:53+00:00
layout: post
link: https://dlang.org/blog/2020/04/28/interfacing-d-with-c-arrays-and-functions-arrays-part-two/
slug: interfacing-d-with-c-arrays-and-functions-arrays-part-two
title: 'Interfacing D with C: Arrays and Functions (Arrays Part 2)'
wordpress_id: 2385
categories:
- Code
- D and C
- Tutorials
permalink: /interfacing-d-with-c-arrays-and-functions-arrays-part-two/
redirect_from: /2020/04/28/interfacing-d-with-c-arrays-and-functions-arrays-part-two/
---

![Digital Mars D logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)

This post is part of [an ongoing series](https://dlang.org/blog/the-d-and-c-series/) on working with both D and C in the same project. [The previous post explored the differences](https://dlang.org/blog/2018/10/17/interfacing-d-with-c-arrays-part-1/) in array declaration and initialization. This post takes the next step: declaring and calling C functions that take arrays as parameters.



## Arrays and C function declarations



Using C libraries in D is extremely easy. Most of the time, things work exactly as one would expect, but as we saw in the previous article there can be subtle differences. When working with C functions that expect arrays, it's necessary to fully understand these differences.

The most straightforward and common way of declaring a C function that accepts an array as a parameter is to to use a pointer in the parameter list. For example, this hypothetical C function:


```c
void f0(int *arr);
```
In C, any array of `int` can be passed to this function no matter how it was declared. Given `int a[]`, `int b[3]`, or `int *c`, the function calls `f0(a)`, `f0(b)`, and `f0(c)` are all the same: a pointer to the first element of each array is passed to the function. Or using the lingo of C programmers, arrays _decay_ to pointers

Typically, in a function like `f0`, the implementer will expect the array to have been terminated with a marker appropriate to the context. For example, strings in C are arrays of `char` that are terminated with the `\0` character (we'll look at D strings vs. C strings in a future post). This is necessary because, without that character, the implementation of `f0` has no way to know which element in the array is the last one. Sometimes, a function is simply documented to expect a certain length, either in comments or in the function name, e.g., a `vector3f_add(float *vec)` will expect that `vec` points to exactly 3 elements. Another option is to require the length of the array as a separate argument:


```d
void f1(int *arr, size_t len);
```
None of these approaches is foolproof. If `f0` receives an array with no end marker or which is shorter than documented, or if `f1` receives an array with an actual length shorter than `len`, then the door is open for memory corruption. D arrays take this possibility into account, making it much easier to avoid such problems. But again, even D's safety features aren't 100% foolproof when calling C functions from D.

There are other, less common, ways array parameters may be declared in C:

```d
void f2(int arr[]);
void f3(int arr[9]);
void f4(int arr[static 9]);
```
Although these parameters are declared using C's array syntax, they boil down to the exact same function signature as `f0` because of the aforementioned pointer decay. The `[9]` in `f3` triggers no special enforcement by the compiler; `arr` is still effectively a pointer to `int` with unknown length. The `[9]` serves as documentation of what the function expects, and the implementation cannot rely on the array having nine elements.

The only potential difference is in `f4`. The `static` added to the declaration tells the compiler that the function must take an array of, in this case, _at least_ nine elements. It could have more than nine, but it can't have fewer. That also rules out null pointers. The problem is, this isn't necessarily enforced. Depending on which C compiler you use, if you shortchange the function and send it less than nine elements you might see warnings if they are enabled, but the compiler might not complain at all. (I haven't tested current compilers for this article to see if any are actually reporting errors for this, or which ones provide warnings.)

The behavior of C compilers doesn't matter from the D side. All we need be concerned with is declaring these functions appropriately so that we can call them from D such that there are no crashes or unexpected results. Because they are all effectively the same, we could declare them all in D like so:


```d
extern(C):
void f0(int* arr);
void f1(int* arr, size_t len);
void f2(int* arr);
void f3(int* arr);
void f4(int* arr);
```
But just because we can do a thing doesn't mean we should. Consider these alternative declarations of `f2`, `f3`, and `f4`:


```d
extern(C):
void f2(int[] arr);
void f3(int[9] arr);
void f4(int[9] arr);
```
Are there any consequences of taking this approach? The answer is yes, but that doesn't mean we should default to `int*` in each case. To understand why, we need first to explore the innards of D arrays.



## The anatomy of a D array



The previous article showed that D makes a distinction between dynamic and static arrays:

```d
int[] a0;
int[9] a1;
```
`a0` is a dynamic array and `a1` is a static array. Both have the properties `.ptr` and `.length`. Both may be indexed using the same syntax. But there are some key differences between them.



### Dynamic arrays



Dynamic arrays are usually allocated on the heap (though that isn't a requirement). In the above case, no memory for `a0` has been allocated. It would need to be initialized with memory allocated via `new` or `malloc`, or some other allocator, or with an array literal. Because `a0` is uninitialized, `a0.ptr` is `null` and `a0.length` is `0`.

A dynamic array in D is an aggregate type that contains the two properties as members. Something like this:


```d
struct DynamicArray {
    size_t length;
    size_t ptr;
}
```
In other words, a dynamic array is essentially a reference type, with the pointer/length pair serving as a handle that refers to the elements in the memory address contained in the `ptr` member. Every built-in D type has a `.sizeof` property, so if we take `a0.sizeof`, we'll find it to be `8` on 32-bit systems, where `size_t` is a 4-byte `uint`, and `16` on 64-bit systems, where `size_t` is an 8-byte `ulong`. In short, it's the size of the handle and not the cumulative size of the array elements.



### Static arrays



Static arrays are generally allocated on the stack. In the declaration of `a1`, stack space is allocated for nine `int` values, all of which are initialized to `int.init` (which is `0`) by default. Because `a1` is initialized, `a1.ptr` points to the allocated space and `a1.length` is `9`. Although these two properties are the same as those of the dynamic array, the implementation details differ.

A static array is a value type, with the value being _all of its elements_. So given the declaration of `a1` above, its nine `int` elements indicate that `a1.sizeof` is `9 * int.sizeof`, or `36`. The `.length` property is a compile-time constant that never changes, and the `.ptr` property, though not readable at compile time, is also a constant that never changes (it's not even an lvalue, which means it's impossible to make it point somewhere else).

These implementation details are why we must pay attention when we cut and paste C array declarations into D source modules.



## Passing D arrays to C



Let's go back to the declaration of `f2` in C and give it an implementation:


```c
void f2(int arr[]) {
    for(int i=0; i<3; ++i)
        printf("%d\n", arr[i]);
}
```
A naïve declaration in D:


```d
extern(C) void f2(int[]);

void main() {
    int[] a = [10, 20, 30];
    f2(a);
}
```
I say naïve because this is never the right answer. Compiling `f2.c` with `df2.d` on Windows (`cl /c f2.c` in the "x64 Native Tools" command prompt for Visual Studio, followed by `dmd -m64 df2.d f2.obj`), then running `df2.exe`, shows me the following output:


    3
    0
    1970470928

There is no compiler error because the declaration of `f2` is pefectly valid D. The `extern(C)` indicates that this function uses the `cdecl` calling convention. Calling conventions affect the way arguments are passed to functions and how the function's symbol is mangled. In this case, the symbol will be either `_f2` or `f2` (other calling conventions, like `stdcall`--`extern(Windows)` in D--have different mangling schemes). The declaration still has to be valid D. (In fact, any D function can be marked as `extern(C)`, something which is necessary when creating a D library that will be called from other languages.)

There is also no linker error. DMD is calling out to the system linker (in this case, Microsoft's `link.exe`), the same linker used by the system's C and C++ compilers. That means the linker has no special knowledge about D functions. All it knows is that there is a call to a symbol, `f2` or `_f2`, that needs to be linked with the implementation. Since the type and number of parameters are not mangled into the symbol name, the linker will happily link with any matching symbol it finds (which, by the way, is the same thing it would do if a C program tried to call a C function which was declared with an incorrect parameter list).

The C function is expecting a single pointer as an argument, but it's instead receiving two values: the array length followed by the array pointer.

The moral of this story is that any C function with array parameters declared using array syntax, like `int[]`, should be declared to accept pointers in D. Change the D source to the following and recompile using the same command line as before (there's no need to recompile the C file):


```d
extern(C) void f2(int*);

void main() {
    int[] a = [10, 20, 30];
    f2(a.ptr);
}
```
Note the use of `a.ptr`. It's an error to try to pass a D array argument where a pointer is expected (with one very special exception, string literals, which I'll cover in the next article in this series), so the array's `.ptr` property must be used instead.

The story for `f3` and `f4` is similar:

```d
void f3(int arr[9]);
void f4(int arr[static 9]);
```
Remember, `int[9]` in D is a static array, not a dynamic array. The following do not match the C declarations:

```d
void f3(int[9]);
void f4(int[9]);
```
Try it yourself. The C implementation:


```c
void f3(int arr[9]) {
    for(int i=0; i<9; ++i)
        printf("%d\n", arr[i]);
}
```
And the D implementation:


```d
extern(C) void f3(int[9]);

void main() {
    int[9] a = [10, 20, 30, 40, 50, 60, 70, 80, 90];
    f3(a);
}
```
This is likely to crash, depending on the system. Rather than passing a pointer to the array, this code is instead passing all nine array elements by value! Now consider a C library that does something like this:


```c
typedef float[16] mat4f;
void do_stuff(mat4f mat);
```
Generally, when writing D bindings to C libraries, it's a good idea to keep the same interface as the C library. But if the above is translated like the following in D:


```d
alias mat4f = float[16];
extern(C) void do_stuff(mat4f);
```
The sixteen floats will be passed to `do_stuff` every time it's called. The same for all functions that take a `mat4f` parameter. One solution is just to do the same as in the `int[]` case and declare the function to take a pointer. However, that's no better than C, as it allows the function to be called with an array that has fewer elements than expected. We can't do anything about that in the `int[]` case, but that will usually be accompanied by a length parameter on the C side anyway. C functions that take typedef'd types like `mat4f` usually don't have a length parameter and rely on the caller to get it right. 

In D, we can do better:


```d
void do_stuff(ref mat4f);
```
Not only does this match the API implementor's intent, the compiler will guarantee that any arrays passed to `do_stuff` are static float arrays with 16 elements. Since a `ref` parameter is just a pointer under the hood, all is as it should be on the C side.

With that, we can rewrite the `f3` example:


```d
extern(C) void f3(ref int[9]);

void main() {
    int[9] a = [10, 20, 30, 40, 50, 60, 70, 80, 90];
    f3(a);
}
```
### Conclusion



Most of the time, when interfacing with C from D, the C API declarations and any example code can be copied verbatim in D. But _most of the time_ is not _all of the time_, so care must be taken to account for those exceptional cases. As we saw in the previous article, carelessness when declaring array variables can usually be caught by the compiler. As this article shows, the same is not the case for C function declarations. Interfacing D with C requires the same care as when writing C code.

In the next article in this series, we'll look at mixing D strings and C strings in the same program and some of the pitfalls that may arise. In the meantime, Steven Schveighoffer's excellent article, "D Slices", [is a great place to start](https://dlang.org/articles/d-array-article.html) for more details about D arrays.

_Thanks to Walter Bright and Átila Neves for their valuable feedback on this article._
