---
author: WalterBright
comments: false
date: 2017-02-22 12:56:07+00:00
layout: post
link: https://dlang.org/blog/2017/02/22/snowflake-strings/
slug: snowflake-strings
title: Snowflake Strings
wordpress_id: 657
categories:
- Compilers &amp; Tools
- Core Team
- Guest Posts
permalink: /snowflake-strings/
redirect_from: /2017/02/22/snowflake-strings/
---

_[Walter Bright](http://walterbright.com/) is the BDFL of the D Programming Language and founder of [Digital Mars](http://digitalmars.com/). He has decades of experience implementing compilers and interpreters for multiple languages, including Zortech C++, the first native C++ compiler. He also created [Empire, the Wargame of the Century](http://www.classicempire.com/)._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)Consider the following D code in **file.d**:

```d
int foo(int i) {
    assert(i < 3);
    return i;
}
```
This is equivalent to the C code:

```c
#include <assert.h>

int foo(int i) {
    assert(i < 3);
    return i;
}
```
The `assert()` in the D code is "lowered" (i.e. rewritten by the compiler) to the following:

    (i < 3 || _d_assertp("file.d", 2))
We're interested in how the compiler writes that string literal, `"file.d"` to the generated object file. The most obvious implementation is to write the characters into the data section and push the address of that to call `_d_assertp()`.

Indeed, that does work, and it's tempting to stop there. But since we're professional compiler nerds obsessed with performance, details, etc., there's a lot more oil in that olive (to borrow one of Andrei's favorite sayings). Let's put it in a press and start turning the screw, because `assert()`s are used a lot.

First off, string literals are immutable (they were originally mutable in C and C++, but are no longer, and D tries to learn from such mistakes). This suggests the string can
be put in read-only memory. What advantages does that deliver?



 	
  * Read-only memory is, well, read only. Attempts to write to it are met with a seg fault exception courtesy of the CPUs memory management logic. Seg faults are a bit graceless, like a cop pulling you over for driving on the wrong side of the road, but at least there wasn't a head-on collision or corruption of the program's memory.

 	
  * Read-only pages are never swapped out by the virtual memory system; they never get marked as "dirty" because they are never written to. They may get discarded and reloaded, but that's much less costly.

 	
  * Read-only memory is safe from corruption by malware (unless the malware infects the MMU, sigh).

 	
  * Read-only memory in a shared library is shared - copies do not have to be made for each user of the shared library.

 	
  * Read-only memory does not need to be scanned for pointers to the heap by the garbage collection system (if the application does GC).


Essentially, shoving as much as possible into read-only memory is good for performance, size and safety. There's the first drop of oil.

The next issue crops up as soon as there's more than one assert:

```d
int foo(int i, int j) {
    assert(i < 3);
    assert(j & 1);
    return i + j;
}
```
The compiler emits two copies of `"file.d"` into the object file. Since they're identical, and read-only, it makes sense to only emit one copy:

```d
string TMP = "file.d";
int foo(int i, int j) {
    (i < 3 || _d_assertp(TMP, 2))
    (j & 1 || _d_assertp(TMP, 3))
    return i + j;
}
```
This is called _string pooling_ and is fairly simple to implement. The compiler maintains a hash table of string literals and their associated symbol names (`TMP` in this case).

So far, this is working reasonably well. But when asserts migrate into header files, macros, and templates, the same string can appear in lots of object files, since the compiler doesn't know what is happening in other object files (the separate compilation model). Other string literals can exhibit this behavior, too, when generic coding practices are used. There needs to be some way to present these in the object file so the linker can pool identical strings.

The dmd D compiler currently supports four different object file formats on different platforms:



 	
  * Elf, for Linux and FreeBSD

 	
  * Mach-O, for OSX

 	
  * MS-COFF, for Win64

 	
  * OMF, for Win32


Each does it in a different way, with different tradeoffs. The methods tend to be woefully under documented, and figuring this stuff out is why I get paid the big bucks.

**Elf**

Elf turns out to have a magic section just for this purpose. It's named `.rodata.strM.N` where `N` is replace by the number of bytes a character has, and `M` is the alignment. For good old `char` strings, that would be `.rodata.str1.1`. The compiler just dumps the strings into that section, and the Elf linker looks through it, removing the redundant strings and adjusting the relocations accordingly. It'll handle the usual string types - `char`, `wchar`, and `dchar` - with aplomb.

There's just a couple flaws. The end of a string is determined by a nul character. This means that strings cannot have embedded nuls, or the linker will regard them as multiple strings and shuffle them about in unexpected ways. One cannot have relocations in those sections, either. This means it's only good for C string literals, not other kinds of data.

This poses a problem for D, where the strings are length-delineated strings, not nul-terminated ones. Does this mean D is doomed to being unable to take advantage of the C-centric file formats and linker design? Not at all. The D compiler simply appends a nul when emitting string literals. If the string does have an embedded nul (allowed in D), it is not put it in these special sections (and the benefit is lost, but such strings are thankfully rare).

**Mach-O**

Mach-O uses a variant of the Elf approach, a special section named `__cstring`. It's more limited in that it only works with single byte `char`s. No `wchar_t`s for you! If there ever was confirmation that UTF-16 and UTF-32 are dead end string types, this should be it.

**MS-COFF**

Microsoft invented MS-COFF by extending the old Unix COFF format. It has many magic sections, but none specifically for strings. Instead, it uses what are called COMDAT sections, one for each string. COMDATs are sections tagged with a unique name, and when the linker is faced with multiple COMDATs with the same name, one is picked and all references to the other COMDATs are rewritten to refer to the Anointed One. COMDATs first appeared in object formats with the advent of C++ templates, since template code generation tends to generate the same code over and over in separate files.

(Isn't it interesting how object file formats are driven by the needs of C and C++?)

The COMDAT for `"hello"` would look something like this:

    <code>??_C@_05CJBACGMB@hello?$AA@:
    db 'h', 'e', 'l', 'l', 'o', 0</code>
The tty noise there is the mangled name of the COMDAT which is generated from the string literal's contents. The algorithm must match across compilation units, as that is how the linker decides which ones are the same (experimenting with it will show that the substring `CJBACGMB` is some sort of hash). Microsoft's algorithm for the mangling and hash is undocumented as far as I can determine, but it doesn't matter anyway, it only has to have a 1:1 mapping between name and string literal. That screams "MD5 hash" to me, so that's what dmd does. The name is an MD5 hash of the string literal contents, which also has the nice property that no matter how large the string gets, the identifier doesn't grow.

COMDATs can have anything stuffed in them, so this is a feature that is usable for a lot more than just strings.

The downside of the COMDAT scheme is the space taken up by all those names, so shipping a program with the debug symbols in it could get much larger.

**OMF**

The caboose is OMF, an ancient format going back to the early 80's. It was extended with a kludgy COMDAT system to support C++ just before most everyone abandoned it. DMD still emits it for Win32 programs. We're stuck with it because it's the only format the default linker (OPTLINK) understands, and so we find a way to press it into service.

Since it has COMDATs, that's the mechanism used. The wrinkle is that COMDATs are code sections or data sections only; there are no other options. We want it to be read-only, so the string COMDATs are emitted as code sections (!). Hey, it works.

**Conclusion**

I don't think we've pressed all the oil out of that olive yet. It may be like `memcpy`, where every new crop of programmers thinks of a way to speed it up.

I hope you've enjoyed our little tour of literals, and may all your string literals be unique snowflakes.

Thanks to Mike Parker for his help with this article.
