---
author: VictorPorton
comments: false
date: 2019-04-29 13:00:23+00:00
layout: post
link: https://dlang.org/blog/2019/04/29/flexible-default-function-parameters/
slug: flexible-default-function-parameters
title: Flexible Default Function Parameters via structs with Nullable Fields
wordpress_id: 2065
categories:
- Code
- Guest Posts
permalink: /flexible-default-function-parameters/
redirect_from: /2019/04/29/flexible-default-function-parameters/
---

The problem

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)

Sometimes we need to combine an aggregate of a set of values with an aggregate of the corresponding set of default values to create a combined result. The result for each member is either the explicitly specified value or, where no value is specified, the default value. This is similar to default function arguments in D. However, D forces one to always specify the first N values in function parameters, but I want to be able to specify an arbitrary subset of the values. Example:

**Explicit values:** `a: 1, b: 2`

**Default values:** `a: 3, b: 4, c: 5`

**Combined result:** `a: 1, b: 2, c: 5`


## Possible solutions


The first idea is to use associative arrays. This approach is inefficient, however, because it combines values with string (or enum at best) names at runtime with associative array lookups and stores. It could be used like this (untested) example:

    combine(["a": 1, "b": 2], ["a": 3, "b": 4, "c": 5])
I was advised to instead use structs with nullable values (see below) to pass multiple values. This is nearly as efficient as possible because the members of the structs are enumerated at compile time (in fact, [I use `static foreach`](https://dlang.org/spec/version.html#staticforeach) in my implementation). So I implemented this solution. The source code is quite useful and [released under the Apache 2.0 license](https://github.com/vporton/struct-params-dlang).

To represent the (explicit) values of type `T`, we use a struct member of type `Nullable!T`. If it is `null`, this means that the explicit value is missing and the default value is used instead; otherwise the specified value is used.


## Example of definition and combination


First, [install the `struct-params` package](https://code.dlang.org/packages/struct-params) with DUB ([see the DUB documentation](https://dub.pm/getting_started); I strongly recommend using DUB to build D projects) or [clone my GitHub repository](https://github.com/vporton/struct-params-dlang).

Then add the following import to your source:

    import struct_params;
Example code:

```d
mixin StructParams!("S", int, "x", float, "y");
immutable S.WithDefaults combinedMain = { x: 12 }; // note y is default initialized to null
immutable S.Regular combinedDefault = { x: 11, y: 3.0 };
immutable combined = combine(combinedMain, combinedDefault);
assert(combined.x == 12 && combined.y == 3.0);
```
`StructParams` is a string mixin, [a D construct which generates D code](https://dlang.org/spec/statement.html#mixin-statement) at compile time and mixes it in at the point of declaration.

`mixin StructParams!("S", int, "x", float, "y");` effectively defines the following struct:

```d
struct S {
  struct Regular {
    int x;
    float y;
  }
  struct WithDefaults {
    Nullable!int x;
    Nullable!float y;
  }
}
```
`WithDefaults` is the struct to pass, for example, explicit values (which can be present (non-null) or missing (null) to be replaced with default values). For this, the D template type `Nullable` is used to represent either a value of a type or a `null` denoting a missing value.

`Regular` is just a standard struct with fields. `Regular` is a type which can be used to pass default values to the function `combine`.

This function `combine` combines explicit values with default values (as described above).

Then we assert that the result is correct.


## Calling functions


Finally, we will do the main thing for which all the above was intended and call a function with combined values:

```d
float f(int a, float b) {
    return a + b;
}
assert(callFunctionWithParamsStruct!f(combined) == combined.x + combined.y);
```
The structure `combined` is “split” into members and the members are passed as parameters to `f` in the order the fields of the struct are defined, that is in the order their names are specified as arguments to the `StructParams` string mixin. For those interested in the details: I use the built-in D `struct` and `class` property `.tupleof` to [split the structure into a tuple](https://dlang.org/spec/struct.html#struct_instance_properties) of members (e.g. explicitly calling `f` with an instance `reg` of `Regular` would look like this: `f(reg.tupleof);`).

We can also call a member function of a `struct` or `class` instance (`t` in the example below):

```d
struct Test {
    float f(int a, float b) {
        return a + b;
    }
}
Test t;
assert(callMemberFunctionWithParamsStruct!(t, "f")(combined) == combined.x + combined.y);
```
It is very unnatural to call the member `f` using a string of its name, but I have not found a better solution.

Another variant would be to use `callFunctionWithParamsStruct!((int a, float b) => t.f(a, b))(combined)`, but this way is inconvenient as it requires specifying arguments explicitly.


## Final considerations


Note that we cannot currently use struct initializers with named arguments, like `S.Regular(x: 11, y: 3.0)`, as the current version of D does not have this feature. There is [a draft D Improvement Proposal (DIP)](https://github.com/dlang/DIPs/pull/71) to introduce the feature, but I hear its author is going to replace it with a more general-case proposal after [DConf 2019 in London](https://dconf.org/2019/index.html).

It would be beneficial to implement such structs with default values, but this seems impossible, but representing every possible D value as a literal is apparently impossible (for example, a structure with circular references to other structures seems not to be representable as a literal). We can attempt to implement it for a subset of types of values, or even for some values of types and not for others, but this would require further consideration.

_Victor Porton is an open source developer, a math researcher, and a Christian writer. He earns his living as a programmer._
