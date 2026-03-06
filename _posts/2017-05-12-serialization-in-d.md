---
author: VladimirPanteleev
comments: false
date: 2017-05-12 15:33:58+00:00
layout: post
link: https://dlang.org/blog/2017/05/12/serialization-in-d/
slug: serialization-in-d
title: Serialization in D
wordpress_id: 754
categories:
- Code
- Core Team
- Guest Posts
permalink: /serialization-in-d/
redirect_from: /2017/05/12/serialization-in-d/
---

_Vladimir Panteleev has spent over a decade using and contributing to D. He is the creator and maintainer of [DFeed](https://github.com/CyberShadow/DFeed), the software powering the D forums, has made numerous contributions to [Phobos](https://github.com/dlang/Phobos), [DRuntime](https://github.com/dlang/druntime), [DMD](https://github.com/dlang/dmd), and [the D website](https://github.com/dlang/dlang.org), and has created several tools useful for maintaining D software (like [Digger](https://github.com/CyberShadow/Digger) and [Dustmite](https://github.com/CyberShadow/DustMite))._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)A few days ago, I saw this blog post by Justin Turpin on the front page of Hacker News:

[The Grass is Always Greener - My Struggles with Rust](https://compileandrun.com/stuggles-with-rust.html)

This was an interesting coincidence in that it occurred during [DConf](http://dconf.org/2017/), where I had mentioned serialization in D a few times during my talk. Naturally, I was curious to see how D stands up to this challenge.


### The Task


Justin's blog starts off with the following Python code:

```python
import configparser
config = ConfigParser()
config.read("config.conf")
```
This is actually very similar to a pattern I use in many of my D programs. For example, [DFeed](https://github.com/CyberShadow/DFeed) (the software behind [forum.dlang.org](https://forum.dlang.org/)), has [this code](https://github.com/CyberShadow/DFeed/blob/5a2fc9284f4b201be27db6cfca8750c9f9e66fbc/web.d#L4027-L4042) for configuring its built-in web server:

```d
struct ListenConfig
{
    string addr;
    ushort port = 80;
}

struct Config
{
    ListenConfig listen;
    string staticDomain = null;
    bool indexable = false;
}
const Config config;

import ae.utils.sini;
shared static this() { config = loadIni!Config("config/web.ini"); }
```
This is certainly more code than the Python example, but that's only the case because I declare the configuration as a D type. The `loadIni` function then accepts the type as a template parameter and returns an instance of it. The strong typing makes it easier to catch typos and other mistakes in the configuration - an unknown field or a non-numeric value where a number is expected will immediately result in an error.

On the last line, the configuration is saved to a global by a static constructor (`shared` indicates it runs once during program initialization, instead of once per thread). Even though `loadIni`'s return type is mutable, D allows the implicit conversion to `const` because, as it occurs in a static constructor, it is treated as an initialization.


### Traits


The Rust code from Justin's blog is as follows:

```rust
#[macro_use]
extern crate serde_derive;
extern crate toml;

#[derive(Deserialize)]
struct MyConfiguration {
  jenkins_host: String,
  jenkins_username: String,
  jenkins_token: String
}

fn gimme_config(some_filename: &str) -> MyConfiguration {
  let mut file = File::open(some_filename).unwrap();
  let mut s = String::new();
  file.read_to_string(&mut s).unwrap();
  let my_config: MyConfiguration = toml::from_str(s).unwrap();
  my_config
}
```
The first thing that jumps out to me is that the `MyConfiguration` struct is annotated with `#[derive(Deserialize)]`. It doesn't seem optional, either - quoting Justin:


This was something that actually really discouraged me upon learning, but you cannot implement a trait for an object that you did not also create. That's a significant limitation, and I thought that one of the main reason Rust decided to go with Traits and Structs instead of standard classes and inheritance was for this very reason. This limitation is also relevant when you're trying to serialize and deserialize objects for external crates, like a MySQL row.


D allows introspecting the fields and methods of any type at compile-time, so serializing third-party types is not an issue. For example (and I'll borrow [a slide from my DConf talk](http://thecybershadow.net/d/dconf2017/#/6)), deserializing one struct field from JSON looks something like this:

```d
string jsonField = parseJsonString(s);
enforce(s.skipOver(":"), ": expected");

bool found;
foreach (i, ref field; v.tupleof)
{
    enum name = __traits(identifier, v.tupleof[i]);
    if (name == jsonField)
    {
        field = jsonParse!(typeof(field))(s);
        found = true;
        break;
    }
}
enforce(found, "Unknown field " ~ jsonField);
```
Because the `foreach` aggregate is a tuple (`v.tupleof` is a tuple of `v`'s fields), the loop will be unrolled at compile time. Then, all that's left to do is compare each struct field with the field name we got from the JSON stream and, if it matches, read it in. This is a minimal example that can be improved e.g. [by replacing the `if` statements with a `switch`](http://thecybershadow.net/d/dconf2017/#/7), which allows the compiler to optimize the string comparisons to hash lookups.

That's not to say D lacks means for adding functionality to existing types. Although D does not have struct inheritance like C++ or struct traits like Rust, it does have:



 	
  * [`alias this`](https://dlang.org/spec/class.html#alias-this), which makes wrapping types trivial;

 	
  * [`opDispatch`](https://dlang.org/spec/operatoroverloading.html#dispatch), allowing flexible customization of forwarding;

 	
  * [template mixins](https://dlang.org/spec/template-mixin.html), which allow easily injecting functionality into your types;

 	
  * finally, there is of course classic OOP inheritance if you use classes.




### Ad-lib and Error Handling


It doesn't always make sense to deserialize to a concrete type, such as when we only know or care about a small part of the schema. D's standard JSON module, `std.json`, currently only allows deserializing to a tree of variant-like types (essentially a DOM). For example:

```d
auto config = readText("config.json").parseJSON;
string jenkinsServer = config["jenkins_server"].str;
```
The code above is the D equivalent of the code [erickt posted on Hacker News](https://news.ycombinator.com/item?id=14285368):

```rust
let config: Value = serde::from_reader(file)
    .expect("config has invalid json");

let jenkins_server = config.get("jenkins_server")
    .expect("jenkins_server key not in config")
    .as_str()
    .expect("jenkins_server key is not a string");
```
As D generally uses exceptions for error handling, the checks that must be done explicitly in the Rust example are taken care of by the JSON library.


### Final thoughts


In the discussion thread for Justin's post, Reddit user SilverWingedSeraph [writes](https://www.reddit.com/r/programming/comments/69necf/the_grass_is_always_greener_my_struggles_with_rust/dh7wis7/):


You're comparing a systems language to a scripting language. Things are harder in systems programming because you have more control over, in this case, the memory representation of data. This means there is more friction because you **have** to specify that information.


This struck me as a false dichotomy. There is no reason why a programming language which has the necessary traits to be classifiable as a system programming language can not also provide the convenience of scripting languages to the extent that it makes sense to do so. For example, D provides type inference and variant types for when you don't care about strong typing, and garbage collection for when you don't care about object lifetime, but also provides the tools to get down to the bare metal in the parts of the code where performance matters.

For my personal projects, I've greatly enjoyed D's capability of allowing rapidly prototyping a design, then optimizing the performance-critical parts as needed without having to use a different language to do so.


### See also





 	
  * [Serialization libraries on the D package registry](http://code.dlang.org/search?q=serialization)

 	
  * [Dynamic Typing in D](https://wiki.dlang.org/Dynamic_typing)


