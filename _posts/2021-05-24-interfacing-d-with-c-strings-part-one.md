---
author: DBlogAdmin
comments: false
date: 2021-05-24 13:48:44+00:00
layout: post
link: https://dlang.org/blog/2021/05/24/interfacing-d-with-c-strings-part-one/
slug: interfacing-d-with-c-strings-part-one
title: 'Interfacing D with C: Strings Part One'
wordpress_id: 2878
categories:
- Code
- D and C
- Tutorials
permalink: /interfacing-d-with-c-strings-part-one/
redirect_from: /2021/05/24/interfacing-d-with-c-strings-part-one/
---

![Digital Mars D logo]({{ '/assets/images/interfacing-d-with-c-strings-part-one/d6.png' | relative_url }})

This post is part of [an ongoing series](https://dlang.org/blog/the-d-and-c-series/) on working with both D and C in the same project. The previous two posts looked into interfacing D and C arrays. Here, we focus on a special kind of array: strings. Readers are advised to read [Arrays Part One](https://dlang.org/blog/2018/10/17/interfacing-d-with-c-arrays-part-1/) and [Arrays Part Two](https://dlang.org/blog/2020/04/28/interfacing-d-with-c-arrays-and-functions-arrays-part-two/) before continuing with this one.



### The same but different



D strings and C strings are both implemented as arrays of character types, but they have nothing more in common. Even that one similarity is only superficial. We've seen in previous blog posts that D arrays and C arrays are different under the hood: a C array is effectively a pointer to the first element of the array (or, in C parlance, [C arrays decay to pointers](https://stackoverflow.com/questions/1461432/what-is-array-to-pointer-decay), except [when they don't](https://stackoverflow.com/questions/17752978/exceptions-to-array-decaying-into-a-pointer)); a D dynamic array is a _fat pointer_, i.e., a length and pointer pair. A D array does not decay to a pointer, i.e., it cannot be implicitly assigned to a pointer or bound to a pointer parameter in an argument list. Example:


```d
extern(C) void metamorphose(int* a, size_t len);

void main() {
    int[] a = [8, 4, 30];
    metamorphose(a, a.length);      // Error - a is not int*
    metamorphose(a.ptr, a.length);  // Okay
}
```
Beyond that, we've got further incompatibilities:





  * each of D's three string types, `string`, `wstring`, and `dstring`, are encoded as Unicode: UTF-8, UTF-16, and UTF-32 respectively. The C `char*` _can_ be encoded as UTF-8, but it isn't required to be. Then there's the C `wchar_t*`, which differs in bit size between implementations, never mind encoding.


  * all of D's string types are dynamic arrays with immutable contents, i.e., `string` is an alias to `immutable(char)[]`. C strings are mutable by default.


  * the last character of every C string is required to be the NUL character (the escape character `\0`, which is encoded as `0` in most character sets); D strings are not required to be NUL-terminated.



It may appear at first blush as if passing D and C strings back and forth can be a major headache. In practice, that isn't the case at all. In this and subsequent posts, we'll see how easy it can be. In this post, we start by looking at how we can deal with NUL termination and wrap up by digging deeper into the related topic of how string literals are stored in memory.



### NUL termination



Let's get this out of the way first: when passing a D string to C, the programmer must ensure it is terminated with `\0`. [`std.string.toStringz`](https://dlang.org/phobos/std_string.html#.toStringz), a simple utility function in [the D standard library (Phobos)](https://dlang.org/phobos/index.html), can be employed for this:


```d
import core.stdc.stdio : puts;
import std.string : toStringz;

void main() {
    string s0 = "Hello C ";
    string s1 = s0 ~ "from D!";
    puts(s1.toStringz());
}
```
`toStringz` takes a single argument of type `const(char)[]` and returns `immutable(char)*` (there's more about `const` vs. `immutable` in Part Two). The form `s1.toStringz`, known as [UFCS (Uniform Function Call Syntax)](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs), is lowered by the compiler into `toStringz(s1)`.

`toStringz` is the idiomatic approach, but it's also possible to append `"\0"` manually. In that case, `puts` can be supplied with the string's pointer directly:


```d
import core.stdc.stdio : puts;

void main() {
    string s0 = "Hello C ";
    string s1 = s0 ~ "from D!" ~ "\0";
    puts(s1.ptr);
}
```
Forgetting to use `.ptr` will result in a compilation error, but forget to append the `"\0"` and who knows when someone will catch it (possibly after a crash in production and one of those marathon debugging sessions which can make some programmers wish they had never heard of programming). So prefer `toStringz` to avoid such headaches.

However, because strings in D are immutable, `toStringz` does allocate memory from the GC heap. The same is true when manually appending `"\0"` with the append operator. If there's a requirement to avoid garbage collection at the point where the C function is called, e.g., in a `@nogc` function or when `-betterC` is enabled, it will have to be done in the same manner as in C, e.g., by allocating/reallocating space with `malloc/realloc` (or some other allocator) and copying the NUL terminator. (Also note that, in some situations, passing pointers to GC-managed memory from D to C can result in unintended consequences. We'll dig into what that means, and how to avoid it, in Part Two.)

None of this applies when we're dealing directly with string literals, as they get a bit of special treatment from the compiler that makes `puts("Hello D from C!".toStringz)` redundant. Let's see why.



#### String literals in D are special



D programmers very often find themselves passing string literals to C functions. Walter Bright recognized early on how common this would be and decided that it needed to be just as seamless in D as it is in C. So he implemented string literals in a way that mitigates the two major incompatibilities that arise from NUL terminators and differences in array internals:





  1. D string literals are implicitly NUL-terminated.


  2. D string literals are implicitly convertible to `const(char)*`.



These two features may seem minor, but they are quite major in terms of convenience. That's why I didn't pass a literal to `puts` in the `toStringz` example. With a literal, it would look like this:


```d
import core.stdc.stdio : puts;

void main() {
    puts("Hello C from D!");
}
```
No need for `toStringz`. No need for manual NUL termination or `.ptr`. It just works.

I want to emphasize that this only applies to string _literals_ (of type `string`, `wstring`, and `dstring`) and not to string variables; once a string literal is included in an expression, the NUL-termination guarantee goes out the window. Also, no other array literal type is implicitly convertible to a pointer, so the `.ptr` property must be used to bind them to a pointer function parameter, e.g., ``giveMeIntPointer([1, 2, 3].ptr)`.

But there is a little more to this story.



#### String literals in memory



Normal array literals will usually trigger a GC allocation (unless the compiler can elide the allocation, such as when assigning the literal to a static array). Let's do a bit of digging to see what happens with a D string literal:


```d
import std.stdio;

void main() {
    writeln("Where am I?");
}
```
To make use of a command-line tool particularly convenient for this example, I compiled the above on 64-bit Linux with all three major compilers using the following command lines:


```bash
dmd -ofdmd-memloc memloc.d
gdc -o gdc-memloc memloc.d
ldc2 -ofldc-memloc memloc.d
```
If we were compiling C or C++, we could expect to find string literals in the read-only data segment, `.rodata`, of the binary. So let's look there via the `readelf` command, which allows us to extract specific data from binaries in the elf object file format, to see if the same thing happens with D. The following is abbreviated output for each binary:


    readelf -x .rodata ./dmd-memloc | less
    Hex dump of section '.rodata':
      0x0008e000 01000200 00000000 00000000 00000000 ................
      0x0008e010 04100000 00000000 6d656d6c 6f630000 ........memloc..
      0x0008e020 57686572 6520616d 20493f00 2f757372 Where am I?./usr
      0x0008e030 2f696e63 6c756465 2f646d64 2f70686f /include/dmd/pho
    ...
    
    readelf -x .rodata ./gdc-memloc | less
    Hex dump of section '.rodata':
      0x00003000 01000200 00000000 57686572 6520616d ........Where am
      0x00003010 20493f00 00000000 2f757372 2f6c6962  I?...../usr/lib
    ...
    
    readelf -x .rodata ./ldc-memloc | less
    Hex dump of section '.rodata':
      0x00001e40 57686572 6520616d 20493f00 00000000 Where am I?.....
      0x00001e50 2f757372 2f6c6962 2f6c6463 2f783836 /usr/lib/ldc/x86

In all three cases, the string is right there in the read-only data segment. The D spec explicitly avoids specifying where a string literal will be stored, but in practice, we can bank on the following: it might be in the binary's read-only segment, or it might be in the normal data segment, but it won't trigger a GC allocation, and it won't be allocated on the stack.

Wherever it is, there's a positive consequence that we can sometimes take advantage of. Notice in the `readelf` output that there is a dot (`.`) immediately following the question mark at the end of each string. That represents the NUL terminator. It is not counted in the string's `.length` (so `"Where am I?".length` is 11 and not 12), but it's still there. So when we initialize a string variable with a string literal or assign a string literal to a variable, the lack of an allocation also means there's no copying, which in turn means the variable is pointing to the literal's location in memory. And that means we can safely do this:


```d
import core.stdc.stdio: puts;

void main() {
    string s = "I'm NUL-terminated.";
    puts(s.ptr);
    s = "And so am I.";
    puts(s.ptr);
}
```
If you've read [the GC series on this blog](https://dlang.org/blog/the-gc-series/), you are aware that the GC can only have a chance to run a collection if an attempt is made to allocate from the GC heap. More allocations mean a higher chance to trigger a collection and more memory that needs to be scanned when a collection runs. Many applications may never notice, but it's a good policy to avoid GC allocations when it's easy to do so. The above is a good example of just that: `toStringz` allocates, we don't need it in either call to `puts` because we can trust that `s` is NUL-terminated, so we don't use it.

To be very clear: this is only true for string variables that have been directly initialized with a string literal or assigned one. If the value of the variable was the result of any other operation, then it cannot be considered NUL-terminated. Examples:

```d
string s1 = s ~ "...I'm Unreliable!!";
string s2 = s ~ s1;
string s3 = format("I'm %s!!", "Unreliable");
```
None of these strings can be considered NUL-terminated. Each case will trigger a GC allocation. The runtime pays no mind to the NUL terminator of any of the literals during the append operations or in the `format` function, so the programmer can't trust it will be part of the result. Pass any one of these strings to C without first terminating it and trouble will eventually come knocking.



#### But hold on…



Given that you're reading a D blog, you're probably adventurous or like experimenting. That may lead you to discover another case that looks reliable:


```d
import core.stdc.stdio: puts;

void main() {
    string s = "Am I " ~ "reliable?";
    puts(s.ptr);
}
```
The above very much looks like appending multiple string literals in an initialization or assignment is just as reliable as using a single string literal. We can strengthen that assumption with the following:


```d
import std.stdio : writeln;

void main() {
    writeln("Am I reliable?".ptr);

    string s = "Am I " ~ "reliable?";
    writeln(s.ptr);
}
```
`writeln` is a templated function that recognizes when it's being given a pointer; rather than treating it as a string and printing what it points to, it prints the pointer's value. So we can print memory addresses in D without a format string.

Compiling the above, again on 64-bit Linux:


```bash
dmd -ofdmd-rely rely.d
gdc -o gdc-rely rely.d
ldc2 -ofldc-rely rely.d
```
Now let's execute them all:


```bash
./dmd-rely
562363F63010
562363F63030

./gdc-rely
5566145E0008
5566145E0008

./ldc-rely
55C63CFB461C
55C63CFB461C
```
We see that `dmd-rely` prints two different addresses, but they're very close together. Both `gdc-rely` and `ldc-rely` print a single address in both cases. And if we make use of `readelf` as we did with the `memloc` example above, we'll find that, in every case, the literals are in the read-only data segment. Case closed!

Well, not so fast.

What's happening is that all three compilers are performing an optimization [known as _constant folding_](https://en.wikipedia.org/wiki/Constant_folding). In short, they can recognize when all operands of an append expression are compile-time constants, so they can perform the append at compile-time to produce a single string literal. In this case, the net effect is the same as `s = "Am I reliable?"`. LDC and GDC go further and recognize that the resulting literal is identical to the one used earlier, so they reuse the existing literal's address ([a.k.a. string interning](https://en.wikipedia.org/wiki/String_interning)). (Note that DMD also performs string interning, but currently it only kicks in when a string literal appears more than twice.)

To be clear: this only works because all of the operands are string literals. No matter how many string literals are involved in an operation, if only one operand is a variable, then the operation triggers a GC allocation.

Although we see that the result of an append operation involving string literals can be passed directly to C just fine, and we've proven that it's stored in read-only memory alongside its NUL terminator, _this is not something we should consider reliable_. It's an optimization that no compiler is required to perform. Though it's unlikely that any of the three major D compilers will suddenly stop constant folding string literals, a future D compiler could possibly be released without this particular optimization and instead trigger a GC allocation.

In short: rely on this at your own risk.

**Addendum**: Compile `rely.d` on Windows with dmd and the binary will yield some very different output:


```bash
dmd -m64 -ofwin-rely.exe rely.d
./win-rely
7FF76701D440
7FF76702BB30
```
There is a much bigger difference in the memory addresses here than in the dmd binary on Linux. We're dealing with the PE/COFF format in this case, and I'm not familiar with anything similar to `readelf` for that format on Windows. But I do know a little something about [Abner Fog's `objconv` utility](https://www.agner.org/optimize/#objconv). Not only does it convert between object file formats, but it can also disassemble them:

```bash
objconv -fasm win-rely.obj
```
This produces a file, `win-rely.asm`. Open it in a text editor and search for a portion of the string, e.g., `"I rel"`. You'll find the two entries aren't too far apart, but one is located in a block of text under this heading:




    rdata SEGMENT PARA 'CONST' ; section number 4




And the other under this heading:




    .data$B SEGMENT PARA 'DATA' ; section number 6




In other words, one of them is in the read-only data segment (`rdata SEGMENT PARA 'CONST'`), and the other is in the regular data segment. This goes back to what I mentioned earlier about the D spec being explicitly silent on where string literals are stored. Regardless, the behavior of the program on Windows is the same as it is on Linux; the second call to `puts` doesn't blow anything up because the NUL terminator is still there, one slot past the last character. But it doesn't change the fact that constant folding of appended string literals is an optimization and only to be relied upon at your own risk.



### Conclusion



This post provides all that's needed for many of the use cases encountered with strings when interacting with C from D, but it's not the complete picture. In Part Two, we'll look at how mutability, immutability, and constness come into the picture, how to avoid a potential problem spot that can arise when passing GC-allocated D strings to C, and how to get D strings from C strings. We'll save encoding for Part Three.

_Thanks to Walter Bright, Ali Çehreli, and Iain Buclaw for their valuable feedback on this article._
