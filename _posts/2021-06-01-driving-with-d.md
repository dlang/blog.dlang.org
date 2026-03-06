---
author: DylanGraham
comments: false
date: 2021-06-01 11:48:12+00:00
layout: post
link: https://dlang.org/blog/2021/06/01/driving-with-d/
slug: driving-with-d
title: Driving with D
wordpress_id: 2892
categories:
- Guest Posts
- User Stories
permalink: /driving-with-d/
redirect_from: /2021/06/01/driving-with-d/
---

Here is what comes to mind when I think of D: fast, expressive, easy, and… driving? That's right, I drive with D.

![]({{ '/assets/images/driving-with-d/d3.png' | relative_url }})Enter my venerable [Holden VZ Ute](https://en.m.wikipedia.org/wiki/Holden_Commodore_(VZ)#Commercial_range) daily driver. From the factory, it came with a rubbish four-speed automatic gearbox. During 18 months of ownership, I destroyed four gearboxes. I could not afford a new vehicle at the time (I'm a 20-year-old Australian computer science student at Monash University), so I had to get creative. I purchased a rock-solid, bulletproof, six-speed automatic gearbox from another car. But that's where the solutions ended. To make it work, I had to build my own circuit board, computer system, and firmware to control the solenoids, hydraulics, and clutches inside the gearbox, handle user input, perform shifting decisions, and interface to my car by pretending to be the four-speed automatic.

I'm quite proud of my solution. It can perform a shift in 250 milliseconds, which is great for racing. It has a steep first gear, giving it a swift takeoff. It has given some more powerful cars a run for their money. It's got flappy paddles, diagnostic data on the screen, and the ability to go ahead and change the way it works whenever I want.

[Here's a very old video](https://youtu.be/tyRuq3pgebI) of the system working. It's not representative of the current system--that ghastly blue screen is gone, the speedo works, and shifting has improved.

The computer is split into two parts: the user interface board, which drives an OLED display and uses an STM32F042, and the mainboard, which handles everything else, utilizing an STM32F407. The two cooperate over a CAN bus ([Controller Area Network](https://en.wikipedia.org/wiki/CAN_bus)). All the firmware to handle this is written in D.

I picked D (as `-betterC`) because of its ingenious Uniform Function Call Syntax (UFCS), design by contract, metaprogramming, ease of interfacing with C, unit testing, portability, `shared`, `@safe`, and codebase maintenance features. Another bonus is the helpful, welcoming community. It has genuinely been a joy [discussing D on the forums](https://forum.dlang.org/), and with the founders and community leaders.



### The Advantages of D



**[Uniform Function Call Syntax (UFCS)](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs)**
This has made my code significantly clearer. My code can accurately follow the flow of data without polluting my stack with single-use variables, nesting many function calls, or other sorts of clutter.
Here is an example of how you could potentially use it in an ECU ([Engine Control Unit](https://en.wikipedia.org/wiki/Engine_control_unit)):

```d
immutable injectorTime = airStoich(100.kpa, 25.degCelsius)
    .airMass
    .fuelMass((14.7f).afr)
    .fuelMol
    .calculateInjectorWidth;
```
This is equivalent to:

```d
immutable injectorTime =
    calculateInjectorWidth(
        fuelMol(
            fuelMass(
                airMass(
                    airStoich(kpa(100), degCelsius(25))
                ),
                afr(14.7)
            )
        )
    ); // brackets have been expanded for reading clarity
```
_Please note: the values in this example are hardcoded to simplify the code and demonstrate how UFCS can give a unit of measurement to a value._

Both representations are valid D code; you can use either.

With UFCS, there's no need to read the code backward or count your brackets, no need to use a gazillion single-use variables. Function calls mirror the flow of data. It's concise.

**[Design by Contract](https://tour.dlang.org/tour/en/gems/contract-programming)**
D's contract programming is [quite similar to Ada's](https://learn.adacore.com/courses/intro-to-ada/chapters/contracts.html). Functions can be marked with preconditions, designated by `in`, and postconditions, designated by `out`. Should a contract fail, an assertion is thrown.


```d
// This demonstrates D's contract programming for a function.
// It uses the short-hand expression based syntax.
int iHaveAContract(void* ptr)
in(ptr !is null) // this is a precondition, if ptr is null, an assertion is raised
in(ptr !is null, "ptr is null :(") // this is a precondition, if ptr is null, an assertion with the error message "ptr is null :(" is raised
out(result; result > 0) // this is a post condition, it captures the return value as result and tests it
out(result; result > 0, "result was too low") // this is a post condition, it captures the return value as result, tests it, and if it fails, raise an assertion with the message "result was too low"
{
    // normal function code here
}
```
If you want to do something a bit more complex in your contracts, an alternative syntax is available:


```d
int iHaveAContract(void* ptr)
in {
    assert(ptr !is null); // this is equivalent to in(ptr !is null) from above.
    MyStruct* ms = cast(MyStruct*)ptr; // we can introduce variables local to this contract
    assert(ms.blah == 2, "MyStruct.blah must be 2!"); // test ms.blah, if it fails, raise an assert with an error message
}
out(result) { // captures the return value as result
    int squareIt = result * result;
    assert(squareIt == 4);
}
do { // this designates the function body
    // normal function code here
}

void main(string[] args)
{
    auto i = iHaveAContract(null); // this will violate the contract
}
```
[Structs]() and [classes]() can include contracts, called _invariants_, that sanity check the state of an instance for its whole lifetime. Invariants are checked after the constructor is run, before the destructor is run, and before _and_ after public function calls.


```d
struct MyStruct
{
    int blah;

    invariant {
        assert(blah == 2, "blah must always be 2 for some reason!");
    }
    // shorthand:
    invariant(blah == 2, "blah must be 2 for some reason!");

    // The invariant is checked at function entry and exit. If value is anything other than 2, the invariant will fail when the function exits
    void setBlah(int value)
    {
        blah = value;
    }
}

void main(string[] args)
{
    MyStruct s; // invariant is run after construction
    s.setBlah(3); // invariant contract will be violated.
}
```
**[Metaprogramming](https://tour.dlang.org/tour/en/gems/template-meta-programming)**
The [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) (DRY) principle is often touted by programmers. D's metaprogramming is an incredible tool to achieve that goal. I use it in my CAN bus implementation. For example:


```d
struct CANPacket(ushort ID) {
    enum id = ID;
    ubyte[8] data;
}
alias HeartbeatPacket = CANPacket!10;
alias BeepHornPacket = CANPacket!140;
```
I've got specific aliased types as `HeartbeatPacket` and `BeepHornPacket`, but I haven't needed to repeat any code. They all follow the same underlying structure, so if I modify `CANPacket`, every alias is also updated.

Maybe you want a more descriptive CAN packet? [Mixin templates](https://tour.dlang.org/tour/en/gems/template-meta-programming) can help with that!


```d
struct GenericCanPacket
{
    ushort id;
    ubyte[8] data; // 8 bytes to store the CAN packet payload
}

struct HeartbeatPacket
{
    ubyte deviceID; // first byte is the device ID
    ubyte statusID; // second byte is the status
}
```
To translate a `GenericCanPacket` to the descriptive `HeartbeatPacket`, we can use mixin templates.


```d
mixin template CanPacketHelperFunctions(ushort ID)
{
    enum id = ID;

    // typeof(this) means that the return type of readFromGeneric will be the struct this template is instantiated in.
    static typeof(this) readFromGeneric(const ref GenericCanPacket p)
    in(p.id == ID, "Generic packet cannot be converted!")
    {
        // do some cast
    }
}

struct HeartbeatPacket
{
    // the stuff declared in CanPacketHelperFunctions is pretty much copy-pasted (not literally) into here
    mixin CanPacketHelperFunctions!10;

    ubyte deviceID; // first byte is the device ID
    ubyte statusID; // second byte is the status
}

void main()
{
    GenericCanPacket generic;
    generic.id = 10;
    generic.data = [ 2 /* deviceID */, 3 /* statusID */ ];

    HeartbeatPacket heartbeat = HeartbeatPacket.readFromGeneric(generic);
    writeln(HeartbeatPacket.id); // the packet ID is 10
    writeln(heartbeat.deviceID); // 2
    writeln(heratbeat.statusID); // 3
}
```
The mixin template `CanPacketHelperFunctions` can be used over and over for all sorts of packet representations, and since it is only declared once, the implementation remains consistent across all types that use it.

**[Interfacing to C](https://dlang.org/spec/interfaceToC.html)**
I frequently must communicate with my microcontroller's HAL and RTOS; D's C interface made that a breeze. Just add an `extern(C)` and it's good to go.


```d
extern(C) c_setPwm(int solenoid, void* userData); // declaration
c_setPwm(4, null); // usage, pretty easy!
```
**[Unit Testing](https://tour.dlang.org/tour/en/gems/unittesting)**
D's built-in unit testing has saved me from blowing my foot off a few times. I can run all my unit tests on Windows to guarantee logical correctness, and then build a final target for my microcontroller. Here is an example:


```d
struct MyStruct
{
    int x;
    int squareIt() { return x * x; }
}

unittest
{
    MyStruct s;
    s.x = 9;
    assert(s.squareIt == 9 * 9); // If for some reason the implementation breaks, then the unit test fails
}
```
**[Deprecation](https://dlang.org/spec/attribute.html#deprecated)**
Codebases can be ever-changing, and sometimes certain functionality may no longer be considered good practice but must be retained for legacy reasons. D provides a way to explicitly mark this in code. Any use of such deprecated code will trigger a deprecation warning from the compiler. Example:


```d
deprecated struct Example
{
    // ...
}

// This deprecation includes a reason
deprecated("This is the reason for deprecation..") struct ExampleWithMessage
{
    // ...
}

void main()
{
    // This will generate the following compiler warning:
    // "Deprecation: struct `Example` is deprecated"
    Example e;

    // This will generate the following compiler warning:
    // "Deprecation: struct `ExampleWithMessage` is deprecated - This is the reason for deprecation.."
    ExampleWithMessage ewm;
}
```
Of course, `deprecated` can be applied to all sorts of things, not just structs.

**[Portability](https://wiki.dlang.org/Cross-compiling_with_LDC)**
Following on from above, D supports a surprisingly large number of targets via [GDC and LDC](https://dlang.org/download.html). If it weren't for D's portability, I would have had to write my project in C++ (ugh). I use LDC, and cross-compiling can be performed by simply adjusting my command line arguments.

**[Shared](https://tour.dlang.org/tour/en/multithreading/synchronization-sharing)**
`shared` is D's way of guarding against multi-threaded access of code. It's not perfect, but I use it as-is, and I think it works well. I do have multiple threads in my codebase, and they need to synchronize data. I mark certain variables as `shared`, which means I must take special care accessing that data. It works with system locks and mutexes. While locked, I can cast `shared` away and use it like a normal variable. This is handy with structs and classes.


```python
shared int sensorValue;
sensorValue = 4; // using it like a single-thread variable, error
atomicStore(sensorValue, 4); // works with atomics
```
**[SafeD](https://dlang.org/spec/function.html#safe-functions)**
`@safe` exists to prohibit sketchy memory activities and enforce best behavior. I haven't had to fight `@safe` much yet because I don't do anything wicked with my memory, but it is comfortable knowing that if I am going to make a mistake, the compiler can assist me in stopping it.

**[Mental Friction](https://www.dpldocs.info/this-week-in-d/Blog.Posted_2021_02_08.html)**
Adam D. Ruppe puts it succinctly: D has low mental friction. The flexibility and expressiveness of the language make it easy to translate one's thoughts into written code and maintain productivity. I don't have to fight D much. This is my personal opinion, but I feel like D is the language in which I'm most productive.



### Final Thoughts



D is the perfect fit for this sort of project-- I think it's going to have a bright future ahead in the embedded world. I'm going to continue using D for my projects. I've got another D-powered automotive project in the works which I hope to show off in the future. Even if D isn't yet suitable for your project, keep an eye on it. D has been making enormous strides in the past few years, especially in regards to memory safety.

_The examples shown in this article are purely meant to demonstrate how D's features can be used in the real world. Do not take them as gospel as to how **you** should program._
