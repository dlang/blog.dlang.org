---
author: WalterBright
comments: false
date: 2023-10-02 17:05:47+00:00
layout: post
link: https://dlang.org/blog/2023/10/02/crafting-self-evident-code-with-d/
slug: crafting-self-evident-code-with-d
title: Crafting Self-Evident Code with D
wordpress_id: 3142
categories:
- Code
- Tutorials
permalink: /crafting-self-evident-code-with-d/
redirect_from: /2023/10/02/crafting-self-evident-code-with-d/
---

![Digital Mars D logo]({{ '/assets/images/crafting-self-evident-code-with-d/d6.png' | relative_url }})

Have you ever looked at your code from five years ago and had to study it to figure out what it was doing? And the further back in time you look, the worse it gets? Pity me, who is still maintaining code I wrote over 40 years ago. This article illustrates many simple methods of making your code self-evident and much easier to understand and maintain





To let you know what you’re fightin’ for, allow me to introduce this little gem I wrote back in 1987:




```c
#include <stdio.h>
#define O1O printf
#define OlO putchar
#define O10 exit
#define Ol0 strlen
#define QLQ fopen
#define OlQ fgetc
#define O1Q abs
#define QO0 for
typedef char lOL;

lOL*QI[] = {"Use:\012\011dump file\012","Unable to open file '\x25s'\012",
  "\012","   ",""};

main(I,Il)
lOL*Il[];
{       FILE *L;
         unsigned lO;
         int Q,OL[' '^'0'],llO = EOF,

         O=1,l=0,lll=O+O+O+l,OQ=056;
         lOL*llL="%2x ";
         (I != 1<<1&&(O1O(QI[0]),O10(1011-1010))),
         ((L = QLQ(Il[O],"r"))==0&&(O1O(QI[O],Il[O]),O10(O)));
         lO = I-(O<<l<<O);
         while (L-l,1)
         {       QO0(Q = 0L;((Q &~(0x10-O))== l);
                         OL[Q++] = OlQ(L));
                 if (OL[0]==llO) break;
                 O1O("\0454x: ",lO);
                 if (I == (1<<1))
                 {       QO0(Q=Ol0(QI[O<<O<<1]);Q<Ol0(QI[0]);
                         Q++)O1O((OL[Q]!=llO)?llL:QI[lll],OL[Q]);/*"
                         O10(QI[1O])*/
                         O1O(QI[lll]);{}
                 }
                 QO0 (Q=0L;Q<1<<1<<1<<1<<1;Q+=Q<0100)
                 {       (OL[Q]!=llO)? /* 0010 10lOQ 000LQL */
                         ((D(OL[Q])==0&&(*(OL+O1Q(Q-l))=OQ)),
                         OlO(OL[Q])):
                         OlO(1<<(1<<1<<1)<<1);
                 }
                 O1O(QI[01^10^9]);
                 lO+=Q+0+l;}
         }
         D(l) { return l>=' '&&l<='\~';
}
```
Yes, this is how we wrote C code back then. I even [won an award](https://www.ioccc.org/winners.html) for it!





Although I am a very slow learner, I do learn over time, and gradually the code got better. You’re probably having the same issues with your code. (Or the code written by coworkers, as I agree that _your_ code certainly does not need improvement!)





This article is about techniques that will help make code self-evident. You are probably already doing some of them. I bet there are some you aren't. I also am sure you’re going to argue with me about some of them. Trust me, you’re wrong! If you don’t agree with me now, you will if you’re still programming five years hence.





I know you’re busy, so let’s jump right in with an observation:





“Anybody can write complicated code. It takes genius to write simple code.”





or, if you prefer:





“The highest accolade your code can garner is: oh pshaw, anybody could have
written that!”





For example, since I started as an aerospace engineer:



![]({{ '/assets/images/crafting-self-evident-code-with-d/lever.jpg' | relative_url }})



consider this lever commonly found in aircraft cockpits. No fair if you already know what it does. Examine it casually. What is it for?





.





.





.





It raises and lowers the landing gear. What’s the clue? It’s got a little tire for a knob! Pull the lever up, and the gear gets sucked up. Push it down, the gear goes down. Is that self-evident or what? It’s a masterpiece of simplicity. It doesn’t even need any labels. If the cockpit is filled with smoke, or you're focused on what's outside the window, your hand knows immediately it’s on the gear lever--not the flaps or the throttles or the copilot’s ejection seat (just kidding). This kind of stupid simple control is what cockpit designers strive for because pulling the right lever is literally a life-and-death decision. I mean _literally_ in the literal sense of the word.





This is what we desperately want to achieve in programming. Stupid simple. We’ll probably fail, but the closer the better.





Diving in…





### Just Shoot Me Now




```c
#define BEGIN {
#define END   }
```
Believe it or not, this was common C practice back in the 1980s. It falls into the category of "Don’t try to make your new language look like your previous language". This problem appears in multiple guises. I still tend to name variables in Fortran style from back when the oceans hadn’t yet formed. Before moving to D, I realized that using C macros to invent a personal custom language on top of C was an abomination. Removing it and replacing it with ordinary C code was a big improvement in clarity.





### Don’t Reinvent bool





Learn what bool is and use it as intended. Accept that the following are all the same:


<table rows="5" cols="2" >
<tr >
<td >false
</td>
<td >true
</td></tr>
<tr >
<td >0
</td>
<td >1
</td></tr>
<tr >
<td >no
</td>
<td >yes
</td></tr>
<tr >
<td >off
</td>
<td >on
</td></tr>
<tr >
<td >0 volts
</td>
<td >5 volts
</td></tr>
</table>



And that this makes code unequivocally worse:




```d
enum { No, Yes };
```
Just use `false` and `true`. Done. And BTW,




```d
enum { Yes, No };
```
is just an automatic "no hire" decision, as `if (Yes)` will thoroughly confuse everyone. If you've done this, run and fix it before someone curses your entire ancestry.





### Horrors Blocked by D





D's syntax has been designed to prevent some types of coding horrors.





#### C++ Regex expressions with operator overloading





I’m not even going to link to this. It can be found with diligent searching. What it does is use operator overloading to make ordinary-looking C++ code actually be a regex! It violates the principle that code should not pretend to be in another language. Imagine the inept error messages the compiler will bless you with if there's a coding mistake with this. D makes this hard to do by only allowing arithmetic operators to be overloaded, which disallows things like an overloaded unary `*`.





(It’s harder, but still possible, to abuse operator overloading in D. But fortunately, making it harder has largely discouraged it.)





#### Metaprogramming with macros





Many people have requested macros be added to D. We've resisted this because macros inevitably result in people inventing their own custom, undocumented language layered over D. This makes it impractical for anyone else to make use of this code. In my not-so-humble opinion, macros are the reason why Lisp has never caught on in the mainstream. No Lisper can read anyone else's Lisp code.





#### C++ Argument Dependent Lookup





Nobody knows what symbol will actually be found. ADL was added so one could do operator overloading on the left operand. D just has a simple syntax for left or right operand overloading.





#### SFINAE





Nobody knows if SFINAE is in play or not for any particular expression.





#### Floor Wax or Tasty Dessert Topping





This refers to the confusion between a struct being a value type or a reference type or some chimera of both. In D, a struct is a value type and a class is a reference type. To be fair, some people still try to build a D chimera type, but they should be cashiered.





#### Multiple inheritance





Nobody has ever made a convincing case for why this is needed. Things get really nasty when diamond inheritance is used. Pity the next guy and avoid the temptation. D has multiple inheritance for interfaces only, which has proved to be more than adequate.





### Code Flow





Code should flow from left to right, and top to bottom. Just like how this article is read.




```d
f() + g() // which executes first?
```
Fortunately, D guarantees a left-to-right ordering (C does not). But what about:




```d
g(f(e(d(c(b(a))),3)));
```
That executes inside out! Quick, which function call does the `3` get passed to? D’s Universal Function Call Syntax to the rescue:




```d
a.b.c.d(3).e.f.g;
```
That's the equivalent, but execution flows clearly left-to-right. Is this an extreme example, or the norm?




```d
import std.stdio;
import std.array;
import std.algorithm;

void main() {
     stdin.byLine(KeepTerminator.yes).
     map!(a => a.idup).
     array.
     sort.
     copy(stdout.lockingTextWriter());
}
```
This code reads from `stdin` by lines, puts the lines in an array, sorts the array, and writes the sorted result to `stdout`. It doesn’t quite meet our "stupid simple" criteria, but it is pretty close. All with a nice flow from left to right and top to bottom.





The example also nicely segues into the next observation.





### The More Control Paths, the Less Understandable







> 
> Shaw: You know a great deal about computers, don’t you?  
> Mr. Spock: I know all about them.
> 
> 






I submit that:



```d
version (X)
     doX();
doY();
if (Z)
     doZ();
 ```
is less comprehensible than:



```d
doX();
doY();
doZ();
```
What happened to the conditional expressions? Move them to the interiors of `doX()` and `doZ()`.





I know what you’re thinking. "But Walter, you didn’t eliminate the conditional expressions, you just moved them!" Quite right, but those conditional expressions properly belong in the functions, rather than enclosing those functions. They are part of the conceptual encapsulation a function provides, so the caller is clean.





### Negation





Negation in English:








> Dr McCoy: We’re trying to help you, Oxmyx.  
> Bela Oxmyx: Nobody helps nobody but himself.  
> Mr. Spock: Sir, you are employing a double negative.  





> Cowardly Lion: Not nobody! Not nohow!







Negation in English is often used as emphasis, rather than logical negation. Our perception of negation is fuzzy and fraught with error. This is something propagandists use to smear someone.





What the propagandist says: "Bob is not a drunkard!"





What the audience hears: "Bob is a drunkard!"





Skilled communicators avoid negation. Savvy programmers do, too. How many times have you overlooked a not operator? I have many times.




```d
if (!noWay)
```
is inevitably perceived as:




```d
if (noWay)
```
I mentioned this discovery to my good friend Andrei Alexandrescu. He didn’t buy it. He said I needed research to back it up. I didn’t have any research, but didn’t change my mind (i.e., hubris). Eventually, I did run across a paper that did do such research and came to the same conclusion as my assumption. I excitedly sent it to Andrei, and to his great credit, he conceded defeat, which is why Andrei is an exceptional man (rare is the person who ever concedes defeat!).





The lesson here is to avoid using negation in identifiers if at all possible.




```d
if (way)
```
Isn't that better?





#### DMD Source Code Hall of Shame





My own code is hardly a paragon of virtue. Some identifiers:







  * `tf.isnothrow`


  * `IsTypeNoreturn`


  * `Noaccesscheck`


  * `Ignoresymbolvisibility`


  * `Include.notComputed`


  * `not nothrow`





I have no excuse and shall have myself flagellated with a damp cauliflower. Did I say I didn’t like the code I wrote five years ago?





This leads us to the D `version` conditional.





### Negation and `version`





[D version conditionals](https://dlang.org/spec/version.html#version-specification) are very simple:




```d
version ( Identifier )
```
_Identifier_ is usually predefined by the compiler or the command line. Only an identifier is allowed--no negation, AND, OR, or XOR. (Collectively call that _version algebra_.) Our users often chafe at this restriction, and I get that it's difficult to accept the rationale at first. It’s not impossible to do version algebra:




```d
version (A) { } else {
    // !A
}

version (A) version (B) {
    // A && B
}

version (A) version = AorB;
version (B) version = AorB;
version (AorB) {
     // A || B
}
```
and so forth. But it’s clumsy and unattractive _on purpose_. Why would D do such a thing? It’s meant to encourage thinking about versions in a positive manner. Suppose a project has a Windows and an OSX build:




```d
version (Windows) {
     ...
}
else version (OSX) {
     ...
}
else
     static assert(0, "unsupported operating system");
```
Isn’t that better than this:




```d
...
version (!Windows){
...
}
```
I’ve seen an awful lot of that style in C. It makes it pointlessly difficult to add support for a new operating system. After all, what the heck is the "not Windows" operating system? That really narrows things down! The former snippet makes it so much easier.





Taking this a step further:



```d
if (A && B && C && D)
    
if (A || B || C || D)
```
are easy for a human to parse. Encountering:




```d
if (A && (!B || C))
```
is always like transitioning from smooth asphalt to a cobblestone road. Ugh. I've made mistakes with such constructions all the time. Not only is it hard to even see the `!`, but it’s still hard to satisfy yourself that it is correct.





Fortunately, De Morgan’s Theorem can sometimes come to the rescue:



```d
(!A && !B) => !(A || B)
(!A || !B) => !(A && B)
```
It gets rid of one negation. Repeated application can often transform it into a much more easily understood equation while being equally correct.





Anecdote: When designing digital logic circuits, the NAND gate is more efficient than the AND gate because it has one less transistor. (AND means (A && B), NAND means !(A && B)). But humans just stink at crafting bug-free NAND logic. When I worked on the design of the [ABEL programming language](https://en.wikipedia.org/wiki/Advanced_Boolean_Expression_Language) back in the 1980s, which was for programming Programmable Logic Devices, ABEL would accept input in positive logic. It would use De Morgan’s theorem to automatically convert it to efficient negative logic. The electronics designers loved it.





To sum up this section, here’s a shameful snippet from Ubuntu’s unistd.h:




```c
#if defined __USE_BSD || (defined __USE_XOPEN && !defined __USE_UNIX98)
```
> 
> Prof Marvel: I can’t bring it back, I don’t know how it works!
> 
> 






### Casts Are Bugs





Casts subvert the protections of the typing system. Sometimes you just gotta have them (to implement `malloc`, for example, the result needs a cast), but far too often they are simply there to correct sloppy misuse of types. Hence, in D casts are done with the keyword `cast`, not a peculiar syntax, making them easily greppable. It's worthwhile to occasionally grep a code base for `cast` and see if the types can be reworked to eliminate the need for the cast and have the type system working for rather than against you.





Pull Request: [remove some dyncast calls](https://github.com/dlang/dmd/pull/15488)





### Self-Documenting Function Declarations




```d
char* xyzzy(char* p)
```
  * Does `p` modify what it points to?


  * Is `p` returned?


  * Does `xyzzy` free `p`?


  * Does `xyzzy` save `p` somewhere, like in a global?


  * Does `xyzzy` throw `p`?





These crucial bits of information are rarely noted in the documentation for the function. Even worse, the documentation often gets it wrong! What is needed is self-documenting code that is enforced by the compiler. D has attributes to cover this:




```c
const char* xyzzy(return scope const char* p)
```
  * `p` doesn’t modify what it points to


  * `p` is returned


  * `p` is not free’d


  * `xyzzy` does not squirrel away a copy of `p`


  * `p` is not thrown in an exception





This is all documentation that now isn't necessary to write, and the compiler will check its accuracy for you. Yes, it is called "attribute soup" for good reason, and takes some getting used to, but it's still better than bad documentation, and adding attributes is optional.





### Function Arguments and Returns





Function inputs and outputs present in the function declaration are the "front door". Any inputs and outputs that are not in the function declaration are "side doors". Side doors include things like global variables, environment variables, getting information from the operating system, reading/writing files, throwing exceptions, etc. Side doors are rarely accounted for in the documentation. The poor sap calling a function has to carefully read its implementation to discern what the side doors are.





Self-evident code should strive to run everything through the front door. Not only does this help with comprehension, but it also enables delightful things like easy unit testing.





### Memory Allocation





An ongoing problem faced by functions that implement an algorithm that needs to allocate memory is what memory allocation scheme should be used. Often a reusable function imposes the memory allocation method on the caller. That's backward.





For memory that is allocated and free'd by the function, the solution is that the function decides how to do it. For allocated objects that are returned by the function, the caller should decide the allocation scheme by passing an argument that specifies it. This argument often takes the form of a "sink" to send the output to. More on that later.





### Pass Abstract “sink” for Output





The auld way (extracted from the DMD source code):




```d
import dmd.errors;
void gendocfile(Module m) {
     ...
     if (!success)
         error("expansion limit");
}
```
`error()` is a function that error messages are sent to. This is a typical formulation seen in conventional code. The error message is going out through the side door. The caller of `gendocfile()` has no say in what's done with the error message, and the fact that it even generates error messages is usually omitted by the documentation. Worse, the error message emission makes it impractical to properly unit test the function.





A better way is to pass an abstract interface "sink" as a parameter and send the error messages to the sink:




```d
import dmd.errorsink;
void gendocfile(Module m, ErrorSink eSink) {
     ...
     if (!success)
         eSink.error("expansion limit");
}
```
Now the caller has total control of what happens to the error messages, and it is implicitly documented. A unit tester can provide a special implementation of the interface to suit testing convenience.





Here’s a real-world PR making this improvement:





[doc.d: use errorSink](https://github.com/dlang/dmd/pull/15471)





### Pass Files as Buffers Rather than Files to Read





Typical code I've written, where the file names are passed to a function to read and process them:




```d
void gendocfile(Module m, const(char)*[] docfiles) {
     OutBuffer mbuf;
     foreach (file; ddocfiles) {
         auto buffer = readFile(file.toDString());
         mbuf.write(buffer.data);
     }
     ...
}
```
This kind of code is a nuisance to unit test, as adding file I/O to the unit tester is very clumsy, and, as a result, no unit tests get written. Doing file I/O is usually irrelevant to the function, anyway. It just needs the _data_ to operate on.





The fix is to pass the contents of the file in an array:




```d
void gendocfile(Module m, const char[] ddoctext) {
     ...
}
```
The PR: [move ddoc file reads out of doc.d](https://github.com/dlang/dmd/pull/15525)





### Write to Buffer, Caller Writes File





A typical function that processes data and writes the result to a file:




```d
void gendocfile(Module m) {
     OutBuffer buf;
     ... fill buf ...
     writeFile(m.loc, m.docfile.toString(), buf[ ]);
}
```
By now, you know that the caller should write the file:




```d
void gendocfile(Module m, ref OutBuffer outbuf) {
     ... fill outbuf ...
}
```
And the PR:
[doc.d: move file writes to caller](https://github.com/dlang/dmd/pull/15535)





### Move Environment Calls to Caller





Here’s a function that obtains input from the environment:




```d
void gendocfile(Module m) {
     char* p = getenv("DDOCFILE");
     if (p)
         global.params.ddoc.files.shift(p);
}
```
It should be pretty obvious by now what is wrong with that. PR to move the environment read to the caller and then pass the info through the front door:





[move DDOCFILE from doc.d to main.d](https://github.com/dlang/dmd/pull/15503)





### Use Pointers to Functions (or Templates)





I was recently working on a module that did text processing. One thing it needed to do was identify the start of an identifier string. Since Unicode is complicated, it imported the (rather substantial) module that handled Unicode. But it bugged me that all that was needed was to determine the start of an identifier; the text processor needed no further knowledge of Unicode.





It finally occurred to me that the caller could just pass a function pointer as an argument to the text processor, and the text processor would need no knowledge whatsoever of Unicode.




```d
import dmd.doc;
bool expand(...) {
     if (isIDStart(p))
         ...
}
```
became:




```d
alias fp_t = bool function(const(char)* p);
bool expand(..., fp_t isIDStart) {
     if (isIDStart(p))
         ...
}
```
Notice how the import just went away, improving the encapsulation and comprehensibility of the function. The function pointer could also be a template parameter, whichever is more convenient for the application. The more moats one can erect around a function, the easier it is to understand.





The PR: [remove dmacro.d dependency on doc.d](https://github.com/dlang/dmd/pull/15470)





### Two Categories of Functions







  * Alters the state of the program





Provide a clue in the name of the function, like `doAction()`.







  * Asks a Question





Again, a clue should be in the name. Something like `isSomething()`, `hasCharacteristic()`, `getInfo()`, etc. Consider making the function `pure` to ensure it has no side effects.





Try not to create functions that both ask a question and modify state. Over time, I've been gradually splitting such functions into two.





### Visual Pattern Recognition





Source code formatters are great. But I don’t use them. Oof! Here’s why:




```d
final switch (of)
{
     case elf:   lib = LibElf_factory();    break;
     case macho: lib = LibMach_factory();   break;
     case coff:  lib = LibMSCoff_factory(); break;
     case omf:   lib = LibOMF_factory();    break;
}
```
It turns out your brain is incredibly good at pattern recognition. By lining things up, a pattern is created. Any deviation from that pattern is likely a bug, and your eyes will be drawn to the anomaly like a stink bug to rotting fruit.





I’ve detected so much rotting fruit by using patterns, and a source code formatter doesn’t do a good job of making patterns.







> 
> Prof. Marvel: I have reached a cataclysmic decision!
> 
> 






### Use `ref` instead of `*`





A `ref` is a restricted form of pointer. Arithmetic is not allowed on it, and `ref` parameters are not allowed to escape a function. This not only informs the user but _informs the compiler_, which will ensure the function is a good boy with the `ref`.





PR: [mangleToBuffer(): use ref](https://github.com/dlang/dmd/pull/15487)





## Takeaways







  * Use language features as intended (don’t invent your own language
on top of it)


  * Avoid negation


  * Left to right, top to bottom


  * Functions do everything through the front door


  * Don’t conflate engine with environment


  * Reduce cyclomatic complexity


  * Separate functions that ask a question from functions that alter state


  * Keep trying--this is a process!





The recommendations here are pretty easy to follow. It'll rarely be necessary to do much refactoring to implement them. I hope the real-life PRs referenced here show how easy it is to make code self-evident!





## Action Item





Open your latest coding masterpiece in your favorite editor. Take a good hard look at it. Sorry, it’s a steaming pile of incomprehensibility! (Join the club.)





Go fix it!
