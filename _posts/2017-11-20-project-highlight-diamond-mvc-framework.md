---
author: DBlogAdmin
comments: false
date: 2017-11-20 14:32:50+00:00
excerpt: According to its website, Diamond is “a full-stack, cross-platform MVC/Template
  Framework” that’s “inspired by ASP.NET and uses vibe.d for its backend”. Jacob Jensen,
  the project’s author, explains.
layout: post
link: https://dlang.org/blog/2017/11/20/project-highlight-diamond-mvc-framework/
slug: project-highlight-diamond-mvc-framework
title: 'Project Highlight: Diamond MVC Framework'
wordpress_id: 1236
categories:
- Project Highlights
- Web Development
permalink: /project-highlight-diamond-mvc-framework/
redirect_from: /2017/11/20/project-highlight-diamond-mvc-framework/
---

![](http://dlang.org/blog/wp-content/uploads/2017/11/logo.png)

Anyone who has been around the D community for longer than an eye blink will have heard of [vibe.d](https://vibed.org/), undoubtedly the most widely-used web application framework written in the D programming language. Those same people could be excused if they haven’t yet heard of [Diamond](https://diamondmvc.github.io/Diamond/), announcements for which have only started showing up in the [D forums](https://forum.dlang.org/) relatively recently.

According to its website, Diamond is “a full-stack, cross-platform MVC/Template Framework” that’s “inspired by ASP.NET and uses vibe.d for its backend”. Jacob Jensen, the project’s author, explains.


I have always been interested in web development, so one of the few projects I started writing in D was a web server using Phobos's `std.socket`. It wasn’t a notable project or anything, but more an experiment. Then I discovered vibe.d and toyed around with it. Unfortunately, coming from a background working with ASP.NET using [Razor](https://www.w3schools.com/asp/razor_intro.asp), I wasn’t a big fan of [the Diet templates](http://vibed.org/docs#html-templates). So initially I started Diamond as an alternative template engine. However, I ended up just adding more and more to the project and it soon became more and more independent. At this point, you don’t really write a lot of vibe.d code when using it, because the most general vibe.d features have wrappers in Diamond to better interact with the rest of the project.


Development on Diamond began in early 2016, but he put it aside a few months later. Then in October of 2017, after picking up some contract web app work, Diamond was resurrected in its 2.0 form.


I decided I wanted to use D, so I simply did a complete revamp of the project and plan to keep maintaining it.


His biggest hurdle has been keeping the design of the framework user-friendly and minimizing complex interactions between controllers, views, and models.


It was a big challenge to ensure that everything worked together in a way that made it feel natural to work with. On top of that I had to make sure that Diamond worked under multiple build types. At first Diamond was written solely with the web in mind and thus only supported websites and web APIs, but I saw the potential to use the template engine for more than just the web, like email templates. Thus I introduced yet a third way of building Diamond applications, which had to be completely separated from the web part of Diamond without introducing complexity into user code or the build process. By introducing stand-alone support, Diamond is now able to be used with existing projects that aren’t already using it for the web, e.g. someone could use Diamond to extend their existing web pages without having to switch the whole project to Diamond, or simply use Diamond for only a small portion of it.


Aside from the challenge of maintaining user-friendliness, Jacob says he’s encountered only a few issues in developing the framework, most of which came when he was refactoring for the 2.0 release. One in particular is interesting, as his solution for it was a D language feature you don’t often hear much about.


When I was introducing attributes to controllers to avoid manual mapping of actions, it took a while to figure out the best approach to it without having to pass additional information about the controller to its base class.


To demonstrate, `Controller` subclasses could originally be declared like so:

```d
class MyController(TView) : Controller!TView
{
    ...
}
```
After he initially added attributes, the refactoring required the base class template to know the derived type at compile time in order to reflect on its attributes. His initial solution required that subclasses specify their own types as an additional template parameter to the `Controller` template in the class declaration.

```d
class MyController(TView) : Controller!(TView, MyController!TView)
{
    ...
}
```
He didn’t like it, but it was the only way he could see to make the base class aware of the derived type. Then he discovered D’s [template this parameters](https://dlang.org/spec/template.html#TemplateThisParameter).

_Template this_ parameters allow any templated member function to know at compile time the static, i.e. declared, type of the instance on which the function is being called.

```d
module base;
class Base 
{
    void printType(this T)() 
    {
        import std.stdio;
        writeln(typeid(T));
    }
}

class Derived : Base {}

void main()
{
    Derived d1 = new Derived;
    auto d2 = new Derived;
    Base b1 = new Derived;

    d1.printType();
    d2.printType();
    b1.printType();
}
```
And this prints (in `modulename.TypeName` format):

    base.Derived
    base.Derived
    base.Base
In Diamond, this is used in the `Controller` constructor in order to parse the UDAs ([User Defined Attributes](https://dlang.org/spec/attribute.html#UserDefinedAttribute)) attached to the derived type at compile time:

```d
class Controller(TView) : BaseController
{
    ...
    this(this TController)(TView view)
    {
        ...

        static if (hasUDA!(TController, HttpAuthentication))
        {
            ...
        }

        static if (hasUDA!(TController, HttpVersion))
        {
            ...
        }
        ...
    }
}
```
The caveat, and the price Jacob is willing to pay for the increased convenience to users, is that instances of derived types should never be declared to have the type of the base class. When working with templated types in D, it’s idiomatic to use type inference anyway:

```d
// This won't pick up the MyController attributes, as the declared
// type is that of the base class
Controller!ViewImpl controller1 = new MyController!ViewImpl;

// But this will
MyController!ViewImpl controller3 = new MyController!ViewImpl;

// And so will this -- it's also more idiomatic
auto controller2 = new MyController!ViewImpl;
```
Overall, Jacob has found the transition from C# to D fairly painless.


Most code I was used to writing, coming from C#, is pretty straight-forward in D. One of the pros of D, however, is its compile-time functionality. I use it heavily in Diamond to parse templates, map routes and controller actions, etc. It’s a really powerful tool in development and probably the most powerful tool in D. I also really like templates in D. They’re implemented in a way that doesn’t make them seem complex, unlike in C++, where templates can often seem obscure and cryptic. D is probably the most natural programming language that I’ve used.


Diamond indirectly supports Mongo and Redis through vibe.d, and has its own MySQL ORM interface that uses the native MySQL library under the hood. He has some plans improve upon the database support, however.


I plan to rewrite the whole MySQL part, since it currently uses some deprecated features – it was based on some old code I had been using. Along with that, I plan on implementing some “generic services” that can be used to create internal services in the project, which will of course wrap database engines such as MySQL, Mongo, Redis, etc., creating a similar API between them all and exposing an easier way to implement sharding.


He also intends to add support for textual data formats other than JSON (such as XML) to make Diamond compatible with SOAP or WCF services, add improved support for components in the view, and provide better integration with JavaScript. He also would like to implement an app server for hosting Diamond applications.

Anyone intending to use D for web work today who, like Jacob, has experience using ASP.NET and Razor should feel right at home using Diamond. For the rest, it’s an alternative to using vibe.d directly that some may find more comfortable. You can find [the Diamond source](https://github.com/DiamondMVC/Diamond), [the current documentation](https://diamondmvc.github.io/Diamond/), and [the in-development official website](https://github.com/DiamondMVC/Diamond-website) (for which Jacob is dog-fooding Diamond) all [at GitHub](https://github.com/DiamondMVC).
