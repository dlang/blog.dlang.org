---
author: SebWil
comments: false
date: 2017-03-08 13:18:34+00:00
layout: post
link: https://dlang.org/blog/2017/03/08/editable-and-runnable-doc-examples-on-dlang-org/
slug: editable-and-runnable-doc-examples-on-dlang-org
title: Editable and Runnable Doc Examples on dlang.org
wordpress_id: 695
categories:
- Compilers &amp; Tools
- Guest Posts
- The Language
permalink: /editable-and-runnable-doc-examples-on-dlang-org/
redirect_from: /2017/03/08/editable-and-runnable-doc-examples-on-dlang-org/
---

_Sebastian Wilzbach was a [GSoC student](http://blog.mir.dlang.io/random/2016/08/19/intro-to-random-sampling.html) for the D Language Foundation in 2016 and has since become a regular contributor to [Phobos](https://github.com/dlang/phobos), D's standard library, and [dlang.org](http://dlang.org/)._



* * *



This article explains the steps that were needed to have editable and runnable examples in the documentation on [dlang.org](http://dlang.org/phobos/index.html). First, let’s begin with the building blocks.


### Unit testing in D


One of D’s coolest features is its `unittest` block, which allows the insertion of testable code anywhere in a program. It has become idiomatic for a function to be followed directly by its tests. For example, let’s consider a simple function `add` which is accompanied by two tests:

```d
auto add(int a, int b)
{
    return a + b;
}

unittest
{
    assert(2.add(2) == 4);
    assert(3.add(4) == 7);
}
```
By default, all `unittest` blocks will be ignored by the compiler. Specifying **-unittest** on the compiler's command line will cause the unit tests to be included in the compiled binary. Combined with **-main**, tests in D can be directly executed with:

```bash
rdmd -main -unittest add.d
```
If a `unittest` block is annotated with [embedded documentation](http://dlang.org/spec/ddoc.html), a D documentation generator can also display the tests as examples in the generated documentation. The DMD compiler ships with a built-in documentation generator ([DDoc](http://dlang.org/spec/ddoc.html)), which can be run with the **-D** flag, so executing:

```bash
dmd -D -main add.d
```
would yield the documentation of the `add` function above with its tests displayed as examples, as demonstrated here:

![](http://dlang.org/blog/wp-content/uploads/2017/03/image00.png)

Please note that the [documentation on dlang.org](http://dlang.org/phobos) is generated with DDoc. However, in case you don’t like DDoc, there are several [other options](https://wiki.dlang.org/Documentation_Generators).


### Executing code on the web


Frequent readers of the D Blog might remember [Damian Ziemba’s DPaste](https://dlang.org/blog/2017/01/30/project-highlight-dpaste/) - an online compiler for the D Programming language. In 2012, he [made the examples on the front page](https://github.com/dlang/dlang.org/pull/132) of D’s website runnable via his service. Back in those old days, the website of the D Programming language looked like this:

![](http://dlang.org/blog/wp-content/uploads/2017/03/image01-1024x378.png)

As a matter of fact, until 2015, communication with DPaste [was done in XML](https://github.com/dlang/dlang.org/pull/1112).


### Putting things together


So D has a unit test system that allows placing executable unit tests next to the functions they test, the tests can also be rendered as examples in the generated documentation, and there exists a service, in the form of DPaste, that allows D code to be executed on the web. The only thing missing was to link them together to produce interactive documentation for a compiled language.

There was one big caveat that needed to be addressed before that could happen. While D’s test suite, which is run on [ten different build machines](http://auto-tester.puremagic.com/), ensures that [all unit tests compile & run without errors](https://dlang.org/blog/2017/01/20/testing-in-the-d-standard-library/), an extracted test might contain symbols that were imported at module scope and thus wouldn’t be runnable on dlang.org. A `unittest` block can only be completely independent of the module in which it is declared if all of its symbols are imported locally in the test’s scope. The solution was rather simple: [extract all tests from Phobos](https://github.com/dlang/tools/blob/master/styles/tests_extractor.d), then compile and execute them separately to ensure that a user won’t hit a “missing import” error on dlang.org. Thanks to D’s ultra-fast frontend, this step takes less than a minute on a typical machine in single-core build mode.

Moreover, to prevent any regressions, this has been [integrated into Phobos’s test suite](https://github.com/dlang/phobos/blob/master/posix.mak#L545) and is run for every PR via [CircleCi](https://circleci.com/gh/dlang/phobos). As Phobos has [extensive coverage](http://codecov.io/gh/dlang/phobos) with unit tests, we started this transition with a [blacklist](https://github.com/dlang/dlang.org/blob/master/js/run_examples.js#L49) and, [step-by-step](https://github.com/dlang/phobos/pull/4966), removed modules for which all extracted tests compile. With continuous checking in place, we were certain that none of the exposed unit tests would throw any errors when executed in the documentation, so we could [do the flip](https://github.com/dlang/dlang.org/pull/1297) and replace the syntax-highlighted unit test examples with an interactive code editor.


### Going one step further


With this setup in place, hitting the “Run” button would merely show the users “All tests passed”. While that’s always good feedback, it conveys less information than is usually desirable.

[Documentation](https://lodash.com/docs) that supports runnable examples tends to send any output to `stdout`. This allows the reader to take the example and modify it as needed while still seeing useful output about the modifications. So, for example, instead of using assertions to validate the output of a function, which is idiomatic in D unit tests and examples:

    assert(myFun() == 4);
Other documentation usually prints to `stdout` and shows the expected output in a comment. In D, that would look like this:

```d
writeln(myFun()); // 4
```
I [initially](https://github.com/dlang/dlang.org/pull/1297/files#diff-034369de775110cb179ed1e925b62a0fR10) tried to do such a transformation with regular expressions, but I was quickly [bitten](http://forum.dlang.org/post/mxdwzkirgzutmbrlpzgb@forum.dlang.org) by the complexity of a context-free language. So I made [another attempt](https://github.com/dlang/dlang.org/pull/1582) using Brian Schott’s [libdparse](https://github.com/Hackerpilot/libdparse), a library to parse and lex D source code. libdparse allows one to traverse the abstract syntax tree (AST) of a D source file. During the traversal of the AST, the transformation tool can rewrite all **AssertExpressions** into `writeln` calls, similar to the way other documentation displays examples. To speak in the [vocabulary](http://www.drdobbs.com/architecture-and-design/so-you-want-to-write-your-own-language/240165488?pgno=2) of compiler devs: we are _lowering_ **AssertExpressions** into the more humanly digestible `writeln` calls!

Once the AST has been traversed and modified, it needs to be transformed into source code again. This led to improvements in libdparse’s formatting capabilities ([1](https://github.com/Hackerpilot/libdparse/pull/128), [2](https://github.com/Hackerpilot/libdparse/pull/130)).


### The future


As of now, there are still a small number of functions in Phobos that don’t have a nice public example that is runnable on dlang.org. [Tooling](https://github.com/dlang/tools/blob/master/styles/has_public_example.d) to check for this has recently been [activated in Phobos](https://github.com/dlang/phobos/pull/4998). So now you can use this tool (**make -f posix.mak has_public_example**) to find functions lacking public tests and remove those modules from the [blacklist](https://github.com/dlang/phobos/blob/master/posix.mak#L554).

Another target for improvement is DPaste. For example, it currently doesn’t cache incoming requests, which could improve the performance of executed examples on dlang.org. However, due to the fast compilation speed of the DMD compiler, this “slow-down” isn’t noticeable and is more of a perfectionist wish.

I hope you enjoy the new “Run” button on the documentation and have as much fun playing with it as I do. [Click here](https://dlang.org/phobos/std_algorithm_searching.html#.minElement) to get started.
