---
author: MaxHaughton
comments: false
date: 2022-01-14 13:32:07+00:00
layout: post
link: https://dlang.org/blog/2022/01/14/using-the-gcc-static-analyzer-on-the-d-programming-language/
slug: using-the-gcc-static-analyzer-on-the-d-programming-language
title: Using the GCC Static Analyzer on the D Programming Language
wordpress_id: 3033
categories:
- Compilers &amp; Tools
permalink: /using-the-gcc-static-analyzer-on-the-d-programming-language/
redirect_from: /2022/01/14/using-the-gcc-static-analyzer-on-the-d-programming-language/
---

![]({{ '/assets/images/using-the-gcc-static-analyzer-on-the-d-programming-language/bug-200x156.jpg' | relative_url }})

Largely thanks to [the tireless work of Iain Buclaw](https://github.com/ibuclaw), the D programming language is part of GCC. As well as having access to an extremely potent set of compiler optimizations and a large group of target platforms, D also benefits from upstream features added to GCC as a whole or even for specific languages. For some projects, this can be very important, as some of these features require large quantities of careful work, for example, mitigations for transient execution vulnerabilities.

A few years ago, [thanks to David Malcolm](https://developers.redhat.com/author/david-malcolm) at Red Hat, GCC gained a static analyzer. This uses a set of algorithms at compile time to find patterns in a program that would lead to memory safety bugs when the program is executed.



## How do I turn it on?



Run GDC like you normally would and add the `-fanalyzer` flag. If you're already bored of reading and want to have a go, please use Matt Godbolt's excellent compiler explorer. [Start with this simple example](https://d.godbolt.org/z/Yz4n6c9nj).



## Which patterns does it look for?





### Some memory bugs



From the GCC documentation, we can get a list of every warning the analyzer can emit:


    -Wanalyzer-double-fclose 
    -Wanalyzer-double-free 
    -Wanalyzer-exposure-through-output-file 
    -Wanalyzer-file-leak 
    -Wanalyzer-free-of-non-heap 
    -Wanalyzer-malloc-leak 
    -Wanalyzer-mismatching-deallocation 
    -Wanalyzer-possible-null-argument 
    -Wanalyzer-possible-null-dereference 
    -Wanalyzer-null-argument 
    -Wanalyzer-null-dereference 
    -Wanalyzer-shift-count-negative 
    -Wanalyzer-shift-count-overflow 
    -Wanalyzer-stale-setjmp-buffer 
    -Wanalyzer-tainted-array-index 
    -Wanalyzer-unsafe-call-within-signal-handler 
    -Wanalyzer-use-after-free 
    -Wanalyzer-use-of-pointer-in-stale-stack-frame 
    -Wanalyzer-write-to-const 
    -Wanalyzer-write-to-string-literal 

These names are fairly descriptive. However, let's take a look at some examples before going into detail.

Let's say we have some code that allocates a buffer for itself via `malloc`, like the following.


```d
int usesTheHeap(size_t x)
{
    import core.stdc.stdlib : malloc, free;
    int[] slice = (cast(int*) malloc(int.sizeof * x))[0..x];
    slice[] = 0;
    // Algorithm goes here
    return 0;
}
```
For this code, the static analyzer gives us two warnings, the first of which is the following:


    warning: leak of 'slice.ptr' [CWE-401]
       11 | }
          | ^
      'usesTheHeap': events 1-3
        |
        |    8 |     int[] slice = (cast(int*) malloc(int.sizeof * x))[0..x];
        |      |                                     ^
        |      |                                     |
        |      |                                     (1) allocated here
        |    9 |     slice[] = 0;
        |      |     ~                                
        |      |     |
        |      |     (2) assuming 'slice.ptr' is non-NULL
        |   10 |     // Algorithm goes here
        |   11 | }
        |      | ~                                    
        |      | |
        |      | (3) 'slice.ptr' leaks here; was allocated at (1)

As you might expect, since we didn't free the memory we allocated, the analyzer warns us that the memory leaks at the end of the scope.

The second warning complains that we used the memory from `malloc` without checking if it was `null`. Program failure due to dereferencing a null-pointer is sometimes desirable in D, so you can turn this off with `-Wno-analyzer-possible-null-dereference` if you need to.

Thanks to `assert` being built into the core language and being lowered to a construct that GCC understands, we can use it to make the analyzer assume a pointer is non-null:


```d
int usesTheHeap(size_t x)
{
    import core.stdc.stdlib : malloc, free;
    void* allocatedBuffer = malloc(int.sizeof * x);
    assert(allocatedBuffer != null);
    // The program may not proceed if the pointer is null
    int[] slice = (cast(int*) allocatedBuffer)[0..x];
    slice[] = 0; //So the analyzer knows this is safe.
    // Algorithm goes here
    return 0;
}
```
### More than `malloc` and `free`



Let's think about something that (obviously) uses memory, but isn't always considered part of memory safety: although it's not encouraged, you can use `setjmp` and `longjmp` from C in D code. As with many C features, these really can blow up in your face.

Look at the following:


```d
import core.sys.posix.setjmp;

void main()
{
    jmp_buf local;
    void set()
    {
        setjmp(local);
    }
    set();
    longjmp(local, 0);
} 
```
We set the buffer inside `set`, but the buffer is now primed, ready, and pointing to nothing (technically it is something but that something is chaotic). Thankfully, the analyzer can warn us about this as in the following:


```d
<source>: In function 'D main':
<source>:11:12: warning: 'longjmp' called after enclosing function of 'setjmp' has returned [-Wanalyzer-stale-setjmp-buffer]
   11 |     longjmp(local, 0);
      |            ^
  'D main': events 1-2
    |
    |    3 | void main()
    |      |      ^
    |      |      |
    |      |      (1) entry to 'D main'
    |......
    |   10 |     set();
    |      |        ~
    |      |        |
    |      |        (2) calling 'set' from 'D main'
    |
    +--> 'set': events 3-5
           |
           |    6 |     void set()
           |      |          ^
           |      |          |
           |      |          (3) entry to 'set'
           |    7 |     {
           |    8 |         setjmp(local);
           |      |               ~
           |      |               |
           |      |               (4) 'setjmp' called here
           |    9 |     }
           |      |     ~     
           |      |     |
           |      |     (5) stack frame is popped here, invalidating saved environment
           |
    <------+
    |
  'D main': events 6-7
    |
    |   10 |     set();
    |      |        ^
    |      |        |
    |      |        (6) returning to 'D main' from 'set'
    |   11 |     longjmp(local, 0);
    |      |            ~
    |      |            |
    |      |            (7) 'longjmp' called after enclosing function of 'setjmp' returned at (5)
    |
```
### Beyond skin-deep



While important, stack corruption and (simple) memory leaks are old hat; catching them is usually relatively (touch wood) easy with modern programming practices, programming language design (i.e., sound memory safety analysis), sanitizers, and toolings like Valgrind or your favorite debugger. For less trivial issues, _finding_ the issues when they happen in a controlled environment is still relatively easy with the above tools if the program fails, but finding _why_ they happened could require manually instrumenting the program. Finding issues early is important and appreciated.

The analyzer is interprocedural, i.e., it can see across function boundaries (when the information is available). In some older codebases you can sometimes see code like this:


```d
struct Handle
{
    void* x;
    void reset()
    {
        free(x);
    }
    ~this()
    {
        free(x);
    }
}
void accept(Handle x)
{
    x.reset();
    // Destructor called 
}
```
This yields a double-free. The analyzer is able to see "inside" the destructor and thus correctly warns about the double-free and what causes it.

The following seems to be sensitive to the optimization settings used but is very important when it works: iterator invalidation. That is to say, we hand out a pointer to somewhere, end up (say) `realloc`-ing, and suddenly that pristine pointer is now a pointer to absolutely nowhere.


```python
struct Vector
{
    int* handle;
    void expand(size_t sz)
    {
        int* newPtr = cast(int*) realloc(handle, sz);
        assert(newPtr);
        handle = newPtr;
    }
    ~this()
    {
        free(handle);
    }
}
void iter(Vector x)
{
    int* copy = x.handle;
    x.expand(1000);
    *copy = 3;
}
```
The analyzer sees this and spits out the following:


```d
<source>: In function 'iter':
<source>:23:11: warning: use after 'free' of 'copy_5' [CWE-416] [-Wanalyzer-use-after-free]
   23 |     *copy = 3;
      |           ^
  'iter': events 1-2
    |
    |   19 | void iter(Vector x)
    |      |      ^
    |      |      |
    |      |      (1) entry to 'iter'
    |......
    |   22 |     x.expand(1000);
    |      |             ~
    |      |             |
    |      |             (2) calling 'expand' from 'iter'
    |
    +--> 'expand': events 3-7
           |
           |    8 |     void expand(size_t sz)
           |      |          ^
           |      |          |
           |      |          (3) entry to 'expand'
           |    9 |     {
           |   10 |         int* newPtr = cast(int*) realloc(handle, sz);
           |      |                                         ~
           |      |                                         |
           |      |                                         (4) freed here
           |      |                                         (5) when '__builtin_realloc' succeeds, moving buffer
           |   11 |         assert(newPtr);
           |      |         ~ 
           |      |         |
           |      |         (6) following 'false' branch...
           |   12 |         handle = newPtr;
           |      |                ~
           |      |                |
           |      |                (7) ...to here
           |
    <------+
    |
  'iter': events 8-9
    |
    |   22 |     x.expand(1000);
    |      |             ^
    |      |             |
    |      |             (8) returning to 'iter' from 'expand'
    |   23 |     *copy = 3;
    |      |           ~  
    |      |           |
    |      |           (9) use after 'free' of 'copy_5'; freed at (4)
    |
```
### Inline assembly



The analyzer was partly intended to help eliminate bugs in the Linux kernel. As such, it is useful to be able to analyze inline assembly (which is commonplace in the kernel). An example will not be given here, but GCC has gained the ability to analyze basic X86 inline assembly.



## Some idiosyncrasies



The static analyzer is implemented as just another pass inside GCC (there are hundreds). This means that some warnings may magically disappear under certain optimization settings as the compiler eliminates dead code and propagates information.

Similarly, the quality of output does vary with the flags used. We won't discuss it here, but options exist to increase the usefulness of diagnostics by performing more sophisticated analysis, for example, by propagating constraints through analyzed branches and thus eliminating some paths which are superficially "possible" but can, in fact, be eliminated by considering the semantics of the code.



## Finding bugs when combining C and D



The static analyzer was designed for use with C (and C++, but mostly the former) and [operates on GCC's IR](https://en.wikipedia.org/wiki/Intermediate_representation). If we use link-time optimization, we can combine the IR from compilation units in different languages (D and C), then use the analyzer to look for bugs across language boundaries.

Let's say we have an unfortunate C library with two functions, `doWork` and `terminate`. They both accept `void*`, but they expect the memory to be allocated by the user of the library rather than by a matching `init` function.


```c
#include <stdlib.h>
void doWork(void* ptr)
{
    // Do something, doesn't matter what here
}
void terminate(void* ptr)
{
    // Clean up things attached to ptr
    free(ptr);
}
```
Assuming we have no access to the C source and assuming the library documentation fails to mention that `terminate` calls `free`, we would likely write the following code:


```d
extern(C) void doWork(void*);
extern(C) void terminate(void*);

void main()
{
    import core.stdc.stdlib : malloc, free;
    void* buf = malloc(100);
    scope(exit) free(buf);
    buf.doWork();
    buf.terminate();
}
```
If we're lucky, we'll see an error message like


    free(): double free detected in tcache 2
    Aborted (core dumped)

which is better than nothing but nonetheless not ideal if we were unfamiliar with the code.

If instead, we compile with `gdc d.d c.c -fanalyzer -flto` (the last flag is essential), we get this warning:


```d
In function ‘D main’:
d.d:11:14: warning: double-‘free’ of ‘buf_6’ [CWE-415] [-Wanalyzer-double-free]
   11 |  scope(exit) free(buf);
      |              ^
  ‘D main’: event 1
    |
    |/usr/lib/gcc/x86_64-linux-gnu/10/include/d/__entrypoint.di:33:5:
    |   33 | int _Dmain(char[][] args);
    |      |     ^
    |      |     |
    |      |     (1) entry to ‘D main’
    |
  ‘D main’: events 2-3
    |
    |d.d:10:8:
    |   10 |  void* buf = malloc(100);
    |      |        ^
    |      |        |
    |      |        (2) allocated here
    |......
    |   13 |  buf.terminate();
    |      |  ~
    |      |  |
    |      |  (3) calling ‘terminate’ from ‘D main’
    |
    +--> ‘terminate’: events 4-5
           |
           |c.c:6:6:
           |    6 | void terminate(void* ptr)
           |      |      ^
           |      |      |
           |      |      (4) entry to ‘terminate’
           |    7 | {
           |    8 |     free(ptr);
           |      |     ~
           |      |     |
           |      |     (5) first ‘free’ here
           |
    <------+
    |
  ‘D main’: events 6-7
    |
    |d.d:13:2:
    |   11 |  scope(exit) free(buf);
    |      |              ~
    |      |              |
    |      |              (7) second ‘free’ here; first ‘free’ was at (5)
    |   12 |  buf.doWork();
    |   13 |  buf.terminate();
    |      |  ^
    |      |  |
    |      |  (6) returning to ‘D main’ from ‘terminate’
    |
```
This found our bug straight away. Thank you very much, static analysis.



## Conclusion



The way this analyzer is implemented can serve as a lesson on the usefulness of IRs as a tool for analysis rather than merely optimization. A similar analysis is currently performed on the AST in the D frontend, but that's slow and fairly ugly to write (let alone read).

I don't think using a static analyzer is a replacement for a carefully designed language-level memory safety story, but I am very glad it exists. The fact that it is usable and useful from D is a testament to the benefits of D's presence in GCC and diversity of implementation.
