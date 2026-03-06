---
author: RainerSchuetze
comments: false
date: 2017-12-20 13:45:07+00:00
excerpt: Rainer Schuetze is the creator and maintainer of Visual D, the D plugin for
  Visual Studio. Recently, he implemented a new name mangling algorithm for the D
  frontend, which was released in DMD 2.077.0. In this post, he explains why it was
  needed and what it does.
layout: post
link: https://dlang.org/blog/2017/12/20/ds-newfangled-name-mangling/
slug: ds-newfangled-name-mangling
title: D's Newfangled Name Mangling
wordpress_id: 1286
categories:
- Code
- Compilers &amp; Tools
- Guest Posts
permalink: /ds-newfangled-name-mangling/
redirect_from: /2017/12/20/ds-newfangled-name-mangling/
---

_Rainer Schuetze is the creator and maintainer of [Visual D](http://rainers.github.io/visuald/visuald/StartPage.html), the D plugin for Visual Studio. Recently, he implemented a new name mangling algorithm for the D frontend, which was released in [DMD 2.077.0](https://dlang.org/changelog/2.077.0.html). In this post, he explains why it was needed and what it does._



* * *





### What is symbol name mangling?


![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)

D embraces the separate compilation model that compiles D source code to object files and uses a linker to bind the object files to an executable binary file. This allows the reuse of precompiled object files and libraries, speeding up the build process. As the linker is usually one that’s also used for other languages with the same compilation model, e.g. C/C++ or Fortran, mixing object files from different languages is straightforward.

In an object file, a symbol name is assigned to each function or global variable, both when it is defined and when it is used via a call or access. The linker uses these names to connect references to definitions of the same name with only very bare knowledge about the symbol. For example, the symbol for this C function declaration,

```d
extern(C) const(char)* find(int ch, const(char)* str);
```
does not tell the linker anything about function arguments or return type, as the C language uses the plain function name `find` as the symbol name (some platforms prepend a `_` to the symbol). If you later change the order of the arguments to

```d
extern(C) const(char)* find(const(char)* str, int ch);
```
but fail to update and recompile all source files that use the new declarartion, the linker will happily bind the resulting object files. In that case, the program is likely to crash, since a character passed to the function will be interpreted as a string pointer and vice versa.

D and C++ avoid this problem by adding more information to the symbol name, i.e. they encode into a symbol name the scope in which the symbol is defined, the function argument types, and the return type. Even if the linker does not interpret this information, linking fails with an _undefined symbol error_ if the definitions used to build the object files don’t match. For example, the D function declaration

    module test;
    extern(D) const(char)* find(int ch, const(char)* str);
has a symbol name `_D4test4findFiPxaZPxa`, where `_D` is a prefix to identify the symbol as being generated from a D source symbol, `4test4find` encodes the “fully qualified name” `find` in module `test`, and `FiPxaZPxa` describes the function type with an integer argument (designated by `i`) and the C-style string pointer type `Pxa` by just concatenating the encodings for argument types. `Z` terminates the function argument list and is followed by the encoding for the return type, again `Pxa` for a C-style string pointer. In contrast,

    extern(D) const(char)* find(const(char)* str, int ch);
is encoded as `_D4test4findFPxaiZPxa`, making it a different symbol with the argument types reversed. The encoding ensures a normalized representation of types and scopes while also providing shorter symbols than minimal source code. This encoding is called “name mangling”.

_Ed: Note that `extern(C)` and `extern(D)` are [linkage attributes](https://dlang.org/spec/attribute.html#linkage). When a function is declared in D without an explicit linkage attribute, `extern(D)` is the default._

In D, some function attributes are also mangled into the symbol name, e.g. `@safe`, `nothrow`, `pure` and `@nogc`. In theory, mangling could also cover parameter names, [user defined attributes](https://dlang.org/spec/attribute.html#UserDefinedAttribute), or even [contracts](https://dlang.org/spec/contracts.html), but that is currently considered excessive.

Please note that even though name mangling can detect some mismatches in the binary interface of functions (i.e. how arguments are passed in registers or on the stack), it won’t catch every error; for example, structs, classes and other user defined types are mangled by name only, so that a change to their definition will still pass unnoticed by the linker.

The mangled name of a symbol is also available during compilation using the `.mangleof` property. This used to be exploited to provide type reflection of the symbol at compile time. This should no longer be necessary due to the introduction of new [`__traits`](https://dlang.org/spec/traits.html) that make this information accessible faster and more convenient, for example,

    __traits(getLinkage,symbol);
or

    __traits(getFunctionAttributes, symbol);
Thus, usage of `.mangleof` is not recommended except for debugging purposes.

When reversing the mangling process in the “demangler”, all the encoded information is kept to make it available to the user, but that does not always yield correct D syntax. The first definition above demangles as

    const(char)* test.find(int, const(char)*)
i.e. the module name `test` is added to the function name.


### Template symbols


The two definitions of `find` shown above can coexist in D and C++, so name mangling is not only a way to detect errors at link time but also a necessity to represent overloads. It should at least contain enough information to distinguish different overloads of the same scoped identifier.

This becomes even more obvious when considering templates that usually instantiate different functions or variable definitions for each argument type. In D, the template instantiation information is added to the qualified name of a symbol.

Consider expression templates, a common example of meta programming used for delayed evaluation of expressions:

```d
module expr;
struct Mul(X,Y)
{
    X x;
    Y y;
}
struct Add(X,Y)
{
    X x;
    Y y;
}

auto mul(X,Y)(X x, Y y) { return Mul!(X,Y)(x, y); }
auto add(X,Y)(X x, Y y) { return Add!(X,Y)(x, y); }
```
A function template is lowered by the compiler to [an eponymous template](https://dlang.org/spec/template.html#implicit_template_properties):

```d
template mul(X, Y)
{
    auto mul(X x, Y y) { return Mul!(X,Y)(x, y); }
}
```
The template name is part of the qualified function name, `expr.mul!(X,Y).mul`, and the auto return type is inferred to be `Mul!(X,Y)`. This causes the symbol to reference the types `X` and `Y` three times. The demangled mangled name of an instantiation with types `double` and `float` of this template is

    expr.Mul!(double,float) expr.mul!(double,float).mul(double,float)
The mangling process of DMD before version 2.077 walks the abstract syntax tree of the declaration and emits the mangled representation of the types whenever it is hit. Now consider stacking operations, e.g.

```d
auto square(X)(X x) { return mul(x, x); }

auto len = square("var");
pragma(msg, len.square.mangleof);
// S4expr66__T3MulTS4expr16__T3MulTAyaTAyaZ3MulTS4expr16__T3MulTAyaTAyaZ3MulZ3Mul

pragma(msg, typeof(len).mangleof.length);
pragma(msg, len.square.mangleof.length);
pragma(msg, len.square.square.mangleof.length);
pragma(msg, len.square.square.square.mangleof.length);
pragma(msg, len.square.square.square.square.mangleof.length);
pragma(msg, len.square.square.square.square.square.mangleof.length);
pragma(msg, len.square.square.square.square.square.square.mangleof.length);
```
With DMD 2.076 or earlier, this displays `28u, 78u, 179u, 381u, 785u, 1594u, 3212u`, showing exponential growth of the mangled symbol name length even though the expression in the source code just grows linearly. This happens because types like `Mul!(Mul!(string, string), Mul!(string, string))` are combined and the mangling repeats their full representation every time they are referenced.

Create a chain of 12 calls to `square` above and the symbol length increases to 207,114. Even worse, the resulting object file for COFF/64-bit is larger than 15 MB and the time to compile increases from 0.1 seconds to about 1 minute. Most of that time is spent generating code for functions only used at compile time.

[Voldemort types](http://www.digitalmars.com/articles/b79.html) returned from template functions can be similar, as they carry the function signature including the template arguments as [part of the type name](https://issues.dlang.org/show_bug.cgi?id=15831). These can also show a dramatic increase in build times without generating as much code as in the example.


### Symbol compression to the rescue


In early 2016, a couple of attempts were made to shorten these long symbols:



 	
  * cut off symbol names if they exceed a given threshold, but append a checksum of the full symbol instead. This was already done with an MD5 hash when emitting symbols for the DigitalMars C compiler tool chain as the OMF object file format does not allow symbols longer than 255 characters. The downside to this is that these symbols can no longer be demangled, so that symbols in linker messages cannot be translated back into human digestible names.

 	
  * apply binary compression to the symbol name. Standard techniques use part of the full symbol name as a dictionary to encode repetitions within the name. This is usually done by encoding a position-length pair using characters outside the normal identifier set. Again, this is already in use when DMD tries to fit symbols into the OMF limit of 255 characters (before applying the MD5 hash trick), but it also has shown some disadvantages: when using characters above the ASCII range, this interferes with UTF8 encoded characters that are also allowed as symbol characters in the D language. It can also break linker output as the console might misinterpret it as a locale-specific character encoding. Avoiding this by applying a binary to ASCII conversion like base64 to the symbols would obfuscate the actual symbol name even more.

 	
  * extend [the mangling grammar](https://dlang.org/spec/abi.html#name_mangling) by allowing references to entities already encoded. This is similar to binary compression, but does not need to encode match length as the entities have terminators embedded into the grammar. The most prominent entities are types. This is the road C++ has taken, as it is also affected by the issues described here. In C++, name mangling is not standardised by the language, but by the compiler or the platform. GNU g++ uses [Itanium C++ ABI mangling](https://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangling), which does a pretty good job with C++ code similar to the example above or in the [Voldemort issue](https://issues.dlang.org/show_bug.cgi?id=15831). Even though Microsoft’s Visual C++ can encode recurring types as well, it still generates very long names because it limits the encoding to the first 10 types of an argument list.


The first attempts at applying the latter scheme to the mangling of D symbols showed disappointing results. As it turned out, these implementations missed a subtle detail of the [mangler in the DMD front end](https://github.com/dlang/dmd/blob/master/src/dmd/dmangle.d) at that time; it reused cached representations of mangled type names to combine them to more complex types. This fails to find repetitions of the types from which a cached type name was built.

This is where I stepped in to create a proof-of-concept version of the mangling [without these omissions](https://github.com/dlang/dmd/pull/5855). Early results [were promising](https://issues.dlang.org/show_bug.cgi?id=15831#c7), so I looked for more opportunities to reduce symbol length:



 	
  * with fully qualified names always containing the package and module names of a symbol, identifiers tend to appear often in a mangled name.

 	
  * qualified names are likely to come from the same module or package, so it would be nice to encode them as a single entity.


The unit tests of [the Phobos runtime library](https://dlang.org/phobos/index.html) are benchmark candidates, as they contain a lot of symbols for template-heavy code. At the given time there were 127,172 symbols found in the map file of the Windows build. These were the results of the different manglings:
<table > 

<tr >
back references
max length
average length
</tr>

<tbody >
<tr >

<td >none
</td>

<td >416133
</td>

<td >369
</td>
</tr>
<tr >

<td >types
</td>

<td >2095
</td>

<td >157
</td>
</tr>
<tr >

<td >types+identifiers
</td>

<td >1263
</td>

<td >128
</td>
</tr>
<tr >

<td >types+identifiers+qualified names
</td>

<td >1114
</td>

<td >117
</td>
</tr>
</tbody>
</table>
(This has been measured with the implementation at the time, which is not exactly the same as the final mangling; it used special characters for the different back reference types, but this turned out not to be a good idea. The D mangling is supposed to be the same on all platforms, but these characters will have a special meaning to the linker on one of them.)

It's rather simple in DMD to determine the identity of identifiers and types, as the latter are merged according to their mangling anyway. Qualified names and their associated symbols turned out to introduce a number of complications, though. Namely, `mangleFunc` in `core.demangle` allows building a mangled name of a function from a fully qualified name given as a `string` function argument and a type specified as a template argument. Implementing this for run-time usage requires copying the full mangling machinery and introspection capabilities of the compiler, which is unrealistic. Considering the limited benefit shown by the above Phobos statistics, the idea of encoding qualified names was dropped.

Here are some details about the new mangling:



 	
  * Back references are now encoded by the character `Q`, followed by the relative position of the original appearance of the same identifier or type. These positions are encoded with respect to base 26, with the last digit encoded by a lowercase letter and the other digits encoded by an uppercase letter. That way, most back references are 2 or 3 characters long, 4 in extreme cases. Using a different encoding for the last digit allows determining the end of a number without looking at the next character. This helps to avoid ambiguities. (The Itanium C++ ABI mangling uses base 36 encoding by combining numbers and letters, but need a termination character `_`.)

 	
  * Counting encodable entities as in the C++ mangling would result in slightly shorter mangled names, but needs the mangler to keep a dynamic list of respective positions. The current demangler is designed not to allocate as long as the supplied output buffer is large enough.

 	
  * Relative positions are chosen instead of absolute positions to allow prepending the `_D` prefix without having to re-encode the symbol. Some platforms also prepend an additional underscore, for which the relative positions are agnostic.

 	
  * The mangling grammar sometimes allows types and identifiers at the same position, so a demangler needs to distinguish between the two even if given by a back reference. That’s why a lookup to the referenced position is necessary to continue demangling; an identifier always starts with a number, while a type always starts with a letter.

 	
  * Using `Q` for back references grabs the last free letter used to encode types, but there is at least one type defined in the mangling grammar that is not supposed to appear in a mangling anyway (namely `TypeIdent`), so it can be resurrected if the necessity appears.


For example, the expression template type shown above now mangles as

    pragma(msg, len.square.mangleof);
    // S4expr__T3MulTSQo__TQlTAyaTQeZQvTQtZQBb
    //                ^^   ^^     ^^ ^^ ^^ ^^^ decode to:
    //                |    |      |  |  |  |
    //                |    |      |  |  |  +- 3Mul
    //                |    |      |  |  +---- S4expr__T3MulTAyaTAyaZ3Mul
    //                |    |      |  +------- 3Mul
    //                |    |      +---------- Aya
    //                |    +----------------- 3Mul
    //                +---------------------- 4expr
with a length of 39 instead of 78 without back references. The resulting sizes are 23, 39, 57, 76, 95, 114, 133 showing linear growth. The chain of 12 calls to `square` shrinks from 207,114 characters to 247, i.e. by a factor of more than 800.

Implementing `mangleFunc` mentioned above for the mangling with back referencing identifiers still is not obvious; while the fully qualified name is not supposed to contain any types (e.g. as a struct template argument) identifiers in the mangled name can appear again in the function type. This was solved by extending the demangler to use [“Design by Introspection” (DbI)](https://dconf.org/2017/talks/alexandrescu.pdf) (as coined by Andrei Alexandrescu):



 	
  * make the `Demangle` struct a template that parameterizes on a struct that supplies a couple of hooks

```d
struct NoHooks {}  // supports: static bool parseLName(ref Demangle); ...
private struct Demangle(Hooks = NoHooks)
{
Hooks hooks;
    // ...
    void parseLName()
    {
        static if(__traits(hasMember, Hooks, "parseLName"))
            if (hooks.parseLName(this))
                return;
            // normal decode...
    }
}
```
  * create a hook that replaces a reoccurring identifier with the appropriate back reference

```d
struct RemangleHooks
{
    char[] result;
    size_t[const(char)[]] idpos;
    // ...
    bool parseLName(ref Demangler!RemangleHooks d)
    {
        // flush input so far to result[]
        if (d.front == 'Q')
        {
            // re-encode back reference...
        }
        else if (auto ppos = currentIdentifier in idpos)
        {
            // encode back reference to identifier at *ppos
        }
        else
        {
            idpos[currentIdentifier] = currentPos;
        }
        return true;
    }
}
```
  * combine the qualified name and the type as before (`core.demangle` is still capable of decoding it) and run it through the hooked demangler

```d
char[] mangleFunc(FuncType)(const(char)[] qualifiedName)
{
    const(char)mangledQualifiedName = encodeLNames(qualifiedName);
    const(char)mangled = mangledQualifiedName ~ FuncType.mangleof;
    auto d = Demangle!RemangleHooks(mangled, null);
    d.mute = true; // no demangled output
    d.parseMangledName();
    return d.hooks.result;
}
```
### Is the new mangling sound?


The back references encoded into the mangling extend the existing mangling. Unfortunately, the latter had ambiguities reported to [the D issue tracking system](https://issues.dlang.org/), with more of these likely yet to be uncovered. The demangler in `core.demangle` rejected about 3% of the unmodified symbols from the Phobos unit tests, while 15% were demangled only partially.

It’s tough to verify the soundness of an addition to an already complex and fragile definition, as a change to the mangling would need an update to the tooling (demangler, debuggers). Anyway, it was a good opportunity to get rid of these, too.

So scrutiny of the existing definition was required. To do this mechanically, the mangling specification from the web site was converted into a grammar digestible by [the bison parser generator](https://www.gnu.org/software/bison/). Bison can create LALR(1) parser tables, which basically means that, while scanning a mangled symbol, looking at a character and its successor is enough to determine whether the character adds to a previous entity or starts a new one. When conflicts are reported when processing a grammar, they might be resolvable with a larger context, but they can also hint at actual problems or undesirable complexity. Adding pseudo-tokens representing handcrafted parser control flow can avoid these conflicts.

[This gist](https://gist.githubusercontent.com/rainers/6cdf73b48837defb9f88/raw/3db6a3c8e72222ccd6b22e8ae01c601bd585e9d8/dwebsite4.bison) shows a grammar for the D mangling scheme without the back references. It still has a couple of conflicts when run through Bison, one of which was determined to be an actual [ambiguity in the definition](https://github.com/dlang/dmd/pull/6702). Adding back references to
the grammar [doesn’t add any conflicts](https://gist.githubusercontent.com/rainers/6cdf73b48837defb9f88/raw/3db6a3c8e72222ccd6b22e8ae01c601bd585e9d8/backref6.bison).

In addition, [`core.demangle`](https://github.com/dlang/druntime/blob/master/src/core/demangle.d) was fixed to work for all symbols but those exposing the known ambiguities.


### Aftermath


Some of the implementations in `std.traits` used the mangling of a symbol to introspect compile-time properties, for example, to determine the linkage. This was done using a simplified demangler. With the introduction of back references, these
didn’t work any more except for simple symbol names. Using a solution as for `core.mangleFunc` is feasible, but can slow down compilation considerably as the demangling needs to be executed via CTFE. Fortunately, new `__traits` have been added which cover all information that can be found in the mangling.

While most users will not notice any changes to their programs other than smaller object and executable file sizes, the new mangling can be a breaking change to external tools like the linker or a debugger. These will continue to work, but until they are updated, be prepared to eventually see the new mangled names for a little while instead of the demangled ones.

Thanks to Mike Parker, Walter Bright and Steven Schveighoffer for review.
