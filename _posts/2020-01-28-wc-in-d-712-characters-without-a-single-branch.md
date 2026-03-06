---
author: RobertSchadek
comments: false
date: 2020-01-28 13:54:16+00:00
layout: post
link: https://dlang.org/blog/2020/01/28/wc-in-d-712-characters-without-a-single-branch/
slug: wc-in-d-712-characters-without-a-single-branch
title: 'wc in D: 712 Characters Without a Single Branch'
wordpress_id: 2303
categories:
- Code
- Guest Posts
- The Language
- Tutorials
permalink: /wc-in-d-712-characters-without-a-single-branch/
redirect_from: /2020/01/28/wc-in-d-712-characters-without-a-single-branch/
---

After reading [“Beating C With 80 Lines Of Haskell: Wc”](https://chrispenner.ca/posts/wc), which I found on [Hacker News](https://news.ycombinator.com/news), I thought D could do better. So I wrote a `wc` in D.


## The Program


It consists of one file and has 34 lines and 712 characters.

```d
import std.stdio : writefln, File;
import std.algorithm : map, fold, splitter;
import std.range : walkLength;
import std.typecons : Yes;
import std.uni : byCodePoint;

struct Line {
	size_t chars;
	size_t words;
}

struct Output {
	size_t lines;
	size_t words;
	size_t chars;
}

Output combine(Output a, Line b) pure nothrow {
	return Output(a.lines + 1, a.words + b.words, a.chars + b.chars);
}

Line toLine(char[] l) pure {
	return Line(l.byCodePoint.walkLength, l.splitter.walkLength);
}

void main(string[] args) {
	auto f = File(args[1]);
	Output o = f
		.byLine(Yes.keepTerminator)
		.map!(l => toLine(l))
		.fold!(combine)(Output(0, 0, 0));

	writefln!"%u %u %u %s"(o.lines, o.words, o.chars, args[1]);
}
```
Sure, it is using [Phobos, D’s standard library](https://dlang.org/phobos/index.html), but then why wouldn’t it? Phobos is awesome and ships with every D compiler. The program itself does not contain a single `if` statement. The Haskell `wc` implementation has several `if` statements. The D program, apart from the `main` function, contains three tiny functions. I could have easily put all the functionally in one range chain, but then it probably would have exceeded 80 characters per line. That’s a major code-smell.


## The Performance


Is the D `wc` faster than the coreutils `wc`? No, but it took me 15 minutes to write mine (I had to [search for `walkLength`](https://dlang.org/phobos/std_range_primitives.html#walkLength), because I forgot its name).
<table > 

<tr >
<td>file</td>
<td>lines</td>
<td>bytes</td>
<td>coreutils</td>
<td>haskell</td>
<td>D</td>
</tr>

<tbody >
<tr >

<td >app.d
</td>

<td >46
</td>

<td >906
</td>

<td >3.5 ms +- 1.9 ms
</td>

<td >39.6 ms +- 7.8 ms
</td>

<td >8.9 ms +- 2.1 ms
</td>
</tr>
<tr >

<td >big.txt
</td>

<td >862
</td>

<td >64k
</td>

<td >4.7 ms +- 2.0 ms
</td>

<td >39.6 ms +- 7.8 ms
</td>

<td >9.8 ms +- 2.1 ms
</td>
</tr>
<tr >

<td >vbig.txt
</td>

<td >1.7M
</td>

<td >96M
</td>

<td >658.6ms +- 24.5ms
</td>

<td >226.4 ms +- 29.5 ms
</td>

<td >1.102 s +- 0.022 s
</td>
</tr>
<tr >

<td >vbig2.txt
</td>

<td >12.1M
</td>

<td >671M
</td>

<td >4.4 s +- 0.058 s
</td>

<td >1.1 s +- 0.039 s
</td>

<td >7.4 s +- 0.085 s
</td>
</tr>
</tbody>
</table>

Memory:
<table > 

<tr >
<td>file</td>
<td>coreutils</td>
<td>haskell</td>
<td>D</td>
</tr>

<tbody >
<tr >

<td >app.d
</td>

<td >2052K
</td>

<td >7228K
</td>

<td >7708K
</td>
</tr>
<tr >

<td >big.txt
</td>

<td >2112K
</td>

<td >7512K
</td>

<td >7616K
</td>
</tr>
<tr >

<td >vbig.txt
</td>

<td >2288K
</td>

<td >42620K
</td>

<td >7712K
</td>
</tr>
<tr >

<td >vbig2.txt
</td>

<td >2360K
</td>

<td >50860K
</td>

<td >7736K
</td>
</tr>
</tbody>
</table>
Is the Haskell `wc` faster? For big files, absolutely, but then it is using threads. For small files, GNU's coreutils still beats the competition. At this stage my version is very likely IO bound, and it’s fast enough anyway.

I’ll not claim that one language is faster than another. If you spend a chunk of time on optimizing a micro-benchmark, you are likely going to beat the competition. That’s not real life. **But I will claim that functional programming in D gives functional programming in Haskell a run for its money.**


## A Bit About Ranges


![Digital Mars D logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)A range is an abstraction that you can consume through iteration without consuming the underlying collection (if there is one). Technically, a range can be a struct or a class that adheres to one of a handful of `Range` interfaces. The most basic form, [the `InputRange`](https://dlang.org/phobos/std_range_primitives.html#isInputRange), requires the function

```d
void popFront();
```
and two members or properties:
```d
T front;
bool empty;
```
`T` is the generic type of the elements the range iterates.

In D, ranges are special in a way that other objects are not. When a range is given to a `foreach` statement, the compiler does a little rewrite.

```d
foreach (e; range) { ... }
```
is rewritten to

```d
for (auto __r = range; !__r.empty; __r.popFront()) {
    auto e = __r.front;
    ...
}
```
`auto e =` infers the type and is equivalent to `T e =`.

Given this knowledge, building a range that can be used by `foreach` is easy.

```d
struct Iota {
	int front;
	int end;

	@property bool empty() const {
		return this.front == this.end;
	}

	void popFront() {
		++this.front;
	}
}

unittest {
	import std.stdio;
	foreach(it; Iota(0, 10)) {
		writeln(it);
	}
}
```
`Iota` is a very simple range. It functions as a generator, having no underlying collection. It iterates integers from front to end in steps of one. This snippet introduces a little bit of D syntax.

```d
@property bool empty() const {
```
The `@property` attribute allows us to use the function `empty` the same way as a member variable (calling the function without the parenthesis). The trailing `const` means that we don’t modify any data of the instance we call `empty` on. The built-in unit test prints the numbers `0` to `10`.

Another small feature is the lack of an explicit constructor. The struct `Iota` has two member variables of type `int`. In the `foreach` statement in the test, we create an `Iota` instance as if it had a constructor that takes two `int`s. This is [a struct literal](https://dlang.org/spec/struct.html#struct-literal). When the D compiler sees this, and the struct has no matching constructor, the `int`s will be assigned to the struct’s member variables from top to bottom in the order of declaration.

The relation between the three members is really simple. If `empty` is `false`, `front` is guaranteed to return a different element, the next one in the iteration, after a call to `popFront`. After calling `popFront` the value of `empty` might have changed. If it is `true`, this means there are no more elements to iterate and any further calls to `front` are not valid. According to [the `InputRange` documentation](https://dlang.org/phobos/std_range_primitives.html#isInputRange):



 	
  * `front` can be legally evaluated if and only if evaluating `empty` has, or would have, equaled `false`.

 	
  * `front` can be evaluated multiple times without calling `popFront` or otherwise mutating the range object or the underlying data, and it yields the same result for every evaluation.


Now, using `foreach` statements, or loops in general, is not really functional in my book. Lets say we want to filter all uneven numbers of the `Iota` range. We could put an `if` inside the `foreach` block, but that would only make it worse. It would be nicer if we had a range that takes a range and a predicate that can decide if an element is okay to pass along or not.

```d
struct Filter {
	Iota input;
	bool function(int) predicate;

	this(Iota input, bool function(int) predicate) {
		this.input = input;
		this.predicate = predicate;
		this.testAndIterate();
	}

	void testAndIterate() {
		while(!this.input.empty
				&& !this.predicate(this.input.front))
		{
			this.input.popFront();
		}
	}

	void popFront() {
		this.input.popFront();
		this.testAndIterate();
	}

	@property int front() {
		return this.input.front;
	}

	@property bool empty() const {
		return this.input.empty;
	}
}

bool isEven(int a) {
	return a % 2 == 0;
}

unittest {
	foreach(it; Filter(Iota(0,10), &isEven)) {
		writeln(it);
	}
}
```
`Filter` is again really simple: it takes one `Iota` and a function pointer. On construction of `Filter`, we call `testAndIterate`, which pops elements from `Iota` until it is either empty or the predicate returns `false`. The idea is that the passed predicate decides what to filter out and what to keep. The properties `front` and `empty` just forward to `Iota`. The only thing that actually does any work is `popFront`. It pops the current element and calls `testAndIterate`. That’s it. That’s an implementation of filter.

Sure, there is a new `while` loop in `testAndIterate`, but rewriting that with recursion is just silly, in my opinion. What makes D great is that you can use the right tool for the job. Functional programming is fine and dandy a lot of the time, but sometimes it’s not. If a bit of inline assembly would be necessary or nicer, use that.

The call to `Filter` still does not look very nice. Assuming, you are used to reading from left to right, `Filter` comes before `Iota`, even though it is executed after `Iota`. D has no pipe operator, but it does have [Uniform Function Call Syntax (UFCS)](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs). If an expression can be implicitly converted to the first parameter of a function, the function can be called like it is a member function of the type of the expression. That's a lot of words, I know. An example helps:

```d
string foo(string a) {
	return a ~ "World";
}

unittest {
	string a = foo("Hello ");
	string b = "Hello ".foo();
	assert(a == b);
}
```
The above example shows two calls to the function `foo`. As the `assert` indicates, both calls are equivalent. What does that mean for our Iota Filter example? UFCS allows us to rewrite the unit test to:

```d
unittest {
	foreach(it; Iota(1,10).Filter(&isEven)) {
		writeln(it);
	}
}
```
Implementing a map/transform range should now be possible for every reader. Sure, `Filter` can be made more abstract through [the use of templates](https://dlang.org/spec/template.html), but that’s just work, nothing conceptually new.

Of course, there are different kinds of ranges, [like a bidirectional range](https://dlang.org/phobos/std_range_primitives.html#isBidirectionalRange). Guess what that allows you to do. A small tip: a bidirectional range has two new primitives called `back` and `popBack`. There are other range types as well, but after you understand the input range demonstrated twice above, you pretty much know them all.

P.S. Just to be clear, do not implement your own `filter`, `map`, or `fold`; the D standard library Phobos has everything you every need. Have a look at [`std.algorithm`](https://dlang.org/phobos/std_algorithm.html) and [`std.range`](https://dlang.org/phobos/std_range.html).


## About the Author


Robert Schadek received a master's degree in Computer Science at the University of Oldenburg. His master thesis was titled “DMCD A Distributed Multithreading Caching D Compiler” where he work on building a D compiler from scratch. He was a computer science PhD student from 2012–2018 at the University of Oldenburg. His PhD research focuses on quorum systems in combination with graphs. Since 2018 he is happily using D in his day job working for Symmetry Investments.


### What is Symmetry Investments?


Symmetry Investments is a global investment company with offices in Hong Kong, Singapore and London. We have been in business since 2014 after successfully spinning off from a major New York-based hedge fund.

At Symmetry, we seek to engage in intelligent risk-taking to create value for our clients, partners and employees. We derive our edge from our capacity to generate Win-Wins – in the broadest sense. Win-Win is our fundamental ethical and strategic principle. By generating Win-Wins, we can create unique solutions that reconcile perspectives that are usually seen as incompatible or opposites, and encompass the best that each side has to offer. We integrate fixed-income arbitrage with global macro strategies in a novel way. We invent and develop technology that focuses on the potential of human-machine integration. We build systems where machines do what they do best, supporting people to do what people do best. We are creating a collaborative meritocracy: a culture where individual contribution serves both personal and collective goals – and is rewarded accordingly. We value both ownership thinking AND cooperative team spirit, self-realisation AND community.

People at Symmetry Investments have been active participants in the D community since 2014. We have sponsored the development of [excel-d](https://github.com/symmetryinvestments/excel-d), [dpp](https://github.com/atilaneves/dpp), [autowrap](https://github.com/symmetryinvestments/autowrap), [libmir](https://github.com/libmir), and various other projects. We started Symmetry Autumn of Code in 2018 and hosted DConf 2019 in London.
