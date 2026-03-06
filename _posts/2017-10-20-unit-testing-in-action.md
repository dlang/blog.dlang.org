---
author: MarioKroeplin
comments: false
date: 2017-10-20 13:57:12+00:00
excerpt: 'Ten years ago, programming in D was like starting over in our company. And,
  of course, unit testing was part of it right from the beginning. D’s built-in simple
  support made it easy to quickly write lots of unit tests. Until some of them failed.
  And soon, the failure became the rule. There’s always someone else to blame: D’s
  simple unit-test support is too simple. A look at Python reveals that the modules
  doctest and unittest live side by side in the standard library. We concluded that
  D’s unit test support corresponds to Python’s doctest, which means that there must
  be something else for the real unit testing.'
layout: post
link: https://dlang.org/blog/2017/10/20/unit-testing-in-action/
slug: unit-testing-in-action
title: Unit Testing In Action
wordpress_id: 1143
categories:
- Code
- Companies
- Guest Posts
permalink: /unit-testing-in-action/
redirect_from: /2017/10/20/unit-testing-in-action/
---

_[![](http://www.funkwerk.com/wp-content/themes/funkwerk/library/images/logo.png)](http://www.funkwerk.com)Mario Kröplin is a developer at [Funkwerk AG](http://www.funkwerk.com/), a German company whose passenger information system is developed in D and was [recently highlighted on this blog](https://dlang.org/blog/2017/07/28/project-highlight-funkwerk/). That post describes Funkwerk’s use of third-party unit testing frameworks and says, “the team recently discovered a way to combine xUnit testing with D’s built-in `unittest`, which may lead to another transition in their unit testing.” That’s Mario’s subject in this post._



* * *





### There and Back Again


Ten years ago, programming in D was like starting over in our company. And, of course, unit testing was part of it right from the beginning. D’s built-in simple support made it easy to quickly write lots of unit tests. Until some of them failed. And soon, the failure became the rule. There’s always someone else to blame: D’s simple unit-test support is too simple. A look at Python reveals that the modules `doctest` and `unittest` live side by side in the standard library. We concluded that D’s unit test support corresponds to Python’s `doctest`, which means that there must be something else for the real unit testing.

Even back then, we immediately found such a unit testing framework in DUnit [_An old D1 unit testing framework that you can read about at the old [dsource.org](http://www.dsource.org/projects/dmocks/wiki/DUnit) – Ed._]. Thanks to good advice for [xUnit testing](http://xunitpatterns.com/), we were happy and content with this approach. At the end of life of D1, a replacement library for D2 was soon found. After a bumpy start, I found myself in the role of the maintainer of [dunit](https://github.com/linkrope/dunit) [_A D2 unit-testing framework that is separate from DUnit – Ed_].

During [DConf 2013](http://dconf.org/2013/), I copied a first example use of user-defined attributes to dunit. This allowed imitating [JUnit 4](http://junit.org/junit4/), where, for example, test methods are annotated with `@Test`. By now, dunit imitates [JUnit 5](http://junit.org/junit5/). So if you want to write unit tests in Java style, dunit is a good choice. But which D programmers would want to do that?

Recently, we reconsidered the weaknesses of D’s unit test support. Various solutions have been found to bypass the blockers (described in the following). On the other hand, good guidelines are added, for example, to use attributes even for `unittest` functions. So we decided to return to making use of D’s built-in unit test support. From our detour we retain some ideas to keep the test implementation maintainable.


### Expectations


Whenever a unit test fails at run time, the question is, why? The error message refers to the line number, where you find something like `assert(answer == 42)`. But what is the value of `answer` if it isn't `42`? The irony is that this need is well understood. If you use a static assert instead, the error message reads like: `static assert 54 == 42 is false`. The fear of code bloat is the reason why you don’t get this automatically at run time.

If you look at the [Language Reference](https://dlang.org/spec/spec.html), you will notice that the chapter [Unit Tests](https://dlang.org/spec/unittest.html) covers primarily the special `unittest` function. It is assumed that `assert` is used for test verification, which is introduced in the chapter [Contract Programming](https://dlang.org/spec/contracts.html). In theory, it’s completely OK to reuse `assert` for test verification. Any failure reveals a programming error that must be fixed. In practice, however, test expectations are quite different from preconditions, postconditions, and invariants. While the expectations are usually specific (`actual == expected`) the contracts rather exclude specific values ​​(`value != 0` or `value !is null`).

So there are lots of implementations of templates like `assertEquals` or `test!"=="`. The problem shows up if you want to have the most helpful error messages: `expected 42 but got 54`. For this, `assertEquals` is too symmetrical. In fact, JUnit’s `assertEquals(expected, actual)` was turned into [TestNG’s](http://testng.org/doc/) `assertEquals(actual, expected)`. Even with UFCS ([Uniform Function Call Syntax](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs)), it is not clear how `a.assertEquals(b)` should be used. From time to time, programmers don’t write the arguments in the intended order. Then the error messages are the opposite of helpful. They are misleading: `expected 54 but got 42`.

Fluent assertions avoid this symmetry problem: `actual.should.eq(expected)` or `expect(actual).to.eq(expected)` are harder to use incorrectly. Thanks to UFCS and lazy parameters, the implementation in D is no problem. The common criticism is “the natural language formulation is too verbose”, or just “too many dots”. Currently, however, this seems to be the only way to get the most helpful error messages.

The next problem is that string comparisons are seldom as simple as: `expected foo but got bar`. Non-printable characters or lengthy texts, such as XML or JSON representations, sabotage error messages that were meant to be helpful. This can be avoided by escaping special characters and by showing differences. Finally, this is what [the fluent-asserts library](https://github.com/gedaiu/fluent-asserts) does.


### Test Execution


At large, we want to get as much information as possible from a failed test run. How many test cases fail? Which test cases fail? Does the happy path fail or rather edge cases? Is it worth addressing the failures, or is it better to undo the change? The approach of stopping on the first error is contrary to these needs. The original idea was to run the unit tests before the start of the actual program. By now, however, separate test runners are often used, which continue in case of a failure. To emphasize this, test expectations usually throw their own exceptions, instead of the unrecoverable `AssertError`. This change already shows how many test cases fail.

Finding out what’s tested in the failing test cases is more difficult. At best, there are corresponding comments for documented unit tests. But an empty [DDoc](https://tour.dlang.org/tour/en/gems/documentation) comment, `///`, is all that’s needed to include the body of the `unittest` function as an example in the documentation. In the worst case, the unit test goes on and on verifying this and that.

The idea of the [Sentence Style For Naming Unit Tests](http://wiki.c2.com/?SentenceStyleForNamingUnitTests) is that the name of the test function describes the test case. In D, however, the `unittest` functions are anonymous. On the other hand, D has [user-defined attributes](https://dlang.org/spec/attribute.html#uda) so that you can even use strings for the test description instead of CamelCase names. [unit-threaded](https://github.com/atilaneves/unit-threaded), for example, shows these string attributes so that you get a good impression of the extent of the problem in case of a failure. In addition, unit-threaded satisfies the requirement to execute test cases selectively. For example, only the one problematic test case or all tests except those tagged as “slow”. It’s promising to use unit-threaded as needed. You let D run the `unittest` functions as long as they pass. Only for troubleshooting should you switch to unit-threaded. You have to be careful, however, to only use compatible features.

By the way: the parallel test execution (from it’s name, the main goal of unit-threaded) was quite problematic with the first test suite we converted. On the other hand, the speedup was just 10%.


### Coverage


The D compiler has [built-in code-coverage analysis](https://dlang.org/code_coverage.html). The ratio of the lines executed in the test is often used as an indicator for the quality of the tests. (See: [Testing in the D Standard Library](https://dlang.org/blog/2017/01/20/testing-in-the-d-standard-library/)) A coverage of 100% cannot be achieved, for example, if you have an `assert(0)`. Lower thresholds for the coverage can always be achieved by cheating. The fact that the `unittest` functions are also incorporated in the coverage is questionable. Imagine that a single line that has not yet been executed requires a lengthy unit test. As a consequence, this new unit test could significantly raise the coverage.

In order to avoid such measurement errors, we decided from the beginning to extract non-trivial unit tests to separate modules. We place these in parallel to the `src` tree in a `unittest` directory. Test utilities are also placed in the `unittest` directory, so that reading the actual code is not encumbered by large `version (unittest)` sections. (We also have test directories for [customer tests](http://xunitpatterns.com/customer%20test.html).) For the coverage, we only count the modules under `src`. Code-coverage analysis creates a report file for each module. For a summary, which we output at the end of each successful test run, we have written a simple script. By now, [covered](https://github.com/ohdatboi/covered) is a ready-made solution.

In order to fully exploit the code-coverage analysis, an unusual formatting is required, for example, for the short-circuit evaluation of expressions with `&&,` `||`, and `?:`. We hope that [dfmt](https://github.com/dlang-community/dfmt) can be changed to reformat the code temporarily.


### Fixtures


What can you do to prevent the test implementation from getting out of control? After all, test code is also code that needs to be maintained. Sometimes the test implementation is more obscure than the code being tested.

As a solution the xUnit patterns suggest a structuring of the test implementation as a [Four-Phase Test](http://xunitpatterns.com/Four%20Phase%20Test.html): fixture setup, exercise system under test, result verification, fixture teardown. The term _fixture_ refers to the test context. For JUnit, this is the test class with attributes that are available to all test methods. A method with the annotation `@BeforeEach` initializes the attributes. This is the fixture setup. Another method with the annotation `@AfterEach` implements the fixture teardown. All methods annotated with `@Test` focus on exercise and verification.

At first glance, this approach seems to be incompatible with D’s `unittest` functions. The `unittest` functions do not get automatic access to the attributes of a class, even if they are defined in the context of a class. On the other hand, one can mimic the approach, for example, by implementing the fixtures next to the `unittest` functions as a `struct`:

```d
unittest
{
    Fixture fixture;
    fixture.setup;
    scope (exit) fixture.teardown;
    (fixture.x * fixture.y).should.eq(42);
}
```
The test implementation can be improved by executing the fixture setup in the constructor (or in [`opCall()`](https://dlang.org/spec/operatoroverloading.html#function-call), since default constructors are disallowed in `struct`s) and the fixture teardown in the destructor:

```d
unittest
{
    with (Fixture())
        (x * y).should.eq(42);
}
```
The `with (Fixture())` pulls the context, in which test methods are executed implicitly in JUnit, explicitly into the `unittest` function. With this simple pattern you can structure unit tests in a tried and trusted way without having to use a framework for test classes ever again.


### Parameterized Tests


A parameterized test is a means to reuse a test implementation with different values ​​or with different types. Within a `unittest` function this would be no problem. Our goal, however, is to get as much information as possible from a failing test run. For which values ​​or which types does the test fail? unit-threaded provides support for parameterized tests with `@Values` ​​and `@Types`. If unit-threaded is not used to run the `unittest` functions, these test cases do not work at all.

With the new [`static foreach` feature](https://dlang.org/spec/version.html#staticforeach) however, it is easy to implement parameterized tests without the support of a framework:

```d
static foreach (i; 0 .. 2)
    static foreach (j; 0 .. 2)
        @(format!"%s + %s == 1"(i, j))
        unittest
        {
            (i + j).should.eq(1);
        }
```
And if you run the failing test with unit-threaded, the descriptions of the failing test cases reveal the problem without the need to take a look at the test implementation:

    0 + 0 == 1: expected 1 but got 0
    1 + 1 == 1: expected 1 but got 2
### Conclusion


D’s built-in unit test support works best when there are no failures. As shown, however, you do not need to change too much to be able to work properly in situations where you rely on helpful error messages. The imitation of a solution from another programming language is often easy in D. Nevertheless, one should reconsider such solutions from time to time.

If we had a wish, we would want separate libraries for expectations and for test execution. Currently, you get frameworks where not all features are great, or they are overloaded with alternative solutions. Such a separation should probably be supported by the [Phobos runtime library](https://dlang.org/phobos/index.html). Currently, each framework defines expectations with its own unit test exceptions. In order to combine them, ugly interdependencies are required to match the exceptions thrown in one library to the exceptions caught in another library. A unit test exception in Phobos could avoid this problem.
