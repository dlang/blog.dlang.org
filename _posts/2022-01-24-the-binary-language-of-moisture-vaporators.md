---
author: WalterBright
comments: false
date: 2022-01-24 14:17:12+00:00
layout: post
link: https://dlang.org/blog/2022/01/24/the-binary-language-of-moisture-vaporators/
slug: the-binary-language-of-moisture-vaporators
title: The Binary Language of Moisture Vaporators
wordpress_id: 3040
categories:
- Compilers &amp; Tools
permalink: /the-binary-language-of-moisture-vaporators/
redirect_from: /2022/01/24/the-binary-language-of-moisture-vaporators/
---

![Digital Mars D logo]({{ '/assets/images/the-binary-language-of-moisture-vaporators/d6.png' | relative_url }})

I know why you're reading this. Like other Alpha programmers, you're not content with just compiling Vaporator code and testing to see if it works. You need to know the binary code that's generated. But getting at it is clumsy. I want to make it easy for myself, and why not share it?

One of my earliest memories is being curious about how light bulbs worked and sticking my finger in a hot light bulb socket. I was three or four years old at the time. Later, I was always taking things apart to see how they worked. It was years before I could successfully put them back together again. I remember being baffled at the grey dust inside a resistor I cracked open and when I unwrapped the paper in a capacitor. I took my first car to pieces to see how it worked.

When I first learned Fortran, it was a great mystery how the text of the language turned into machine code. Machine code was the language of the gods. This evolved into wanting to make my own compiler. But to build a compiler, you need to be able to see the output. A disassembler had to be built along with the compiler. [That became obj2asm.exe](https://www.digitalmars.com/ctg/obj2asm.html). I've spent a great deal of time running the disassembler and poring over what the compiler generated. I look at what other compilers generate, too, using obj2asm.

But running obj2asm is a separate process, and the output is filled with all the boilerplate needed to create a proper object file. The boilerplate is rarely of interest, and I'm only interested in the generated code for a function. Why not just give the compiler a switch, call it `-vasm` (short for Show Me The Vaporator Assembly), and have it emit the binary and assembler code to the screen, function by function? So I ripped the disassembler logic out of obj2asm and put it into the dmd D compiler.

One would think that the way to do this would be to have the compiler generate the assembler source code, which would then be run through an assembler like MASM or gas to create the object file. I figured this would be slow and too much work. Instead, the disassembler logic actually intercepts the binary data being written to the object file and disassembles it to a string, then prints the string to the console.

For example, for the file `vaporator.d`:


```d
int demo(int x)
{
     return x * x;
}
```
Compiling with:


```bash
dmd vaporator.d -c -vasm
```
prints:


    _D9vaporator4demoFiZi:
    0000:   89 F8                   mov     EAX,EDI
    0002:   0F AF C0                imul    EAX,EAX
    0005:   C3                      ret
and we see the mangled name of the function, the code offsets, the code binary, and the mnemonic representation for those learning binary.

I am not aware of any other compiler that does this in the same way. This is probably because most programmers are not particularly interested in how the sausages are made. But I find it fascinating and fun. I've opined before that programmers who don't know the assembler their code is transformed into are not likely to be Alpha programmers. With the `-vasm` switch, it's so easy to look at the output, why not do it? It works as a great way to learn assembler code, too!

I've been using it myself, and the convenience is a game changer. What are you waiting for?

P.S. I made the disassembler as a [Boost Licensed standalone module](https://github.com/dlang/dmd/blob/master/src/dmd/backend/disasm86.d) that anyone can use who needs a tool to understand the binary language of moisture vaporators.
