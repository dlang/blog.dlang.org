---
author: MarcoDeWild
comments: false
date: 2019-03-25 12:44:55+00:00
layout: post
link: https://dlang.org/blog/2019/03/25/using-const-to-enforce-design-decisions/
slug: using-const-to-enforce-design-decisions
title: Using const to Enforce Design Decisions
wordpress_id: 2006
categories:
- Code
- Guest Posts
- The Language
permalink: /using-const-to-enforce-design-decisions/
redirect_from: /2019/03/25/using-const-to-enforce-design-decisions/
---

![](http://dlang.org/blog/wp-content/uploads/2016/08/d3.png)

The saying goes that the best code is no code. As soon as a project starts to grow, technical debt is introduced. When a team is forced to adapt to a new company guideline inconsistent with their previous vision, the debt results from a business decision. This could be tackled at the company level. Sometimes technical debt can arise simply due to the passage of time, when new versions of dependencies or of the compiler introduce breaking changes. You can try to tackle this by letting your local physicist stop the flow of time. More often however, technical debt is caused when issues are fixed by a quick hack, due to time pressure or a lack of knowledge of the code base. Design strategies that were carefully crafted are temporarily neglected. This blog post will focus on using the `const` modifier. It is one of the convenient tools D offers to minimize the increase of technical debt and enforce design decisions.

To keep a code base consistent, often design guidelines, either explicit or implicit, are put in place. Every developer on the team is expected to adhere to the guidelines as a gentleman’s agreement. This effectively results in a policy that is only enforced if both the programmer and the reviewer have had enough coffee. Simple changes, like adding a call to an object method, might seem innocent, but can reduce the consistency and the traceability of errors. To detect this in a code review requires in-depth knowledge of the method’s implementation.

An example from a real world project I’ve worked on is generating financial transactions for a read-only view, the `display` function in the following code fragment. Nothing seemed wrong with it, until I realized that those transactions were persisted and eventually used for actual payments _without being calculated again_, as seen in the `process` method. Potentially different payments occurred depending on whether the user decided to glance at the summary, thereby triggering the generation with new currency exchange rates, or just blindly clicked the OK button. That’s not what an innocent bystander like myself expects and has caused many frowns already.

```d
public class Order
{
    private Transaction[] _transactions;

    public Transaction[] getTransactions()
    {
        _transactions = calculate();
        return _transactions;
    }

    public void process()
    {
        foreach(t; _transactions){
            // ...
        }
    }
}

void display(Order order)
{
    auto t = order.getTransactions();
    show(t);
}
```
The internet has taught me that if it is possible, it will one day happen. Therefore, we should make an attempt to make the undesired impossible.


## A constant feature


By default, variables and object instances in D are mutable, just like in many other programming languages. If we want to prevent objects from ever changing, we can mark them `immutable`, i.e. `immutable MyClass obj = new MyClass();`. `immutable` means that we can modify neither the object reference (_head constant_) nor the object’s properties (_tail constant_). The first case corresponds to `final` in Java and `readonly` in C#, both of which signify head constant only. D’s implementation means that nobody can ever modify an object marked `immutable`. What if an object needs to be mutable in one place, but immutable in another? [That’s where D’s `const`](https://dlang.org/spec/const3.html#const_and_immutable) pops in.

Unlike `immutable`, whose contract states that an object cannot be mutated through any reference, `const` allows an object to be modified through another, non-`const` reference. This means it’s illegal to initialize an `immutable` reference with a mutable one, but a `const` reference can be initialized with a mutable, `const`, or `immutable` reference. In a function parameter list, `const` is preferred over `immutable` because it can accept arguments with either qualifier or none. Schematically, it can be visualized as in the following figure.

![const relationships](http://dlang.org/blog/wp-content/uploads/2019/03/image2.png)


## A constant detour


D’s `const` differs from C++ `const` in a significant way: it’s transitive (see [the const(FAQ)](https://dlang.org/articles/const-faq.html) for more details). In other words, it’s not possible to declare any object in D as head constant. This isn’t obvious from examples with classes, since classes in D are reference types, but is more apparent with pointers and arrays. Consider these C++ declarations:

    const int *cp0;         // mutable pointer to const data
    int const *cp1;         // alternative syntax for the same
Variable declarations in C++ are best read from right to left. Although `const int` is likely the more common syntax, `int const` matches the way the declaration should be read: _cp0 is a mutable pointer to a const int_. In D, the equivalent of `cp0` and `cp1` is:

    const(int)* dp0;
The next example shows what head constant looks like in C++.

    int *const cp2;         // const pointer to mutable data
We can read the declaration of `cp2` as: _cp2 is a const pointer to a mutable int_. There is no equivalent in D. It’s possible in C++ to have any number and combination of `const` and mutable pointers to `const` or mutable data. But in D, if `const` is applied to the outermost pointer, then it applies to all the inner pointers and the data as well. Or, as they say, it’s turtles all the way down.

The equivalent in C++ looks like this:

    int const *const cp3;         // const pointer to const data
This declaration says _cp3 is a const pointer to const data_, and is possible in D like so:

    const(int*) dp1;
    const int* dp2;     // same as dp1
All of the above syntax holds true for D arrays:

    const(int)[] a1;    // mutable reference to const data
    const(int[]) a2;    // const reference to const data
    const int[] a3;     // same as a2
`const` in the examples can be replaced with `immutable`, with the caveat that initializers must match the declaration, e.g. `immutable(int)*` can only be initialized with `immutable(int)*`.

Finally, note that classes in D are reference types; the reference is baked in, so applying `const` or `immutable` to a class reference is always equivalent to `const(p*)` and there is no equivalent to `const(p)*` for classes. Structs in D, on the other hand, are value types, so pointers to structs can be declared like the `int` pointers in the examples above.


## A constant example


For the sake of argument, we assume that updating the currency exchange rates is a function that, by definition, needs to mutate the order. After updating the order, we want to show the updated prices to our user. Conceptually, the display function should not modify the order. We can prevent mutation by adding the `const` modifier to our function parameter. The implicit rule is now made explicit: the function takes an `Order` as input, and treats the object as a constant. We no longer have a gentleman’s agreement, but a formal contract. Changing the contract will, hopefully, require thorough negotiation with your peer reviewer.

```d
void updateExchangeRates(Order order);
void display(const Order order);

void updateAndDisplay()
{
    Order order = //…
    updateExchangeRates(order);
    display(order); // Implicitly converted to const.
}
```
As with all contracts, defining it is the easiest part. The hard part is enforcing it. The D compiler is our friend in this problem. If we try to compile the code, we will get a compiler error.

```d
void display(const Order order)
{
    // ERROR: cannot call mutable method
    auto t = order.getTransactions();
    show(t);
}
```
We never explicitly stated that `getTransactions` doesn’t modify the object. As the method is virtual by default, the compiler cannot derive the behavior either way. Without that knowledge, the compiler is required to assume that the method might modify the object. In other words, in the D justice system one is guilty until proven innocent. Let’s prove our innocence by marking the method itself `const`, telling the compiler that we do not intend to modify our data.

```d
public class Order
{
    private Transaction[] _transactions;

    public Transaction[] getTransactions() const
    {
        _transactions = calculate(); // ERROR: cannot mutate field
        return _transactions;
    }
}

void display(const Order order)
{
    auto t = order.getTransactions(); // Now compiles :)
    show(t);
}
```
By marking the method `const`, the original compile error has moved away. The promise that we do not modify any object state is part of the method signature. The compiler is now satisfied with the method call in the `display` function, but finds another problem. Our getter, which we stated should not modify data, actually does modify it. We found our code smell by formalizing our guidelines and letting the compiler figure out the rest.

It seems promising enough to try it on a real project.


## A constant application


I had a pet project lying around and decided to put the effort into enforcing the constraint. This is what inspired me to write this post. The project is [a four-player mahjong game](https://github.com/Zevenberge/Mahjong). The relevant part, in abstraction, is highlighted in the image.

![Mahjong abstraction](http://dlang.org/blog/wp-content/uploads/2019/03/image1.png)

The main engine behind the game is the white box in the center. A player or AI is sent a message with a `const` view of the game data for display purposes and to determine their next move. A message is sent back to the engine, which then determines the mutation on the internally mutable game data. The most obvious win is that I cannot accidentally modify the game data when drawing my UI. Which, of course, appeared to be the case before I refactored in the `const`-ness of the game data.

Upon closer inspection, coming from the UI there is only one way to manipulate the state of the game. The UI sends a message to the engine and remains oblivious of the state changes that need to be applied. This also encourages layered development and improves testability of the code. So far, so good. But during the refactoring, a problem arose. Recall that marking an object with `const` means that only member functions that promise not to modify the object, those marked with `const` themselves, can be called. Some of these could be trivially fixed by applying `const` or, at worst, `inout` ([a sort of wildcard](https://dlang.org/spec/function.html#inout-functions)). However, the more persistent issues, like in the `Order` example, required me to go back to the drawing board and rethink my domain. In the end, being forced to think about mutability versus immutability improved my understanding of my own code base.


## A constant verdict


Is `const` all good? It’s not a universal answer and certainly has downsides. The most notable one is that this kills lazy initialization where a property is computed only when first requested and then the result is cached. Sometimes, like in the earlier example, this is a code smell, but there are legit use cases. In my game, I have a class that composes the dashboard per player. Updating it is expensive and rarely required. The screen, however, gets rendered sixty times a second. It makes sense to cache the dashboards and only update them when the player objects change. I could split the method in two, but then my abstraction would leak its optimization. I settled for not using `const` here, as it was a module-private class and didn’t have a large impact on my codebase.

A complaint that is sometimes heard regarding `const` is that it does not work well with one of D’s main selling points, ranges. A range is D’s implementation of a lazy iterator, usable in `foreach` loops and heavily used in `std.algorithm`. The functions in `std.algorithm` can handle ranges with `const` elements perfectly fine. However, iterating a range changes the internal values and ultimately consumes the range. Therefore, when a range itself is `const`, it cannot be iterated over. I think this makes sense by design, but I can imagine that this behavior can be surprising in edge cases. I haven’t encountered this as I generate new ranges on every query, both on mutable and `const` objects.

A guideline for when to use `const` would be to separate the queries from the command, a.k.a. ye olde [Command-Query Separation (CQS)](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation). All queries should, in principle, be `const`. To perform a mutation, even on nested objects, one should call a method on the object. I’d double down on this and state that member functions should be commands, with logic that can be overridden, and therefore never be marked constant. Queries basically serve as a means to peek at encapsulated data and don’t need to be overridden. This should be a `final`, non-virtual, function that simply exposes a read-only view on the inner field. For example, using D’s module-private modifier on the field in conjunction with an acquainted function in the same module, we can put the logic inside the class definition and the queries outside.

```d
// in order.d
public class Order
{
    private Transaction[] _transactions; // Accessible in order.d

    public void process(); // Virtual and not restricted
}

public const(Transaction)[] getTransactions(const Order order)
{
    // Function, not virtual, operates on a read-only view of Order
    return order._transactions;
}

// in view.d
void display(const Order order)
{
    auto t = order.getTransactions();
    show(t);
}
```
We should take care, however, not to overapply the modifier. The question that we need to answer is not “Do I modify this object here?”, but rather, “Does it make sense that the object is modified here?” Wrongly assuming constant objects will result in trouble when you finally need to change the instance due to a new feature. For example, in my game a central `Game` class contains the players’ hands, but doesn’t explicitly modify them. However, given the structure of my code, it does not make sense to make the player objects constant, as free functions in the engine do use mutable player instances of the game object.

Reflecting on my design, even when writing this blog post, gave me valuable insights. Taking the effort to properly use the tool called `const` has paid off for me. It improved the structure of my code and improved my understanding of the ramblings I write. It is like any other tool not a silver bullet. It serves to formalize our gentleman’s agreement and is therefore just as fragile as one.



* * *



_Marco graduated in physics, were he used Fortran and Matlab. He used [Programming in D](http://ddili.org/ders/d.en/index.html) to learn application programming. After 3 years of being a C# developer, he is now trainer-coach of mainly Java and C# [developers at Sogyo](https://www.sogyo.nl). Marco uses D for programming experiments and side projects including his darling mahjong game._
