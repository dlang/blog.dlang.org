---
author: DBlogAdmin
comments: false
date: 2016-10-07 13:41:39+00:00
layout: post
link: https://dlang.org/blog/2016/10/07/project-highlight-dlangui/
slug: project-highlight-dlangui
title: 'Project Highlight: DlangUI'
wordpress_id: 314
categories:
- GUIs
- Project Highlights
permalink: /project-highlight-dlangui/
redirect_from: /2016/10/07/project-highlight-dlangui/
---

Vadim Lopatin is an active D user who, like many in the D community, comes from a Java and C++ background.


My current job is writing a Java backend for a virtual call center . I've also worked as a C++ developer on [IP PBX devices](https://www.google.co.kr/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwjssfeqhMjPAhXGH5QKHV4lC-sQFggtMAE&url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FIP_PBX&usg=AFQjCNGQZJ3F3VRnf0RRx4bi1v_QlEOOAw&sig2=pJ8fibD9N2VN7CsfPUw9gQ). Programming is my hobby as well. My biggest hobby project, which I've rewritten from scratch twice in the last 15 years, is [CoolReader](https://github.com/buggins/coolreader), a cross platform e-book reader written in C++.


He kept hearing news about D and, over time, became more interested in its "cool features", like [CTFE](https://tour.dlang.org/tour/en/gems/compile-time-function-evaluation-ctfe) and [code generation](http://ddili.org/ders/d.en/mixin.html). So, three years ago, he decided to initiate a couple of projects to learn its features.


[DDBC](https://github.com/buggins/ddbc) is a database connector similar to Java's [JDBC](http://www.oracle.com/technetwork/java/overview-141217.html), with an API close to the original. [HibernateD](https://github.com/buggins/hibernated) is an [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) library, similar to the Java-based [Hibernate](http://hibernate.org/). Unlike Java, D allows the use of compile-time code introspection and code generation. It was interesting work, and I was impressed by the power of D.


Both projects proved to be no more than learning exercises, however, as he never used either himself and neither became popular in the community. Now they are largely abandoned, but he has since found another area where he could apply his talents and, as it turns out, where community interest has been much higher.

The new idea came about as he surveyed the state of available GUI libraries in D. While there were [several options](http://wiki.dlang.org/Libraries_and_Frameworks) to choose from, he wasn't satisfied by the fact that they were all either non-native wrappers or not cross-platform. He had already written a cross-platform GUI in C++ for CoolReader GL, a version of his ebook reader that uses the same GUI on all supported platforms. Why not implement another one in D?

He has a long list of items he thinks are important for a GUI library to check off. A few of them are:




> 
> 
 	
>   * Cross-platform -- the same code should work on all platforms with simple recompilation.
> 
 	
>   * Internationalization -- it should be easy to write multilingual apps. Unicode everywhere. Strings externalized to resources.
> 
 	
>   * Hardware acceleration -- take advantage of DirectX or OpenGL where available, but it should be possible to use software rendering where they aren't.
> 
 	
>   * Resolution independence -- flexible layouts must be used instead of fixed pixel-by-pixel positioning of controls.
> 




A markup language for describing layouts, touch screen support, 3D rendering, customizable look-and-feel, easy event handling, and several other items complete the list. A big set of requirements for one person to work on alone, but he already had a good deal of experience with the GUI he wrote for CoolReader. So when he got going with his [DlangUI](https://github.com/buggins/dlangui) project, his previous work is where he started.


Part of DlangUI is a direct port of the CoolReader GL GUI. It was easy to reuse big parts of C++ code thanks to the similarity of D and C++ syntax.


So he set about checking items off of his list. Such as support for hardware acceleration via an interface that easily supports different rendering backends, one of which is implemented using OpenGL. But as things got under way, he discovered that there is one particular issue with porting C++ to D that arises in the parts that can't be directly reused.


The D GC does not bring any help for resource management, since object destructors may be called in any thread, in any order, or never at all. If an object owns some resources, it ought to be destroyed in a predictable way. Therefore, widgets and other objects holding resources must be destroyed manually by their owners

DlangUI uses reference counting for easy freeing of owned objects. Widgets remove their children on destroy. Windows remove their widgets when closing. I had to add debug mode instance counts for various objects, and corresponding messages in the log, to make sure all resources are freed gracefully.

Some resources (e.g. images) are cached. Their references may be taken from the cache, used, and then released often. To allow cleanup of caches, all such resources have usage flags. The cache provides a checkpoint method which removes the usage flag from all items, and a cleanup method which frees all cache items which have not been used since the last checkpoint.


He has worked on a number of items from his list, such as theme customization.


DlangUI themes are inspired by the Android API. It borrows Android's state drawables (they may be even used as is), nine-patch PNGs, and resource versions for different screen sizes or resolutions. Usually widgets don't use a hardcoded look and feel or layout properties. Instead, they use a style ID referencing to currently selected theme. If the theme is changed in runtime, all widgets receive a corresponding notification so that they can reload any cached values from the new theme. Simply providing a new theme changes the  look and feel significally.

Currently, two standard themes are provided in DlangUI: default (light) and dark. Applications may specify a standard theme as a parent, and override only the styles it needs. Standard theme resources are usually embedded into the application executable using the cool D feature `import("filename")`. Applications may embed their own resources as well. This allows creating a single file app withoug any additional resource files needing to be shipped with the executable.


Another check mark can  be place next to layouts. Here, he again looked to Android.


To support multiple screen resolutions and sizes, widgets must be placed and resized using layouts instead of direct pixel-based positioning. DlangUI uses Android API-like layouts for grouping, placing and resizing widgets, based on a two-phase measure/layout scheme.


And, while a GUI can be assembled entirely in code, he took inspiration from elsewhere for ideas to knock the markup item off his list.


Manually writing code to create a widget hierarchy and setting their properties is a bit boring. DlangUI offers the possibility to create widgets using a JSON-based description similar to [Qt QML](http://doc.qt.io/qt-5/qtqml-index.html). I call it DML. Currently, only the creation of widgets and the setting of their properties are supported. In future, I hope to add the ability to describe signal handlers in DML, and automatically assign signals to handlers, and widget instances to variables. There is a GUI app, [dlangui:dmledit](https://github.com/buggins/dlangui/tree/master/examples/dmledit), which helps to write DML. It combines a text editor for DML and a preview window to see the results.


When it comes to being cross-platform, a lot has been done so far, thanks to different backend implementations: Win32, [SDL2](https://www.libsdl.org/download-2.0.php), [DSFML](http://dsfml.com/), X11, and Android. Not long ago, Vadim even [announced a text-based interface](http://forum.dlang.org/thread/lbilqqcqbojzfqhanjdf@forum.dlang.org) which works in the Linux terminal or Windows console.


It was a real surprise for me how few changes were required to implement text-mode support. Besides the backend code and the text-mode drawing buffer implementation, most of the changes came in the form of a Console theme. Only a few fixes were required in the widgets, removing several hardcoded margins and sizes. Even [DlangIDE](https://github.com/buggins/dlangide), a DlangUI-based IDE for the D programming language, is now usable in terminals.


Here's what DlangUI's components normally look like on Windows.
![screenshot-example1-windows](http://dlang.org/blog/wp-content/uploads/2016/10/screenshot-example1-windows.png)

And this is what DlangIDE looks like running in the Windows console.

![dlangide](http://dlang.org/blog/wp-content/uploads/2016/10/dlangide.png)

When compared to [screenshots](http://buggins.github.io/dlangui/screenshots.html) of programs running with different DlangUI backends, seeing it in a terminal like that is pretty darn cool.

DlangUI manages event handling via signals and has built-in support for 3D graphics, including [a 3D scene package](https://github.com/buggins/dlangui/tree/master/src/dlangui/graphics/scene). And work still continues on making Vadim's list smaller, as well as addressing the problems with the library.


The most mentioned issue the non-native look and feel of widgets. Although it's possible to make a theme looking exactly like native one, it would not track system theme changes anyway. There's no system menu support on OS X and in Gnome (where a common menu is used for all apps). The documentation is poor. There is some DDOX-generated documentation, but it's not detailed enough and I seldom update. I need more tutorials and examples. And some advanced controls are missing, e.g. an HTML view.


He also says that there are too few developers working on the project. While some users have submitted PRs, the majority of the work has been done by Vadim alone. Given what he has produced so far, that's a pretty impressive achievement. But, in addition to solving the problems above, he's got a lot more he wants to implement, such as:




> 
> 
 	
>   * An XML+CSS rendering widget to show/edit HTML or rich text
> 
 	
>   * Refactoring DlangUI to extract window creation, OpenGL context creation, drawing, font support, input events code from widget set - for cases when no widgets are needed
> 
 	
>   *  Mobile platforms support improvements - add iOS backend, improve android support, improve touch mode support
> 
 	
>   *  Native system menu support on OS X and Gnome
> 
 	
>   *  Support for fallback fonts in font engines, from which to get missing symbols
> 
 	
>   *  A native OSX backend based on Cocoa instead of libSDL2
> 
 	
>   * Improvements in Scene3D to make it suitable for writing 3D games
> 




If you need a GUI for your D app, [DlangUI](https://github.com/buggins/dlangui) is a viable option today. More importantly, if you're able and willing to help out here and there, Vadim sure could use a few more hands on a few more keyboards!






