---
author: WalterBright
comments: false
date: 2018-02-07 13:07:26+00:00
excerpt: Do you ever get tired of bugs that are easy to make, hard to check for, often
  don’t show up in testing, and blast your kingdom once they are widely deployed?
  They cost you time and money again and again. If you were only a better programmer,
  these things wouldn’t happen, right?
layout: post
link: https://dlang.org/blog/2018/02/07/vanquish-forever-these-bugs-that-blasted-your-kingdom/
slug: vanquish-forever-these-bugs-that-blasted-your-kingdom
title: Vanquish Forever These Bugs That Blasted Your Kingdom
wordpress_id: 1350
categories:
- BetterC
- Code
- Core Team
permalink: /vanquish-forever-these-bugs-that-blasted-your-kingdom/
redirect_from: /2018/02/07/vanquish-forever-these-bugs-that-blasted-your-kingdom/
---

_[Walter Bright](http://walterbright.com/) is the BDFL of the D Programming Language and founder of [Digital Mars](http://digitalmars.com/). He has decades of experience implementing compilers and interpreters for multiple languages, including Zortech C++, the first native C++ compiler. He also created [Empire, the Wargame of the Century](http://www.classicempire.com/). This post is [the second in a series](https://dlang.org/blog/the-d-and-c-series/#betterC) about [D’s BetterC mode](https://dlang.org/blog/2017/08/23/d-as-a-better-c/)._



* * *



![](http://dlang.org/blog/wp-content/uploads/2018/02/bug.jpg)

Do you ever get tired of bugs that are easy to make, hard to check for, often don’t show up in testing, and [blast your kingdom](https://getyarn.io/yarn-clip/ac6765ca-f2c6-49ec-a1ba-a7e9b0f692bf) once they are widely deployed? They cost you time and money again and again. If you were only a better programmer, these things wouldn’t happen, right?

Maybe it’s not you at all. I’ll show how these bugs are not your fault - they’re the tools’ fault, and by improving the tools you’ll never have your kingdom blasted by them again.

And you won’t have to compromise, either.


### Array Overflow


Consider this conventional program to calculate the sum of an array:

```c
#include <stdio.h>

#define MAX 10

int sumArray(int* p) {
    int sum = 0;
    int i;
    for (i = 0; i <= MAX; ++i)
        sum += p[i];
    return sum;
}

int main() {
    static int values[MAX] = { 7,10,58,62,93,100,8,17,77,17 };
    printf("sum = %d\n", sumArray(values));
    return 0;
}
```
The program should print:

```python
sum = 449
```
And indeed it does, on my Ubuntu Linux system, with both `gcc` and `clang` and `-Wall`. I’m sure you already know what the bug is:

    for (i = 0; i <= MAX; ++i)
                  ^^
This is the classic “[fencepost problem](https://en.wikipedia.org/wiki/Off-by-one_error#Fencepost_error)”. It goes through the loop 11 times instead of 10. It should properly be:

    for (i = 0; i < MAX; ++i)
Note that even with the bug, the program still produced the correct result! On my system, anyway. So I wouldn’t have detected it. On the customer’s system, well, then it mysteriously fails, and I have a remote [heisenbug](https://en.wikipedia.org/wiki/Heisenbug). I’m already tensing up anticipating the time and money this is going to cost me.

It’s such a rotten bug that over the years I have reprogrammed my brain to:



 	
  1. Never, ever use “inclusive” upper bounds.

 	
  2. Never, ever use `<=` in a for loop condition.


By making myself a better programmer, I have solved the problem! Or have I? Not really. Let’s look again at the code from the perspective of the poor schlub who has to review it. He wants to ensure that `sumArray()` is correct. He must:



 	
  1. Look at all callers of `sumArray()` to see what kind of pointer is being passed.

 	
  2. Verify that the pointer actually is pointing to an array.

 	
  3. Verify that the size of the array is indeed `MAX`.


While this is trivial for the trivial program as presented here, it doesn’t really scale as the program complexity goes up. The more callers there are of `sumArray`, and the more indirect the data structures being passed to `sumArray`, the harder it is to do what amounts to data flow analysis in your head to ensure it is correct.

Even if you get it right, are you sure? What about when someone else checks in a change, is it still right? Do you want to do that analysis again? I’m sure you have better things to do. This is a tooling problem.

The fundamental issue with this particular problem is that a C array decays to a pointer when it's an argument to a function, even if the function parameter is declared to be an array. There’s just no escaping it. There’s no detecting it, either. (At least gcc and clang don’t detect it, maybe someone has developed an analyzer that does).

And so the tool to fix it is [D as a BetterC](https://dlang.org/blog/2017/08/23/d-as-a-better-c/) compiler. D has the notion of a _dynamic array_, which is simply a fat pointer, that is laid out like:

```d
struct DynamicArray {
    T* ptr;
    size_t length;
}
```
It’s declared like:

    int[] a;
and with that the example becomes:

```d
import core.stdc.stdio;

extern (C):   // use C ABI for declarations

enum MAX = 10;

int sumArray(int[] a) {
    int sum = 0;
    for (int i = 0; i <= MAX; ++i)
        sum += a[i];
    return sum;
}

int main() {
    __gshared int[MAX] values = [ 7,10,58,62,93,100,8,17,77,17 ];
    printf("sum = %d\n", sumArray(values));
    return 0;
}
```
Compiling:

```bash
dmd -betterC sum.d
```
Running:

```bash
./sum
Assertion failure: 'array overflow' on line 11 in file 'sum.d'
```
That’s more like it. Replacing the `<=` with `<` we get:

```bash
./sum
sum = 449
```
What’s happening is the dynamic array `a` is carrying its dimension along with it and the compiler inserts an array bounds overflow check.

But wait, there’s more.

There’s that pesky `MAX` thing. Since the `a` is carrying its dimension, that can be used instead:

    for (int i = 0; i < a.length; ++i)
This is such a common idiom, D has special syntax for it:

```d
foreach (value; a)
    sum += value;
```
The whole function `sumArray()` now looks like:

```d
int sumArray(int[] a) {
    int sum = 0;
    foreach (value; a)
        sum += value;
    return sum;
}
```
and now `sumArray()` can be reviewed in isolation from the rest of the program. You can get more done in less time with more reliability, and so can justify getting a pay raise. Or at least you won’t have to come in on weekends on an emergency call to fix the bug.

“Objection!” you say. “Passing `a` to `sumArray()` requires two pushes to the stack, and passing `p` is only one. You said no compromise, but I’m losing speed here.”

Indeed you are, in cases where `MAX` is a manifest constant, and not itself passed to the function, as in:

    int sumArray(int *p, size_t length);
But let’s get back to “no compromise.” D allows parameters to be passed by reference,
and that includes arrays of fixed length. So:

```d
int sumArray(ref int[MAX] a) {
    int sum = 0;
    foreach (value; a)
        sum += value;
    return sum;
}
```
What happens here is that `a`, being a `ref` parameter, is at runtime a mere pointer. It is typed, though, to be a pointer to an array of `MAX` elements, and so the accesses can be array bounds checked. You don’t need to go checking the callers, as the compiler’s type system will verify that, indeed, correctly sized arrays are being passed.

“Objection!” you say. “D supports pointers. Can’t I just write it the original way? What’s to stop that from happening? I thought you said this was a mechanical guarantee!”

Yes, you can write the code as:

```d
import core.stdc.stdio;

extern (C):   // use C ABI for declarations

enum MAX = 10;

int sumArray(int* p) {
    int sum = 0;
    for (int i = 0; i <= MAX; ++i)
        sum += p[i];
    return sum;
}

int main() {
    __gshared int[MAX] values = [ 7,10,58,62,93,100,8,17,77,17 ];
    printf("sum = %d\n", sumArray(&values[0]));
    return 0;
}
```
It will compile without complaint, and the awful bug will still be there. Though this time I get:

```python
sum = 39479
```
which looks suspicious, but it could have just as easily printed 449 and I’d be none the wiser.

How can this be guaranteed not to happen? By adding the attribute `@safe` to the code:

```d
import core.stdc.stdio;

extern (C):   // use C ABI for declarations

enum MAX = 10;

@safe int sumArray(int* p) {
    int sum = 0;
    for (int i = 0; i <= MAX; ++i)
        sum += p[i];
    return sum;
}

int main() {
    __gshared int[MAX] values = [ 7,10,58,62,93,100,8,17,77,17 ];
    printf("sum = %d\n", sumArray(&values[0]));
    return 0;
}
```
Compiling it gives:

    sum.d(10): Error: safe function 'sum.sumArray' cannot index pointer 'p'
Granted, a code review will need to include a grep to ensure `@safe` is being used, but that’s about it.

In summary, this bug is vanquished by preventing an array from decaying to a pointer when passed as an argument, and is vanquished forever by disallowing indirections after arithmetic is performed on a pointer. I’m sure a rare few of you have never been blasted by buffer overflow errors. Stay tuned for the next installment in this series. Maybe your moat got breached by the next bug! (Or maybe your tool doesn’t even have a moat.)
