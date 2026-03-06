---
author: VictorPorton
comments: false
date: 2019-02-10 14:12:39+00:00
layout: post
link: https://dlang.org/blog/2019/02/10/writing-a-d-wrapper-for-a-c-library/
slug: writing-a-d-wrapper-for-a-c-library
title: Writing a D Wrapper for a C Library
wordpress_id: 1950
categories:
- Code
- D and C
- Guest Posts
- Tutorials
permalink: /writing-a-d-wrapper-for-a-c-library/
redirect_from: /2019/02/10/writing-a-d-wrapper-for-a-c-library/
---

![](http://dlang.org/blog/wp-content/uploads/2019/02/hackathon.png)In porting to D a program I created for a research project, I wrote a D wrapper of a C library in an object-oriented manner. I want to share my experience with other programmers. This article provides some D tips and tricks for writers of D wrappers around C libraries.

I initially started [my research project](https://en.wikiversity.org/wiki/Automatic_transformation_of_XML_namespaces) using [the Ada 2012 programming language](http://www.ada2012.org/) (see my article “Experiences on Writing Ada Bindings for a C Library” in [Ada User Journal, Volume 39, Number 1, March 2018](http://ada-europe.org/archive/auj/auj-39-1-toc.pdf)). Due to a number of bugs that I was unable to overcome, I started looking for another programming language. After some unsatisfying experiments with Java and Python, I settled on the D programming language.


### The C Library


We have a C library, written in an object-oriented style (C structure pointers serve as objects, and C functions taking such structure pointers serve as methods). Fortunately for us, there is no inheritance in that C library.

The particular libraries we will deal with are the [Redland RDF Libraries](http://librdf.org/), a set of libraries which parse [Resource Description Framework (RDF)](https://www.w3.org/RDF/) files or other RDF resources, manages them, enables RDF queries, etc. Don’t worry if you don’t know what RDF is, it is not really relevant for this article.

The first stage of this project was to write a D wrapper over librdf. I modeled it on the Ada wrapper I had already written. One advantage I found in D over Ada is that template instantiation is easier—there’s no need in D to instantiate every single template invocation with a separate declaration. I expect this to substantially simplify the code of XML Boiler, my program which uses this library.

I wrote both raw bindings and a wrapper. The bindings translate the C declarations directly into D, and the wrapper is a new API which is a full-fledged D interface. For example, it uses D types with constructors and destructors to represent objects. It also uses some other D features which are not available in C. This is a work in progress and your comments are welcome.

The source code of my library (forked from Dave Beckett’s original multi-language bindings of his libraries) [is available at GitHub](https://github.com/vporton/redland-bindings) (currently only in the `dlang` branch). Initially, I tried some automatic parsers of C headers which generate D code. I found these unsatisfactory, so I wrote the necessary bindings myself.


### Package structure


I put my entire API into the `rdf.*` package hierarchy. I also have the `rdf.auxiliary` package and its subpackages for things used by or with my bindings. I will discuss some particular `rdf.auxiliary.*` packages below.


### My mixins


In Ada I used tagged types, which are a rough equivalent of D classes, and derived `_With_Finalization` types from `_Without_Finalization` types (see below). However, tagged types increase variable sizes and execution time.

In D I use structs instead of classes, mainly for efficiency reasons. D structs do not support inheritance, and therefore have no [virtual method table (vtable)](https://en.wikipedia.org/wiki/Virtual_method_table), but do provide constructors and destructors, making classes unnecessary for my use case (however, see below). To simulate inheritance, I use [template mixins](https://dlang.org/spec/template-mixin.html) (defined in the `rdf.auxiliary.handled_record` module) and [the `alias this` construct](https://dlang.org/spec/class.html#alias-this).

As I’ve said above, C objects are pointers to structures. All C pointers to structures have the same format and alignment (ISO/IEC 9899:2011 section 6.2.5 paragraph 28). This allows the representation of any pointer to a C structure as a pointer to an opaque struct (in the below example, `URIHandle` is an opaque struct declared as `struct URIHandle;`).

Using the mixins shown below, we can declare the public structs of our API this way (you should look into the actual source for real examples):

```d
struct URIWithoutFinalize {
    mixin WithoutFinalize!(URIHandle,
                           URIWithoutFinalize,
                           URI,
                           raptor_uri_copy);
    // …
}
struct URI {
    mixin WithFinalize!(URIHandle,
                        URIWithoutFinalize,
                        URI,
                        raptor_free_uri);
}
```
The difference between the `WithoutFinalize` and `WithFinalize` mixins is explained below.


### About finalization and related stuff


The main challenge in writing object-oriented bindings for a C library is finalization.

In the C library in consideration (as well as in many other C libraries), every object is represented as a pointer to a dynamically allocated C structure. The corresponding D object can be a struct holding the pointer (aka handle), but oftentimes a C function returns a so-called “shared handle”—a pointer to a C struct which we should not free because it is a part of a larger C object and shall be freed by the C library only when that larger C object goes away.

As such, I first define both (for example) `URIWithoutFinalize` and `URI`. Only `URI` has a destructor. For `URIWithoutFinalize`, a shared handle is not finalized. As D does not support inheritance for structs, I do it with template mixins instead. Below is a partial listing. See the above `URI` example on how to use them:

```d
mixin template WithoutFinalize(alias Dummy,
                               alias _WithoutFinalize,
                               alias _WithFinalize,
                               alias copier = null)
{
    private Dummy* ptr;
    private this(Dummy* ptr) {
        this.ptr = ptr;
    }
    @property Dummy* handle() const {
        return cast(Dummy*)ptr;
    }
    static _WithoutFinalize fromHandle(const Dummy* ptr) {
        return _WithoutFinalize(cast(Dummy*)ptr);
    }
    static if(isCallable!copier) {
        _WithFinalize dup() {
            return _WithFinalize(copier(ptr));
        }
    }
    // ...
}


mixin template WithFinalize(alias Dummy,
                            alias _WithoutFinalize,
                            alias _WithFinalize,
                            alias destructor,
                            alias constructor = null)
{
    private Dummy* ptr;
    @disable this();
    @disable this(this);
    // Use fromHandle() instead
    private this(Dummy* ptr) {
        this.ptr = ptr;
    }
    ~this() {
        destructor(ptr);
    }
    /*private*/ @property _WithoutFinalize base() { // private does not work in v2.081.2
        return _WithoutFinalize(ptr);
    }
    alias base this;
    @property Dummy* handle() const {
        return cast(Dummy*)ptr;
    }
    static _WithFinalize fromHandle(const Dummy* ptr) {
        return _WithFinalize(cast(Dummy*)ptr);
    }
    // ...
}
```
I’ve used [template alias parameters](https://dlang.org/spec/template.html#aliasparameters) here, which allow a template to be parameterized with more than just types. The `Dummy` argument is the type of the handle instance (usually an opaque struct). The `destructor` and `copier` arguments are self-explanatory. For the usage of the `constructor` argument, see the real source (here it is omitted).

The `_WithoutFinalize` and `_WithFinalize` template arguments should specify the structs we define, allowing them to reference each other. Note that the `alias this` construct makes `_WithoutFinalize` essentially a base of `_WithFinalize`, allowing us to use all methods and properties of `_WithoutFinalize` in `_WithFinalize`.

Also note that instances of the `_WithoutFinalize` type may become invalid, i.e. it may contain dangling access values. It seems that there is no easy way to deal with this problem because of the way the C library works. We may not know when an object is destroyed by the C library. Or we may know but be unable to appropriately “explain” it to the D compiler. Just be careful when using this library not to use objects which are already destroyed.


### Dealing with callbacks


To deal with C callbacks (particularly when accepting a `void*` argument for additional data) in an object-oriented way, we need a way to convert between C `void` pointers and D class objects (we pass D objects as C “user data” pointers). D structs are enough (and are very efficient) to represent C objects like librdf library objects, but for conveniently working with callbacks, classes are more useful because they provide good callback machinery in the form of virtual functions.

First, the D object, which is passed as a callback parameter to C, should not unexpectedly be moved in memory by the D garbage collector. So I make them descendants of this class:

```d
class UnmovableObject {
    this() {
        GC.setAttr(cast(void*)this, GC.BlkAttr.NO_MOVE);
    }
}
```
Moreover, I add the property `context()` to pass it as a `void*` pointer to C functions which register callbacks:

```d
abstract class UserObject : UnmovableObject {
    final @property void* context() const { return cast(void*)this; }
}
```
When we create a callback we need to pass a D object as a C pointer and an `extern(C)` function defined by us as the callback. The callback receives the pointer previously passed by us and in the callback code we should (if we want to stay object-oriented) convert this pointer into a D object pointer.

What we need is a bijective (“back and forth”) mapping between D pointers and C `void*` pointers. This is trivial in D: just use the `cast()` operator.

How to do this in practice? The best way to explain is with an example. We will consider how to create an I/O stream class which uses the C library callbacks to implement it. For example, when the user of our wrapper requests to write some information to a file, our class receives _write_ message. To handle this message, our implementation calls our virtual function `doWriteBytes()`, which actually handles the user’s request.

```d
private immutable DispatcherType Dispatch =
    { version_: 2,
      init: null,
      finish: null,
      write_byte : &raptor_iostream_write_byte_impl,
      write_bytes: &raptor_iostream_write_bytes_impl,
      write_end  : &raptor_iostream_write_end_impl,
      read_bytes : &raptor_iostream_read_bytes_impl,
      read_eof   : &raptor_iostream_read_eof_impl };


class UserIOStream : UserObject {
    IOStream record;
    this(RaptorWorldWithoutFinalize world) {
        IOStreamHandle* handle = raptor_new_iostream_from_handler(world.handle,
                                                                  context,
                                                                  &Dispatch);
        record = IOStream.fromNonnullHandle(handle);
    }
    void doWriteByte(char byte_) {
        if(doWriteBytes(&byte_, 1, 1) != 1)
            throw new IOStreamException();
    }
    abstract int doWriteBytes(char* data, size_t size, size_t count);
    abstract void doWriteEnd();
    abstract size_t doReadBytes(char* data, size_t size, size_t count);
    abstract bool doReadEof();
}
```
And for example:

```d
int raptor_iostream_write_bytes_impl(void* context, const void* ptr, size_t size, size_t nmemb) {
    try {
        return (cast(UserIOStream)context).doWriteBytes(cast(char*)ptr, size, nmemb);
    }
    catch(Exception) {
        return -1;
    }
}
```
### More little things


I “encode” C strings (which can be `null`) as a D template instance, `Nullable!string`. If the string is `null`, the holder is empty. However, it is often enough to transform an empty D string into a `null` C string (this can work only if we don’t differentiate between empty and `null` strings). See `rdf.auxiliary.nullable_string` for an actually useful code.

I would write a lot more advice on how to write D bindings for a C library, but you can just follow my source, which can serve as an example.


### Static if


One thing which can be done in D but not in Ada is compile-time comparison [via `static if`](https://dlang.org/spec/version.html#staticif). This is a D construct (similar to but more advanced than C conditional preprocessor directives) which allows conditional compilation based on compile-time values. I use `static if` with my custom `Version` type to enable/disable features of my library depending on the available features of the version of the base C library in use. In the following example, `rasqalVersionFeatures` is a D constant defined in my `rdf.config package`, created by the GNU configure script from the `config.d.in` file.

```d
static if(Version(rasqalVersionFeatures) >= Version("0.9.33")) {
    private extern extern(C)
    QueryResultsHandle* rasqal_new_query_results_from_string(RasqalWorldHandle* world,
                                                             QueryResultsType type,
                                                             URIHandle* base_uri,
                                                             const char* string,
                                                             size_t string_len);
    static create(RasqalWorldWithoutFinalize world,
                  QueryResultsType type,
                  URITypeWithoutFinalize baseURI,
                  string value)
    {
        return QueryResults.fromNonnullHandle(
            rasqal_new_query_results_from_string(world.handle,
                                                 type,
                                                 baseURI.handle,
                                                 value.ptr, value.length));
    }
}
```
### Comparisons


Order comparisons between structs can be easily done with this mixin:

```d
mixin template CompareHandles(alias equal, alias compare) {
    import std.traits;
    bool opEquals(const ref typeof(this) s) const {
        static if(isCallable!equal) {
          return equal(handle, s.handle) != 0;
        } else {
          return compare(handle, s.handle) == 0;
        }
    }
    int opCmp(const ref typeof(this) s) const {
      return compare(handle, s.handle);
    }
}
```
Sadly, this mixin has to be called in both the `_WithoutFinalization` and the `_WithFinalization` structs. I found no solution to write it once.


### Conclusion


I’ve found that D is a great language for writing object-oriented wrappers around C libraries. There are some small annoyances like using class wrappers around structs for callbacks, but generally, D wraps up around C well.



* * *



_Victor Porton is an open source developer, a math researcher, and a Christian writer. He earns his living as a programmer._
