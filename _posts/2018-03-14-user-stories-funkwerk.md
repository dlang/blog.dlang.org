---
author: DBlogAdmin
comments: false
date: 2018-03-14 14:06:32+00:00
excerpt: In this post, we cap off the Funkwerk series with the launch of a new feature
  we creatively call “User Stories”. Now and again, we’ll publish a post in which
  D users talk of their experiences with D, not about specific projects, but about
  the language itself. They’ll tell of things like their favorite features, why they
  use it, how it has changed the way they write code, or anything they’d like to say
  that expresses how they feel about programming in D.
layout: post
link: https://dlang.org/blog/2018/03/14/user-stories-funkwerk/
slug: user-stories-funkwerk
title: 'User Stories: Funkwerk'
wordpress_id: 1479
categories:
- Code
- Community
- Companies
- Guest Posts
- User Stories
permalink: /user-stories-funkwerk/
redirect_from: /2018/03/14/user-stories-funkwerk/
---

The deadline for the early-bird [registration for DConf 2018 in Munich](https://dlang.org/blog/2018/01/31/dconf-2018-register-now/) is coming up on March 17th. The price will go up from $340 to $400. If you’d like to go, hurry and sign up to save yourself $60. And remember, the NH Munich Messe hotel, the conference venue, is offering [a special deal on single rooms plus breakfast](https://dlang.org/blog/2018/02/23/dconf-2018-munich-the-venue/) for attendees.



* * *



![](https://i2.wp.com/www.funkwerk.com/wp-content/themes/funkwerk/library/images/logo.png?resize=250%2C81)

A few of the DConf attendees are coming from [a local company called Funkwerk](http://www.funkwerk.com). They’re a D shop that we’ve highlighted here on this blog in [a series of posts](https://dlang.org/blog/d-in-production/) about their projects (you’ll see [one of their products in action](https://dlang.org/blog/2017/07/28/project-highlight-funkwerk/) if you take the subway or local train service in Munich).

In this post, we cap off the Funkwerk series with the launch of a new feature we creatively call “User Stories”. Now and again, we’ll publish a post in which D users talk of their experiences with D, not about specific projects, but about the language itself. They’ll tell of things like their favorite features, why they use it, how it has changed the way they write code, or anything they’d like to say that expresses how they feel about programming in D.

For this inaugural post, we’ve got three programmers from Funkwerk. First up, Michael Schnelle talks about the power of ranges. Next, Ronny Spiegel tells why generated code is better code. Finally, Stefan Rohe enlightens us on Funkwerk’s community outreach.


## The power of ranges


_Michael Schnelle has been working as a software developer for about 5 years. Before starting with D 3 years ago, he worked in (Web)Application Development, mostly with Java, Ruby on Rails, and C++, and did Thread Modeling for Applications. He enjoys coding in D and likes how it helps programmers write clean code._

In my experience, no matter what I am programming, I always end up applying functions to a set of data and filter this set of data. Occasionally I also execute something with side effects in between. Let’s look at a simplified use case: the transformation of a given set of data and filtering for a condition afterwards. I could simply write:

```d
foreach(element; elements) {
  auto transformed = transform(element);
  if (metCondition(transformed) {
     results ~= transformed
  } 
}
```
Using the power from [`std.algorithm`](https://dlang.org/phobos/std_algorithm.html), I can instead write:

    filter!(element => metCondition(element))
           (map!(element => transform(element))(elements));
At this point, we have a mixture of functional and object-oriented code, which is quite nice, but still not quite as readable or easy to understand as it could be. Let’s combine it with [UFCS (Uniform Function Call Syntax)](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs):

    elements.map!(element => element.transform)
            .filter!(element => element.metCondition);
I really like this kind of code, because it is clearly self-explanatory. The `foreach` loop, on the other hand, only tells me how it is being done. If I look through our code at Funkwerk, it is almost impossible to find traditional loops.

But this only takes you one step further. In many cases, there happen to be side effects which need to be executed during the workflow of the program. For this kind of thing, the library provides functions like [`std.range.tee`](https://dlang.org/phobos/std_range.html#tee). Let’s say I want to execute something external with the transformed value before filtering:

    elements
      .map!(element => element.transform)
      .tee!(element => operation(element))
      .filter!(element => element.metCondition)
      .array;
It is crucial that operations with side effects are only executed with higher-order functions that are built for that purpose.

```d
int square(int a) { writefln("square value"); return a*a; }

[4, 5, 8]
  .map!(a => square(a))
  .tee!(a => writeln(a))
  .array;
```
The code above would print out the square value six times, because `tee` calls `range.front` twice. It is possible to avoid this by using functions like [`std.algorithm.iteration.cache`](https://dlang.org/phobos/std_algorithm_iteration.html#.cache), but in my opinion, the nice way would be to avoid side effects in functions that are not meant for that.

In the end, D gives you the possibility to combine the advantages of object-oriented and functional programming, resulting in more readable and maintainable code.


## Generated code is better code


_Ronny Spiegel has worked as a professional software developer for almost 20 years. He started out using C and C++, but when he joined Funkwerk he really started to love the D language and the tools it provides to introspect code and to automate things at compile time._

In [a previous blog post](https://dlang.org/blog/2017/09/06/the-evolution-of-the-accessors-library/), I gave a short overview of the evolution of the accessors library. As you might imagine, I really like the idea of using the compiler to generate code; in the end this usually results in less work for me and, as a direct result, causes fewer errors.

The establishment of coding guidelines is crucial for a team in order to create maintainable software, and so we have them here at Funkwerk. There is a rule that every value object (or entity) has to implement the `toString` method in order to provide diagnostic output. The provided string shall be unambiguous so that it’s more like Python’s `__repr__` than `__str__`.

Example:

    StationMessage(GeneralMessage(4711, 2017-12-12T10:00:00Z), station="BAR", …)
The generated string should follow some conventions:



 	
  * provide a way to uniquely reconstruct data from a string

 	
    * start with the class name
    
     	
    * continue with any potential superclasses
    
     	
    * list all fields providing their name and value separated by a comma
  * be compact but still human readable (for developers)

 	
    * skip the name where it matches the type (e.g. a field of type `SysTime` is called `time`)
    
     	
    * skip the name if the field is called `id` (usually there’s an `IdType` used for type safety)
    
     	
    * there’s some special output format defined for types like `Date` and `SysTime`
    
     	
    * `Nullable!T`’s will be skipped if `null` etc.
To format output in a consistent manner, we implemented a `SinkWriter` wrapping `formattedWrite` in a way that follows the listed conventions. If this `SinkWriter` is used everywhere, this is the first step to fully generate the `toString` method.

Unfortunately that’s not enough; it’s very common to forget something when adding a new field to a class. Today I stumbled across some code where a field was missing in the diagnostics output and that led to some confusion.

Using [(template) mixins](https://dlang.org/spec/template-mixin.html) together with [CTFE (Compile Time Function Execution)](https://dlang.org/spec/function.html#interpretation) and the provided type traits, D provides a powerful toolset which enables us to generate such functions automatically.

We usually implement an alternative `toString` method which uses a sink delegate as described in [https://wiki.dlang.org/Defining_custom_print_format_specifiers](https://wiki.dlang.org/Defining_custom_print_format_specifiers). The implementation is a no-brainer and looks like this:

```d
public void toString(scope void delegate(const(char)[]) sink) const
{
    alias MySelf = Unqual!(typeof(this));

    sink(MySelf.stringof);
    sink("(");

    with (SinkWriter(sink))
    {
        write("%s", this.id_);
        write("station=%s", this.station_);
        // ...
    }

    sink(")");
}
```
This code seems to be so easy that it might be generalized like this:

```d
public void toString(scope void delegate(const(char)[]) sink) const
{
    import std.traits : FieldNameTuple, Unqual;

    alias MySelf = Unqual!(typeof(this));

    sink(MySelf.stringof);
    sink("(");

    with (SinkWriter(sink))
    {
        static foreach (fieldName; FieldNameTuple!MySelf)
        {{
            mixin("const value = this." ~ fieldName ~ ";");
            write!"%s=%s"(fieldName, value);
        }}
    }

    sink(")");
}
```
The above is just a rough sketch of how such a generic function might look. For a class to use this generation approach, simply call something like

    mixin(GenerateToString);
inside the class declaration, and that’s it. Never again will a field be missing in the class’s `toString` output.

Generating the `toString` method automatically might also help us to switch from the common `toString` method to an alternative implementation. If there will be more conventions over time, we will only have to extend the `SinkWriter` and/or the `toString`-template, and that’s it.

As a summary: **Try to generate code if possible - it is less error prone and D supports you with a great set of tools!**


## Funkwerk and the D-Community


_[Stefan Rohe](https://github.com/lindt) started the D-train at Funkwerk back in 2008. They have loved DLang since then and replaced D1-Tango with D2-Phobos in 2013. They are strong believers in [open source](https://github.com/funkwerk) and [local communities](https://www.meetup.com/de-DE/Munich-D-Programmers/), and are thrilled to see you all in Munich at [DConf 2018](http://dconf.org/2018/index.html)._

Funkwerk is the largest D shop in south Germany, so we hire _D-velopers_, mainly just through being known for programming in D. In order to give a little bit back to the D community at large and help the local community grow, Funkwerk hosted the foundational edition of the [Munich D Meetup](https://www.meetup.com/de-DE/Munich-D-Programmers/).


### The local community is important …


![Munich Meetup at Brainlab](http://dlang.org/blog/wp-content/uploads/2018/03/Brainlab.png)

The meetup was founded in August 2016, 8 years after the first line of D code at Funkwerk was written. Since then, the Meetup has grown steadily to ~350 members. At that number, it is still not the biggest D Meetup, but it is the most visited and the most active. It provides a chance for locals in Munich to interact with like-minded D-interested people each month. And with an alternating level of detail and a different location each month, it stays interesting and attracts different participants.


### … and so is the global community


To engage with the global community, Funkwerk is willing to open source some of its general-purpose D libraries. They can all be found under [github.com/funkwerk](https://github.com/funkwerk), and some are registered in [the DUB registry](https://code.dlang.org/).

To mention are:



 	
  * [accessors](https://github.com/funkwerk/accessors) - a library to auto generate getters and setters with UDAs

 	
  * [depend](https://github.com/funkwerk/depend) - a tool that checks actual import dependencies against a UML model of target dependencies

 	
  * [d2uml](https://github.com/funkwerk/d2uml) - reverse engineering of D source code into PlantUML class outlines


Feel free to use these and let us know how you like them.
