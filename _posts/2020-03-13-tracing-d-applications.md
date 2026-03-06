---
author: AlexandrDruzhinin
comments: false
date: 2020-03-13 18:52:54+00:00
layout: post
link: https://dlang.org/blog/2020/03/13/tracing-d-applications/
slug: tracing-d-applications
title: Tracing D Applications
wordpress_id: 2325
categories:
- Code
- Guest Posts
- Tutorials
permalink: /tracing-d-applications/
redirect_from: /2020/03/13/tracing-d-applications/
---

![](http://dlang.org/blog/wp-content/uploads/2018/02/bug.jpg)At one time or another during application development you need to make a decision: does your application work like it should and, if not, what is wrong with it? There are different techniques to help you decide, some of which are logging, tracing, and profiling. How are they different? One way to look at it is like this:



 	
  * when you know exactly the events you are interested in to make the decision, you use logging

 	
  * when you don’t know exactly the events you need to make a decision and you are forced to collect as many events as you can, you use tracing

 	
  * when you need to collect some events and analyze them to derive new information, you use profiling


In this article, we focus on tracing.

When developing an application, you can use tracing to monitor its characteristics at run time to, for example, estimate its performance or memory consumption. There are several options to do so, and some of them are:

 	
  * means provided by the programming language (for example, using D’s `writef`, a.k.a. `printf` debugging)

 	
  * debuggers (using scripts or remote tracing)

 	
  * OS-specific tracing frameworks (linux {k|u}probes and usdt probes, linux kernel event, performance events in windows etc)


In this article, the following contrived D example is used to help illustrate all three cases. We’ll be focusing on Linux. All example code in this article can be found in the GitHub repository at [https://github.com/drug007/tracing_post](https://github.com/drug007/tracing_post).

```d
foreach(counter; 0..total_cycles)
{
    // randomly generate one of three kinds of event
    Event event = cast(Event) uniform(0, 3);

    // "perform" the job and benchmark its CPU execution time
    switch (event)
    {
        case Event.One:

            doSomeWork;

        break;
        case Event.Two:

            doSomeWork;

        break;
        case Event.Three:

            doSomeWork;

        break;
        default:
            assert(0);
    }
}
```
`doSomeWork` simulates a CPU-intensive job by using DRuntime’s `Thread.sleep` method. This is a very common pattern where an application runs in cycles and, on every iteration, performs a job depending on the application state. Here we can see that the application has three code paths (`CaseOne`, `CaseTwo`, and `CaseThree`). We need to trace the application at run time and collect information about its timings.


## The `writef`-Based Approach


Using `writef/ln` from Phobos, [D’s standard library](https://dlang.org/phobos/index.html), to trace the application is naive, but can be very useful nevertheless. The code [from tracing_writef.d](https://github.com/drug007/tracing_post/blob/master/tracing_writef.d):

```d
    case Event.One:
            auto sw = StopWatch(AutoStart.no);
            sw.start();

            doSomeWork;

            sw.stop();
            writefln("%d:\tEvent %s took: %s", counter, event, sw.peek);
        break;
```
With this trivial approach, `StopWatch` from the standard library is used to measure the execution time of the code block of interest. Compile and run the application with the command `dub tracing_writef.d` and look at its output:

    Running ./example-writef
    0:      Event One took:   584 ms, 53 μs, and 5 hnsecs
    1:      Event One took:   922 ms, 72 μs, and 6 hnsecs
    2:      Event Two took:   1 sec, 191 ms, 73 μs, and 8 hnsecs
    3:      Event Two took:   974 ms, 73 μs, and 7 hnsecs
    ...

There is a price for this—we need to compile tracing code into our binary, we need to manually implement the collection of tracing output, disable it when we need to, and so on—and this means the size of the binary increases. To summarize:

**Pros**



 
  * all the might of Phobos is available to employ ([except when in BetterC mode](https://dlang.org/blog/2017/08/23/d-as-a-better-c/))

 	
  * tracing output can be displayed in a human readable format (look at the nice output of `Duration` above; thanks to Jonathan M. Davis for [the `std.datetime` package](https://dlang.org/phobos/std_datetime.html))

 	
  * source code is portable

 	
  * easy to use

 	
  * no third-party tools required


**Cons**



 	
  * the application must be rebuilt and restarted in order to make any changes, which is inappropriate for some applications (such as servers)

 	
  * no low-level access to the application state

 	
  * noise in the code due to the addition of tracing code

 	
  * can be unusable due to a lot of debug output

 	
  * overhead can be large

 	
  * can be hard to use in production


This approach is very suitable in the early stages of development and less useful in the final product. Although, if the tracing logic is fixed and well defined, this approach can be used in production-ready applications/libraries. For example, this approach was suggest by Stefan Koch for tracing the DMD frontend to [profile performance and memory consumption](https://github.com/dlang/dmd/pull/7792).


## Debugger-Based Approach


The debugger, in this case GDB, is a more advanced means to trace applications. There is no need to modify the application to change the tracing methodology, making it very useful in production. Instead of compiling tracing logic into the application, breakpoints are set. When the debugger stops execution on a breakpoint, the developer can use the large arsenal of GDB functionality to inspect the internal state of the _inferior_ (which, in GDB terms, usually refers to [the process being debugged](https://sourceware.org/gdb/onlinedocs/gdb/Inferiors-and-Programs.html)). It is not possible in this case to use Phobos directly, but helpers are available and, moreover, you have access to registers and the stack—options which are unavailable in the case of `writef` debugging.

Let’s take a look [the code from tracing_gdb.d](https://github.com/drug007/tracing_post/blob/master/tracing_gdb.d) for the first event:
```d
case Case.One:

    doSomeWork;

break;
```
As you can see, now there is no tracing code and it is much cleaner. The tracing logic is placed in [a separate file called trace.gdb](https://github.com/drug007/tracing_post/blob/master/trace.gdb). It consists of a series of command blocks configured to execute on specific breakpoints, like this:

    set pagination off
    set print address off
    
    break app.d:53
    commands
    set $EventOne = currClock()
    continue
    end
    
    break app.d:54
    commands
    set $EventOne = currClock() - $EventOne
    printf "%d:\tEvent One   took: %s\n", counter, printClock($EventOne)
    continue
    end
    
    ...
    
    run
    quit

In the first line, pagination is switched off. This enables scrolling so that there is no need to press “Enter” or “Q” to continue script execution when the current console fills up. The second line disables showing the address of the current breakpoint in order to make the output less verbose. Then breakpoints are set on lines 53 and 54, each followed by a list of commands (between the `commands` and `end` labels) that will be executed when GDB stops on these breakpoints. The breakpoint on line 53 is configured to fetch the current timestamp (using a helper) before `doSomeWork` is called, and the one on line 54 to get the current timestamp after `doSomeWork` has been executed. In fact, line 54 is an empty line in the source code, but GDB is smart enough to set the breakpoint on the next available line. `$EventOne` is [a convenience variable](https://www.sourceware.org/gdb/onlinedocs/gdb/Convenience-Vars.html) where we store the timestamps to calculate code execution time. `currClock()` and `printClock(long)` are helpers to let us prettify the formatting by means of Phobos. The last two commands in the script initiate the debugging and quit the debugger when it’s finished.

To build and run this tracing session, use the following commands:

```bash
dub build tracing_gdb.d --single
gdb --command=trace.gdb ./tracing-gdb | grep Event
```
`trace.gdb` is the name of the GDB script and `tracing-gdb` is the name of the binary. We use `grep` to make the GDB output look like `writefln` output for easier comparison.

**Pros**



 	
  * the code is clean and contains no tracing code

 	
  * there is no need to recompile the application to change the tracing methodology—in many cases, it’s enough to simply change the GDB script

 	
  * there is no need to restart the application

 	
  * it can be used in production (sort of)

 	
  * there is no overhead if/when not tracing and little when tracing

 	
  * [watchpoints](https://www.sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html#Set-Watchpoints) and [catchpoints](https://www.sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html#Set-Catchpoints) can be used instead of breakpoints


**Cons**



 	
  * using breakpoints in some cases may be inconvenient, annoying, or impossible.

 	
  * GDB’s pretty-printing provides “less pretty” output because of the lack of full Phobos support compared to the `writef` approach

 	
  * sometimes GDB is not available in production


The point about setting breakpoints in GDB being inconvenient is based on the fact that you can use only an address, a line number, or a function name ([see the gdb manual](https://www.sourceware.org/gdb/onlinedocs/gdb/Set-Breaks.html#Set-Breaks)). Using an address is too low level and inconvenient. Line numbers are ephemeral and can easily change when the source file is edited, so the scripts will be broken (this is annoying). Using function names is convenient and stable, but is useless if you need to place a tracing probe inside a function.

A good example of using GDB for tracing is [Vladimir Panteleev’s dmdprof](https://github.com/CyberShadow/dmdprof).


## The USDT-Based Approach


So far we have two ways to trace our application that are complimentary, but is there a way to unify all the advantages of these two approaches and avoid their drawbacks? Of course, the answer is yes. In fact there are several ways to achieve this, but hereafter only one will be discussed: USDT (Userland Statically Defined Tracing).

Unfortunately, due to historical reasons, the Linux tracing ecosystem is fragmented and rather confusing. There is no plain and simple introduction. Get ready to invest much more time if you want to understand this domain. The first well-known, full-fledged tracing framework was DTrace, developed by Sun Microsystems (now it [is open source and licensed under the GPL](http://dtrace.org/blogs/about/)). Yes, strace and ltrace have been around for a long time, but they are limited, e.g., they do not let you trace what happens inside a function call. Today, DTrace is available on Solaris, FreeBSD, macOS, and Oracle Linux. DTrace is not available in other Linux distributions because it was initially licensed under the CDDL. In 2018, it was relicensed under the GPL, but by then Linux had its own tracing ecosystem. As with everything, Open Source has disadvantages. In this case, it resulted in fragmentation. There are now several tools/frameworks/etc. that are able to solve the same problems in different ways but somehow and sometimes can interoperate with each other.

We will [be using bpftrace](https://github.com/iovisor/bpftrace), a high level tracing language [for Linux eBPF](http://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html). In D, USDT probes are provided by [the usdt library](http://code.dlang.org/packages/usdt). Let’s start from [the code in tracing_usdt.d](https://github.com/drug007/tracing_post/blob/master/tracing_usdt.d):
```d
case Case.One:
	mixin(USDT_PROBE!("dlang", "CaseOne", kind));

	doSomeWork;

	mixin(USDT_PROBE!("dlang", "CaseOne_return", kind));
break;
```
Here [we mixed in](https://dlang.org/spec/expression.html#mixin_expressions) two probes at the start and the end of the code of interest. It looks similar to the first example using `writef`, but a huge difference is that there is no logic here. We only defined two probes that are NOP instructions. That means that these probes have almost zero overhead and we can use them in production. The second great advantage is that we can change the logic while the application is running. That is just impossible when using the `writef` approach. Since we are using bpftrace, we need to write a script, [called bpftrace.bt](https://github.com/drug007/tracing_post/blob/master/bpftrace.bt), to define actions that should be performed on the probes:

```
usdt:./tracing-usdt:dlang:CaseOne
{
	@last["CaseOne"] = nsecs;
}

usdt:./tracing-usdt:dlang:CaseOne_return
{
	if (@last["CaseOne"] != 0)
	{
		$tmp = nsecs;
		$period = ($tmp - @last["CaseOne"]) / 1000000;
		printf("%d:\tEvent CaseOne   took: %d ms\n", @counter++, $period);
		@last["CaseOne"] = $tmp;
		@timing = hist($period);
	}
}
...
```
The first statement is the action block. It triggers on the USDT probe that is compiled in the `./tracing-usdt` executable (it includes the path to the executable) with the `dlang` provider name and the `CaseOne` probe name. When this probe is hit, then the global (indicated by the `@` sign) associative array `last` updates the current timestamp for its element `CaseOne`. This stores the time of the moment the code starts running. The second action block defines actions performed when the `CaseOne_return` probe is hit. It first checks if corresponding element in the `@last` associative array is already initialized. This is needed because the application may already be running when the script is executed, in which case the `CaseOne_return` probe can be fired before `CaseOne`. Then we calculate how much time code execution took, output it, and store it in a histogram called `timing`.

The BEGIN and END blocks at the top of `bpftrace.bt` define actions that should be performed at the beginning and the end of script execution. This is nothing more than initialization and finalization. Build and run the example with:

```bash
dub build tracing_usdt.d   --single --compiler=ldmd2 # or gdc
./tracing-usdt &                                     # run the example in background
sudo bpftrace bpftrace.bt                            # start tracing session
```
Output:

    Attaching 8 probes...
    0:	Event CaseThree took: 552 ms
    1:	Event CaseThree took: 779 ms
    2:	Event CaseTwo   took: 958 ms
    3:	Event CaseOne   took: 1174 ms
    4:	Event CaseOne   took: 1059 ms
    5:	Event CaseThree took: 481 ms
    6:	Event CaseTwo   took: 1044 ms
    7:	Event CaseThree took: 611 ms
    8:	Event CaseOne   took: 545 ms
    9:	Event CaseTwo   took: 1038 ms
    10:	Event CaseOne   took: 913 ms
    11:	Event CaseThree took: 989 ms
    12:	Event CaseOne   took: 1149 ms
    13:	Event CaseThree took: 541 ms
    14:	Event CaseTwo   took: 1072 ms
    15:	Event CaseOne   took: 633 ms
    16:	Event CaseTwo   took: 832 ms
    17:	Event CaseTwo   took: 1120 ms
    ^C
    
    
    
    @timing:
    [256, 512)             1 |@@@@@                                               |
    [512, 1K)             10 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
    [1K, 2K)               7 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                |

In the session output above there are only 18 lines instead of 20; it’s because `tracing-usdt` was started before the `bpftrace` script so the first two events were lost. Also, it's necessary to kill the example by typing `Ctrl-C` after `tracing-usdt` completes. After the `bpftrace` script stops execution, it ouputs the contents of the `timing` map as a histogram. The histogram says that one-time code execution takes between 256 and 512 ms, ten times between 512 and 1024 ms, and seven times more between 1024 and 2048 ms. These builtin statistics make using `bpftrace` easy.

**Pros**



 	
  * provides low-level access to the code (registers, memory, etc.)

 	
  * minimal noise in the code

 	
  * no need to recompile or restart when changing the tracing logic

 	
  * almost zero overhead

 	
  * can be effectively used in production


**Cons**



 	
  * learning USDT can be hard, particularly considering the state of the Linux tracing ecosystem

 	
  * requires external tools (frontends)

 	
  * OS specific


Note: GDB has had support for USDT probes since version 7.5. To use it, modify the `trace.gdb` script to set breakpoints using USDT probes instead of line numbers. That eases development because it eliminates the need to synchronize line numbers during source code modification.

Futher reading:



 	
  * [Profiling D’s Garbage Collection with Bpftrace](https://theartofmachinery.com/2019/04/26/bpftrace_d_gc.html)

 	
  * [Systemtap Probes and Gdb](https://blog.sergiodj.net/2012/10/27/gdb-and-systemtap-probes-part-2.html)




## Conclusion


<table > 

<tr >
Feature
writef
gdb
usdt
</tr>

<tbody >
<tr >

<td >pretty
printing
</td>

<td style="text-align: center;" >by means of Phobos
and other libs
</td>

<td >by means of pretty-printing
</td>

<td >limited builtins
</td>
</tr>
<tr >

<td >low-level
</td>

<td style="text-align: center;" >no
</td>

<td >yes
</td>

<td >yes
</td>
</tr>
<tr >

<td >clean code
</td>

<td style="text-align: center;" >no
</td>

<td >yes
</td>

<td >sort of
</td>
</tr>
<tr >

<td >recompilation
</td>

<td style="text-align: center;" >yes
</td>

<td >no
</td>

<td >no
</td>
</tr>
<tr >

<td >restart
</td>

<td style="text-align: center;" >yes
</td>

<td >no
</td>

<td >no
</td>
</tr>
<tr >

<td >usage
complexity
</td>

<td style="text-align: center;" >easy
</td>

<td >easy+
</td>

<td >advanced
</td>
</tr>
<tr >

<td >third-party
tools
</td>

<td style="text-align: center;" >no
</td>

<td >only debugger
</td>

<td >tracing system front end
</td>
</tr>
<tr >

<td >cross platform
</td>

<td style="text-align: center;" >yes
</td>

<td >sorta of
</td>

<td >OS specific
</td>
</tr>
<tr >

<td >overhead
</td>

<td style="text-align: center;" >can be large
</td>

<td >none
</td>

<td >can be ignored
even in production
</td>
</tr>
<tr >

<td >production ready
</td>

<td style="text-align: center;" >sometimes possible
</td>

<td >sometimes impossible
</td>

<td >yes
</td>
</tr>

</tr>
</tbody>
</table>
**Feature** descriptions:



 	
  * **pretty printing** is important if the tracing output should be read by humans (and can be ignored in the case of inter-machine data exchange)

 	
  * **low-level** means access to low-level details of the traced binary, e.g., registers or memory

 	
  * **clean code** characterizes whether additional tracing code which is unrelated to the applications’s business logic would be required.

 	
  * **recompilation** determines if it is necessary to recompile when changing the tracing methodology

 	
  * **restart** determines if it is necessary to restart the application when changing the tracing methodology

 	
  * **usage complexity** indicates the level of development experience that may be required to utilize this technology

 	
  * **third-party tools** describes tools not provided by standard D language distributions are required to use this technology

 	
  * **cross platform** indicates if this technology can be used on different platforms without changes

 	
  * **overhead** - the cost of using this technology

 	
  * **production ready** - indicates if this technology may be used in a production system without consequences


