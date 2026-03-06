---
author: BasVeel
comments: false
date: 2018-06-20 13:13:23+00:00
excerpt: Imagine there is this little-known programming language in which you enjoy
  programming in your free time. You know it is ready for prime time and you dream
  about using it at work everyday. This is the story about how I made a dream like
  that come true.
layout: post
link: https://dlang.org/blog/2018/06/20/how-an-engineering-company-chose-to-migrate-to-d/
slug: how-an-engineering-company-chose-to-migrate-to-d
title: How an Engineering Company Chose to Migrate to D
wordpress_id: 1604
categories:
- Code
- Companies
- Compilers &amp; Tools
- Guest Posts
- User Stories
permalink: /how-an-engineering-company-chose-to-migrate-to-d/
redirect_from: /2018/06/20/how-an-engineering-company-chose-to-migrate-to-d/
---

_Bastiaan Veelo is the lead developer of a specialised program for the
computer aided geometric design of ship hulls called Fairway, for the
company SARC in the Netherlands._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)Imagine there is this little-known programming language in which you enjoy programming in your free time. You know it is ready for prime time and you dream about using it at work everyday. This is the story about how I made a dream like that come true.


## My early acquaintance with D


Back when “google” was not yet a common verb, I was doing a web search for “parsing C++”. The reason was that writing a report for an assignment had derailed into writing a syntax highlighter [for noweb](https://en.wikipedia.org/wiki/Noweb) [using bison](https://en.wikipedia.org/wiki/GNU_bison) [and flex](https://en.wikipedia.org/wiki/Flex_(lexical_analyser_generator)), and I found out firsthand that C++ is not easy to parse. That web search [brought up this page](https://web.archive.org/web/20011005113647/http://www.digitalmars.com:80/d/overview.html) ([present version](https://dlang.org/overview.html#for_d)) with an overview of the D Programming Language, and the following statement has had me hooked ever since:


D’s lexical analyzer and parser are totally independent of each other and of the semantic analyzer. This means it is easy to write simple tools to manipulate D source perfectly without having to build a full compiler. It also means that source code can be transmitted in tokenized form for specialized applications.


“Genius,” I thought, “here we have someone who knows what he’s doing.” This is representative of the pragmatic professionalism that still radiates from the D community, and it combines with an unpretentious flair that makes it pleasant to be around. This funny quote [decorated its homepage](https://web.archive.org/web/20011015103847/http://www.digitalmars.com:80/d/) for many years:


“Great, just what I need.. another D in programming.” – Segfault


Nevertheless, I didn’t have many opportunities to use the language and I largely remained sitting on the fence, observing its development.


## Programming professionally


With mostly academic programming experience, I started programming professionally in 2006 for [SARC, a Dutch engineering company](https://www.sarc.nl/) serving the maritime industry. Since the early ’80s they have been developing software for ship design and onboard loading calculations, which today amounts to roughly half a million lines of code. I think their success can partly be attributed to their choice of programming language: Extended Pascal ([the ISO 10206 standard](http://pascal-central.com/docs/iso10206.pdf), not one of the many proprietary extensions of Pascal).

Extended Pascal was a great improvement over ISO 7185 Pascal. Its compiler, by Prospero Software from England, was fast and well documented. The language is small enough and its syntax appropriately verbose to make engineering professionals quickly productive in programming. Personally though, I spent most of my time programming in C++, modernizing their [system for computer aided design of ship hulls](https://www.sarc.nl/fairway/) [using Qt](https://www.qt.io/) [and Coin3D](https://bitbucket.org/Coin3D/coin/wiki/Home).


## When your company outlives a programming language


Although selecting an ISO standard in favor of a proprietary Pascal dialect seemed wise at the time, it is apparent now that the company has outlived the language. Prospero Development Software Ltd was officially dissolved 15 years ago. Still, its former director, Tony Hetherington, continued giving support many years after, but he’d be close to 86 years old now and can no longer be reached. Its website is gone, [last archived in 2013](https://web.archive.org/web/20131023234615/http://www.prosperosoftware.com:80/). [There’s GNU Pascal](http://www.gnu-pascal.de/gpc/h-index.html), which also supports ISO 10206, but that project has stopped moving and long ago lost synchrony with gcc. Although there is no immediate crisis, it is clear that something needs to happen sometime if the company wants to continue its activities in the coming decades.


## Changing the odds


A couple of years ago, I secretly started playing with the fantasy of replacing Extended Pascal with D. Even though D’s syntax is somewhat different from Pascal, it shares at least four important similarities: support for nested functions, boundary checking, modules, and compilation speed. In addition, it has many traits that make the language attractive to engineers: good focus on performance and numerics, garbage collection, dynamic arrays, easy parallelization, understandable templates, contract programming, memory safety, unit tests, and even wysiwyg strings and formatted numerals. D’s language features encourage experimentation, which resonates well with engineers.

So I wondered what I could do to highlight D’s significance to my employer and show it’s an attractive language to switch to. I thought I could make a compelling case if I could write a parser in D that would take Extended Pascal source and transpile it to D source. At least I would have fun trying!

So I went over to [code.dlang.org](https://code.dlang.org/) to see if there were any D alternatives to flex and bison. There, [I found Pegged](https://code.dlang.org/packages/pegged), and instantly the fun began. Pegged combines the functionality of flex and bison in one incredibly easy to use package, for which its creator Philippe Sigaud obviously [enjoyed writing excellent documentation](https://github.com/PhilippeSigaud/Pegged/wiki/Pegged-Tutorial). Nowadays, Pegged is part of [the D language tour](https://tour.dlang.org/) and you can [try it out on-line](https://tour.dlang.org/tour/en/dub/pegged) without having to install a thing. The beauty is that the grammar from the Extended Pascal [language specification](http://pascal-central.com/docs/iso10206.pdf) maps almost linearly to [the PEG from which](https://github.com/veelo/Pascal2D/blob/master/source/epgrammar.d) Pegged generates the parser. For this it makes heavy use of D’s generic programming capabilities and compile-time function evaluation — it can generate a parser at compile time if you want it to!

However, it wasn’t smooth sailing all along. As I was testing D, I suddenly found _myself_ being tested as well. I learned the hard way that there is [a phenomenon called left-recursion](https://en.wikipedia.org/wiki/Left_recursion), from which a PEG parser typically cannot break out of. And the Extended Pascal grammar is left-recursive in several ways. Consequently, I spent many evenings and weekends researching parsing theory, until eventually I managed to extend Pegged with [support for all kinds of left-recursion](https://github.com/PhilippeSigaud/Pegged/wiki/Left-Recursion)! From one thing came another, and I added [longest match alternation](https://github.com/PhilippeSigaud/Pegged/wiki/Extended-PEG-Syntax#longest-match-alternation), [case insensitive literals](https://github.com/PhilippeSigaud/Pegged/wiki/Grammar-Debugging), the [`toHTML()` method](https://github.com/PhilippeSigaud/Pegged/wiki/Parse-Result) for dynamically [browsing the syntax tree](https://cdn.rawgit.com/PhilippeSigaud/Pegged/ade2aa5d/pegged/examples/extended_pascal/example.html), and a tracer for [logging the parsing process](https://github.com/PhilippeSigaud/Pegged/wiki/Grammar-Debugging).

Obviously, I was having fun. But more importantly, I was demonstrating that the D programming language is accessible enough that a naval architect can understand other people’s code and expand it in non-trivial ways. The icing on the cake came when I was asked to present my experiences at [DConf 2017 in Berlin](http://dconf.org/2017/talks/veelo.html), which [you can watch here](https://youtu.be/t5y9dVMdI7I) (and [here’s the extra bit I presented at lunch time](https://youtu.be/3ugQ1FFGkLY) for the livestream audience).

At this time, I was able to automatically translate the following trivial example:

```d
program hello(output);

begin
    writeln('Hello D''s "World"!');
end.
```
into D:

```d
import std.stdio;

// Program name: hello
void main(string[] args)
{
    writeln("Hello D's \"World\"!");
}
```
# Language competition


Having come this far, [the founder of SARC](https://www.sarc.nl/) agreed that it was time to investigate the merits of various alternative programming languages. We would do a thorough and objective comparison based on trial translations of a comprehensive set of language features. Due to the amount of manual labor that this requires, we had to drastically prune the space of programming languages in an initial review round. Note that what I am about to present does _not_ declare which programming language is the best in our industry. What _we_ are looking for is a language that allows an efficient transition from Extended Pascal without interrupting our business, and which enables us to take advantage of modern insights and tools.

In the initial review round we looked at general language characteristics. Here I’ll just highlight what fell through the sieve and why.

Performance is important to us, which is why we did not consider interpreted languages. C++ is in use for one component of our software, but that was written from the ground up. We feel that the [options for translation](http://bulletin.iis.nsk.su/files/article/markin.pdf) are not favorable, that its long compile times are a serious hindrance to productivity, and that there are too many ways in which one can shoot one’s self in the foot. We cannot require our expert naval architects to also become experts in C++.

Nowadays, whenever D is publicly evaluated, the younger languages Go and Rust are often brought up as alternatives. Here, we need not go into an in-depth comparison of these languages because both Rust and Go lack one feature that we rely on heavily: nested functions with access to variables in their enclosing scope. Solutions for eliminating nested functions, like bringing them into global scope and passing extra variables, or breaking files up into smaller modules, we find unattractive because it would complicate automated translation, and we’d like to preserve the structure and style of our current code. [GNU C does offer nested functions](https://gcc.gnu.org/onlinedocs/gcc/Nested-Functions.html), but it is a non-standard extension and it has been predicted that many will [move away from C due to its unsafe features](https://www.youtube.com/watch?v=Lo6Q2vB9AAg&feature=youtu.be&t=24m37s). After this initial pruning, three languages remained on our shortlist: **Free Pascal**, **Ada** and **D**.

As a basis for our detailed comparison, we wrote fifteen small programs that each used a specific feature of Extended Pascal that is important in our current code base. We then translated those programs into each language on our shortlist. We kept a simple score board on how well these features were represented in each language: +1 if the feature is supported or can be implemented, 0 if the lack of the feature can be worked around, and -1 if it can’t. This is what came out of that evaluation:
<table > 

<tr >
Test
Free Pascal
Ada
D
</tr>

<tbody >
<tr >

<td >Arrays beginning at arbitrary indices
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Sets
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Schema types
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Types with custom initial values
</td>

<td style="text-align: center;" >-1
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Classes
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Casts
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Protection against use of dangling pointers
</td>

<td style="text-align: center;" >-1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Thread safe memory [de]allocation
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Calling into Windows API
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Forwarding Windows callbacks to nested functions
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Speed of calculations
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Calling procedures written in assembly
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Calling procedures in a DLL
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Binary compatibility of strings
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >+1
</td>

<td style="text-align: center;" >+1
</td>
</tr>
<tr >

<td >Binary compatible file i/o
</td>

<td style="text-align: center;" >-1
</td>

<td style="text-align: center;" >0
</td>

<td style="text-align: center;" >0
</td>
</tr>
<tr >

<td >**Score**
</td>

<td style="text-align: center;" >**6**
</td>

<td style="text-align: center;" >**10**
</td>

<td style="text-align: center;" >**14**
</td>
</tr>
</tbody>
</table>
So, Free Pascal is the only candidate with negative scores, Ada positions itself in the middle, and D achieves an almost perfect score. Not effortlessly, though; we’ll talk about some of the technical challenges later. Because Free Pascal is, like D, fully Open Source and written in itself, extending the language and filling in the gaps is theoretically possible. Although some of its deficiencies could certainly be resolved that way, others would be quite complicated and/or unlikely to be accepted upstream.

We also estimated the productivity of the languages. Free Pascal scored high because it is closest to what we are used to. Despite its dissimilar syntax, D scored high because of its expressiveness and flexibility. Ada scored lowest because of its rigidity and because of the extra work the programmer has to put in (most importantly casts and conversions). Ada is more verbose than Pascal which was disliked by some of us because it can somewhat obscure the essence of what a piece of code tries to express, and frequently the code became not only verbose but cryptic, which was unanimously disliked.

Third, we estimated the future prospects and the advantages each language could bring to the table. Although Free Pascal has a more active community than we expected it to have, we do not see great potential for growth. Ada is renowned for its support for writing reliable code (although it has no monopoly in that field) but it does come at a cost and requires real effort. D has a dynamic and open community, supports both script-like productivity and high performance, includes various features for writing reliable software (approaching Ada but at a much lower cost), and offers some unique advanced features with which wonders can be accomplished.

Finally, we estimated the effort of translation. Although Free Pascal is very similar to Extended Pascal, missing features pose a real problem and would require a high degree of manual translation and rewriting. [Although `p2ada` exists](http://p2ada.sourceforge.net/), it only works partially in our case and does not fully support Extended Pascal. Because Ada frequently requires additional code (casting to the correct type, pulling in a package, instantiating a generic type, adding a pragma, splitting up `Put_Line`s etc.), writing or extending a reliable transpiler into Ada would be more difficult than doing the same into D.


# Selecting a winner


I gave away the winner in the title, but we landed at that conclusion as follows. Ada was the first language to be dropped. We really felt that the extra work that the programmer has to put in is a brake on productivity and creativity. Although it barely played a role in our evaluation, illustrative is the difference between the Ada and D equivalents to the [Expressive C++17 Challenge](https://www.fluentcpp.com/2017/09/25/expressive-cpp17-coding-challenge/). The [D solution](https://seb.wilzba.ch/b/2018/02/the-expressive-c17-coding-challenge-in-d/) is both concise and expressive, the [Ada solution](https://two-wrongs.com/expressive-ada-2012-challenge) is hardly expressive and consists of more lines than I want to write or read. Also of secondary importance, but difficult to ignore, is the difference between the communities surrounding the languages, which in Ada’s case is AdaCore Support, who has no problems demanding annual five-figure subscription fees.

Although akin to our current language, Free Pascal was mainly dropped due to its porting challenges and our estimation that its potential is lower and its future outlook is less optimistic than that of D. If we were to choose Free Pascal, we would basically invest a lot of effort only to arrive at a technological solution that we felt would be of lower quality than we currently have.

And that’s were I saw a dream come true: A clap on the table by the company founder and it was decided to commit to the effort of bringing twenty-five years worth of Extended Pascal code to D!


# What makes a difference


In short, my experience is that if a feature is not present in the language, D is powerful enough that the feature can be implemented in a library. Translating each sample program by hand has really helped to focus on replicating functionality, leaving the translation process for later concern. This has led to [writing a compatibility library](https://veelo.github.io/Pascal2D/) with types and functions that are vital for the conversion. Now that equivalents are known and the parser is done, I just have to implement code generation.

Below follows another example that currently translates automatically and executes identically. It iterates over a fixed length array running from `2` to `20` inclusive, fills it with values, prints the memory footprint and writes it to binary file:

```d
program arraybase(input,output);

type t = array[2..20] of integer;
var a : t;
    n : integer;
    f : bindable file of t;

begin
  for n := 2 to 20 do
    a[n] := n;
  writeln('Size of t in bytes is ',sizeof(a):1); { 76 }
  if openwrite(f,'array.dat') then
    begin
      write(f,a);
      close(f);
    end;
end.
```
Transpiled to D (or should I say Dascal?) and [post-processed by dfmt](https://code.dlang.org/packages/dfmt) to fix up formatting:

```d
import epcompat;
import std.stdio;

// Program name: arraybase
alias t = StaticArray!(int, 2, 20);

t a;
int n;
Bindable!t f;

void main(string[] args)
{
    for (n = 2; n <= 20; n++)
        a[n] = n;
    writeln("Size of t in bytes is ", a.sizeof); // 76
    if (openwrite(f, "array.dat"))
    {
        epcompat.write(f, a);
        close(f);
    }
}
```
Of course this is by no means idiomatic D, but the fact that it is recognizable and readable is nice, especially for my colleagues who will have to go through an unusual transition. By the way, did you notice that code comments are preserved?

One _very-nice-to-have_ feature is binary file compatibility; In fact it may have been the killer feature, without which D might not have been so victorious. The case is that whenever a persistent data structure is extended in our software, we make sure that we can still read and convert that structure from its prior format. That way, if a client pulls out an old design from its archives and runs it through our current software, it will still work without the user even being aware that conversion occurs, possibly in multiple steps. Not having to give up that ability is very attractive.

But it wasn’t easy to get there. The main difficulty is the difference in how strings are represented in D and the Prospero implementation of Extended Pascal, in memory and on file. This presented the challenge of how to preserve binary compatibility in file I/O with data structures that contain string members.


## Strings


In Prospero Extended Pascal, strings are implemented as a schema type, which is a parameterized type that can be used in the following ways:

    type string80 = string(80);
    var str1 : string80;
        str2 : string(60);
    procedure foo(s : string);
This defines `string80` to be an alias for a string type discriminated to have a capacity of 80 characters. Discriminated string variables, like `str1` and `str2`, can be passed to functions and procedures that take undiscriminated strings as arguments, like `foo`, which thereby work on strings of any capacity. In memory, `str1` is laid out as a sequence of 80 `char`s, followed by a `ushort` that encodes the length of the string. I say _encodes_ because a shorter string is padded with `\0`s up to the capacity and the `ushort` actually contains the length of that padding. This way, when a pointer to the string is passed to a C function and the contents of the string occupy its full capacity, the `0` in the padding length doubles as the terminating `\0` of the C string.

My first thought was to mimic this data representation with a D template. But that would require procedures like `foo` to be turned into templates as well, which would escalate horribly into template bloat, a problem with multiple string arguments and argument ordering, and would complicate translation. Besides, schema types can also be discriminated at run time, which does not translate to a template.

Could some sort of inheritance scheme be the solution? Not really, because instances of D classes live on the heap, so a string embedded in a `struct` would just be a pointer instead of the `char` array and `ushort`.

But binary layout is actually only relevant in files, and in a stroke of insight I realized that this must be why [user-defined attributes, or UDAs,](https://dlang.org/spec/attribute.html#uda) exist. If I annotate the string with the correct capacity for file I/O, then I can just use native D `string`s everywhere, which genuinely must be the best possible translation and solves the function argument issue. Annotation can be done with an instance of a `struct` like

```d
struct EPString
{
    ushort capacity;
}
```
The above Pascal snippet then translates to D like so:

```d
@EPString(80) struct string80 { string _; alias _ this; }
string80 str1;
@EPString(60) string str2;
void foo(string s);
```
Notice how the `string80` alias is translated into the slightly convoluted `struct` instead of a normal D `alias`, which would have looked like

    @EPString(80) alias string80 = string;
    </code>
Although that compiles, there is no way to retrieve the UDA in that case because plain `alias` does not introduce a symbol. Then `hasUDA!(typeof(str1), EPString)` would have been equivalent to `hasUDA!(string, EPString)` which evaluates to `false`. By using the `struct`, `string80` is a symbol so `typeof(str1)` gives `string80`, and `hasUDA!(string80, EPString)` evaluates to `true` in this example.

There is one side effect that we will have to learn to accept, and that is that taking a slice of a string does not produce the same result in D as it does in Extended Pascal. That is because string indices start at 1 in Extended Pascal and at 0 in D. My strategy is to eliminate slices from the source and replace them with a call to the standard `substr` function, which I can implement with index correction. Finding all string slices can be accomplished with a switch in the transpiler that makes it insert a `static if` to test if the slice is being taken on a `string`, and abort compilation if it is. (`Array`s are transpiled into a custom array type that handles slices and indices compatibly with Extended Pascal.)


## Binary compatible file I/O


Now, to write `struct`s to file and handle any embedded `@EPString()`-annotated strings specially, we can use compile-time introspection in an overload to `toFile` that acts on `struct`s as shown below. I have left out handling of aliased strings for clarity, as well as `shortstring`, which is a legacy string type with yet a different binary format.

```d
void toFile(S)(S s, File f) if (is(S == struct))
{
    import std.traits;
    static if (!hasIndirections!S)
        f.lockingBinaryWriter.put(s);
    else
        // TODO unions
        foreach(field; FieldNameTuple!S)
        {
            // If the member has itself a toFile method, call it.
            static if (hasMember!(typeof(__traits(getMember, s, field)), "toFile") &&
                       __traits(compiles, __traits(getMember, s, field).toFile(f)))
                __traits(getMember, s, field).toFile(f);
            // If the member is a struct, recurse.
            else static if (is(typeof(__traits(getMember, s, field)) == struct))
                toFile(__traits(getMember, s, field), f);
            // Treat strings specially.
            else static if (is(typeof(__traits(getMember, s, field)) == string))
            {
                // Look for a UDA on the member string.
                static if (hasUDA!(__traits(getMember, s, field), EPString))
                {
                    enum capacity = getUDAs!(__traits(getMember, s, field), EPString)[0].capacity;
                    static assert(capacity > 0);
                    writeAsEPString(__traits(getMember, s, field), capacity, f);
                }
                else static assert(false, `Need an @EPString(n) in front of ` ~ fullyQualifiedName!S ~ `.` ~ field );
            }
            // Just write other data members.
            else static if(!isFunction!(__traits(getMember, s, field)))
                f.lockingBinaryWriter.put(__traits(getMember, s, field));
        }
}
```
At the time of writing, I still have work to do for `union`s, which are used in the translation of variant records (including considering the use of one of the seven existing library solutions [1](https://dlang.org/phobos/std_variant.html#.Algebraic), [2](https://code.dlang.org/packages/tag), [3](https://code.dlang.org/packages/tagged_union), [4](https://code.dlang.org/packages/taggedalgebraic), [5](https://github.com/nordlow/phobos-next/blob/master/src/vary.d#L30), [6](https://code.dlang.org/packages/minivariant), [7](https://code.dlang.org/packages/sumtype)).

Currently, [detecting `union`s is a bit involved ](https://forum.dlang.org/post/zwpctoccawmkwfoqkoyf@forum.dlang.org). Also, there is a complication in the determination of the size of a union when the largest variant contains strings: the D version of that variant may _not_ be the largest because D `string`s are just slices. I’ll probably work around this by adding a dummy variant that is a fixed size array of bytes to force the size of the `union` to be compatible with Extended Pascal. This is the reason why D scored a mere `0` in file format compatibility. It is amazing what D allows you to do though, so I may be able to do all of that automatically and award D a perfect score retroactively. On the other hand, it is probably easiest to just add the dummy variant in the Pascal source at the few places where it matters and be done with it.


# The way forward


Obviously, this is long term planning. It has taken years to grow into D; it will possibly take a year, and probably longer, to migrate to D. Unless others turn up who are in the same boat as us (please contribute!) it’ll be me who has to row this ship to D-land and I still have my regular duties to attend to. My colleagues will continue to develop in Extended Pascal as usual, and once [my transpiler](https://github.com/veelo/Pascal2D) is able to translate all or almost all of it, we will make the switch to D overnight. From then on, we’ll be in it for the long run. We trust to be with D and D to be with us for decades to come!
