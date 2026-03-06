---
author: KaiNacke
comments: false
date: 2018-04-06 13:23:05+00:00
excerpt: Many companies run their business processes on an SAP system. Often other
  applications need to access the SAP system because they provide data or request
  some calculation. There are many ways to achieve this… Let’s use D for it!
layout: post
link: https://dlang.org/blog/2018/04/06/d-goes-business/
slug: d-goes-business
title: D Goes Business
wordpress_id: 1518
categories:
- Code
- Guest Posts
permalink: /d-goes-business/
redirect_from: /2018/04/06/d-goes-business/
---

_A long-time contributor to the D community, Kai Nacke is the author of '[D Web Development](https://www.packtpub.com/web-development/d-web-development)' and the maintainer of [LDC](https://wiki.dlang.org/LDC), the LLVM D Compiler. Be sure to catch Kai at [DConf 2018 in Munich](http://dconf.org/2018/index.html), May 2 - 5, where he'll be speaking about "[D for the Blockchain](http://dconf.org/2018/talks/nacke.html)"._



* * *



Many companies run their business processes on [an SAP system](https://www.sap.com/index.html). Often other applications need to access the SAP system because they provide data or request some calculation. There are many ways to achieve this… Let’s use D for it!




### The SDK and the D binding


![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)

As an SAP customer you can download [the SAP NetWeaver Remote Function Call SDK](https://support.sap.com/en/temp/connectors/nwrfcsdk.html). You can call RFC-enabled functions in the SAP system from a native application. Conversely, it is possible from an ABAP program to call a function in a native program. The API documentation for the library is available as a separate download. I recommend downloading it together with the library.

The C/C++ interface is well structured but can be very tedious to use. That’s why I not only [created a D binding for the library](https://github.com/redstar/sapnwrfc-d) but also used some D magic to make the developer’s life much easier. I introduced the following additions:



 	
  * Most functions have a parameter of type `RFC_ERROR_INFO`. As the name implies, this is only needed in case of an error. For each of these functions a new function is generated without the `RFC_ERROR_INFO` parameter and the return type `RFC_RC` changed to `void`. Instead, a `SAPException` is thrown in case of error.

 	
  * Functions named `RfcGet*Count` now return a `size_t` value instead of using an `out` parameter. This is possible because of the above step which changed the return type to `void`.

 	
  * Functions named `Rfc*ByIndex` use a `size_t` parameter for the index, thus better integrating with D.

 	
  * Functions which have a pointer and length value now accept a D array instead.

 	
  * Use of pointers to zero-terminated UTF–16 strings is replaced with `wstring` type parameters.


Using the library is easy, just add

    dependency "sapnwrfc-d"
to [your `dub.sdl` file](https://code.dlang.org/package-format?lang=sdl). One caveat: the libraries provided by the SAP package must be installed in such a way that the linker can find them. On Windows, you can add the path to the `lib` folder of the SDK to the `LIB` and `PATH` environment variables.


### An example application


Let’s create an application calling a remote-enabled function on the SAP system. I use the `DATE_COMPUTE_DAY` function because it is simple but still has import and export parameters. This function takes a date (a string of format “YYYYMMDD”) and returns the weekday as a number (1 = Monday, 2 = Tuesday and so on).

The application needs two parameters: the system identifier of the SAP system and the date. The system identifier denotes the destination for the RFC call. The parameters for the connection must be in the `sapnwrfc.ini` file, which must be located in the same folder as the application executable. An invocation of the application using the SAP system X01 looks like this:

    D:\OpenSource\sapnwrfc-example>sapnwrfc-example.exe X01 20180316
    Date 20180316 is day 5
First, [create the application with DUB](https://code.dlang.org/getting_started):

    D:\OpenSource>dub init sapnwrfc-example
    Package recipe format (sdl/json) [json]: sdl
    Name [sapnwrfc-example]:
    Description [A minimal D application.]: An example rfc application
    Author name [Kai]: Kai Nacke
    License [proprietary]:
    Copyright string [Copyright © 2018, Kai Nacke]:
    Add dependency (leave empty to skip) []: sapnwrfc-d
    Added dependency sapnwrfc-d ~>0.0.5
    Add dependency (leave empty to skip) []:
    Successfully created an empty project in 'D:\OpenSource\sapnwrfc-example'.
    Package successfully created in sapnwrfc-example
    
    D:\OpenSource>
Let’s edit the application in `source\app.d`. Since this is only an example application, I’ve put all the code into the `main()` function. In order to use the library you simply import the `sapnwrfc` module. Most functions can throw a `SAPException`, so you want to wrap them in a `try` block.

```d
import std.stdio;
import sapnwrfc;

int main(string[] args)
{
    try
    {
        // Put your code here
    }
    catch (SAPException e)
    {
        writefln("Error occured %d %s", e.code, e.codeAsString);
        writefln("'%s'", e.message);
        return 100;
    }
    return 0;
}
```
The library uses only UTF–16. Like the C/C++ version, the alias `cU()` can be used to create a zero-terminated UTF–16 string from a D `string`. I convert the command line parameters first:

```d
    auto dest = cU(args[1]);
    auto date = cU(args[2]);
```
Now initialize the library. Most important, this function loads the `sapnwrfc.ini` file and initializes the environment. If this call is missing then it is implicitly done inside the library. Nevertheless, I recommend calling the function. It is possible that I will add more functionality to this function.

        RfcInit();
The next step is to open a connection to the SAP system. Since the connection parameters are in the `sapnwrf.ini` file, it is only necessary to provide the destination. In your own application you do not need to use the `sapnwrf.ini` file. You can provide all parameters in the `RFC_CONNECTION_PARAMETER[]` array which is passed to the `RfcOpenConnection()` function.

```d
    RFC_CONNECTION_PARAMETER[1] conParams = [ { "DEST"w.ptr, dest } ];
    auto connection = RfcOpenConnection(conParams);
    scope(exit) RfcCloseConnection(connection);
```
Please note the D features used here. A string literal is always zero-terminated, therefore there is no need to use `cU()` on the `"DEST"w` literal. With [the scope guard](https://tour.dlang.org/tour/en/gems/scope-guards), I make sure that the connection is closed at the end of the block.

Before you can make an RFC call, you have to retrieve the function meta data (function description) and create the function from it.

```d
    auto desc = RfcGetFunctionDesc(connection, "DATE_COMPUTE_DAY"w);
    auto func = RfcCreateFunction(desc);
    scope(exit) RfcDestroyFunction(func);
```
The `RfcGetFunctionDesc()` calls the SAP system to look up the meta data. The result is cached to avoid a network round trip each time you invoke this function. The implication is that the remote user needs the right to perform the look up. If this step fails with a security-related error, you should talk to your SAP administrator and review the rights of the remote user.

The `DATE_COMPUTE_DAY` function has one import parameter, `DATE`. To pass a parameter you call one of the `RfcSetXXX()` or `RfcSetXXXByIndex()` functions. The difference is that the first variant uses the parameter name (here: `"DATE"`) or the index of the parameter in the signature (here: `1`). I often use the named parameter because the resulting code is much more readable. The date data type expects an 8 character UTF–16 array.

        RfcSetDate(func, "DATE", date[0..8]);
Now the function can be called:

        RfcInvoke(connection, func);
The computed weekday is returned in the export parameter `DAY`. There is a set of `RfcGetXXX()` and `RfcGetXXXByIndex()` functions to retrieve the value.

        wchar[1] day;
        RfcGetChars(func, "DAY", day);
Let’s print the result:

        writefln("Date %s is weekday %s", date[0..8], day);
Congratulations! You’ve finished your first RFC call.

Build the application with `dub build`. Before you can run the application you still need to create the `sapnwrfc.ini` file. This looks like:

    DEST=X01
    MSHOST=x01.your.domain
    GROUP=PUBLIC
    CLIENT=001
    USER=kai
    PASSWD=secret
    LANG=EN
The SDK contains a commented `sapnwrfc.ini` file in the `demo` folder. If you are on Windows and your system still uses SAP GUI with the (deprecated) `saplogon.ini` file, then you can use the `createini` example application from my bindings library to convert the `saplogon.ini` file into the `sapnwrfc.ini` file.


### Summary


Calling an RFC function of an SAP system with D is very easy. D features like support for UTF–16 strings, scope guards, and exceptions make the source quite readable. The presented example application is part of the D bindings library and can be downloaded from GitHub at [https://github.com/redstar/sapnwrfc-d](https://github.com/redstar/sapnwrfc-d).
