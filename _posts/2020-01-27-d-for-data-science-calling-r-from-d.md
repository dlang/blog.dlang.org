---
author: LanceBachmeier
comments: false
date: 2020-01-27 14:04:43+00:00
layout: post
link: https://dlang.org/blog/2020/01/27/d-for-data-science-calling-r-from-d/
slug: d-for-data-science-calling-r-from-d
title: 'D For Data Science: Calling R from D'
wordpress_id: 2296
categories:
- Code
- D and C
- Guest Posts
permalink: /d-for-data-science-calling-r-from-d/
redirect_from: /2020/01/27/d-for-data-science-calling-r-from-d/
---

![Digital Mars D logo](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)D is a good language for data science. The advantages include a pleasant syntax, interoperability with C (in many cases as simple as adding an `#include` directive to import a C header file [via the dpp tool](https://dlang.org/blog/2019/04/08/project-highlight-dpp/)), C-like speed, a large standard library, static typing, built-in unit tests and documentation generation, and a garbage collector that’s there when you want it but can be avoided when you don’t.

Library selection for data science is a different story. Although there are some libraries available, such as [those provided by the mir project](http://code.dlang.org/packages/mir), the available functionality is extremely limited compared with languages like R and Python. The good news is that it’s possible to call functions in either language from D.

This article shows how to embed an R interpreter inside a D program, pass data between the two languages, execute arbitrary R code from within a D program, and call the R interface to C, C++, and Fortran libraries from D. Although I only provide examples for Linux, the same steps apply for Windows if you’re using WSL, and with minor modifications to the DUB package file, everything should work on macOS. Although it is possible to do so, I don’t talk about calling D functions from R, and I don’t include any discussion of interoperability with Python. (This [is normally done using pyd](http://code.dlang.org/packages/pyd).)


## Dependencies


The following three dependencies should be installed:



 	
  * R

 	
  * R package RInsideC

 	
  * R package embedr


It’s assumed that anyone reading this post already has R installed or can install it if they don’t. The RInsideC package is a slightly modified version of the excellent [RInside](http://dirk.eddelbuettel.com/code/rinside.html) project of Dirk Eddelbuettel and Romain Francois. RInside provides a C++ interface to R. The modifications provide a C interface so that R can be called from any language capable of calling C functions. [Install the package using devtools](https://github.com/r-lib/devtools):

    library(devtools)
    install_bitbucket("bachmeil/rinsidec")
[The embedr package](https://embedr.netlify.com/) provides the necessary functions to work with R from within D. That package is also installed with devtools:

    install_bitbucket("bachmeil/embedr")
## A First Program


The easiest way to do the compilation is to use [D’s package manager, called DUB](https://dub.pm/getting_started). From within your project directory, open R and create a project skeleton:

    library(embedr)
    dubNew()

This will create a `/src` subdirectory to hold your project’s source code if it doesn’t already exist, add a file called `r.d` to `/src` and create a `dub.sdl` file in the project directory. Create a file in the `/src` directory called `hello.d`, containing the following program:

```d
import embedr.r;

void main() {
  evalRQ(`print("Hello, World!")`);
}
```
From the terminal, in the project directory (the one holding `dub.sdl`, not the `/src` subdirectory), enter

```bash
dub run
```
This will print out “Hello, World!”. The important thing to realize is that even though you just used DUB to compile and run a D program, it was R that printed “Hello, World!” to the screen.


## Executing R Code From D


There are two ways to execute R code from a D program. `evalR` executes a string in R and returns the output to D, while `evalRQ` does the same thing but suppresses the output. `evalRQ` also accepts an array of strings that are executed sequentially.

Create a new project directory and run `dubNew` inside it, as you did for the first example. In the `src/` subdirectory, add a file named `reval.d`:

```d
import embedr.r;
import std.stdio;

void main() {
  // Example 1
  evalRQ(`print(3+2)`); // evaluates to 5 in R, R prints the output [1] 5 to the screen

  // Example 2
  writeln(evalR(`3+2`).scalar); // evaluates to 5 in R, output is 5

  // Example 3
  evalRQ(`3+2`); // evaluates to 5 in R, but there is no output

  // Example 4
  evalRQ([`x <- 3`, `y <- 2`, `z <- x+y`, `print(z)`]); // evaluates this code in R
}
```
Example 1 tells R to print the sum of `3` and `2`. Because we use `evalRQ`, no output is returned to D, but R is able to print to the screen. Example 2 evaluates` 3+2` in R and returns the output to D in the form of an `Robj`. `evalR(``3+2``).scalar` executes `3+2` inside R, captures the output in an `Robj`, and converts the` Robj` into a `double` holding the value `5`. This value is passed to the `writeln` function and printed to the screen. Example 3 doesn’t output anything, because `evalRQ` does not return any output, and R isn’t being told to print anything to the screen. Example 4 executes the four strings in the array sequentially, returning nothing to D, but the last tells R to print the value of `z` to the screen.

There’s not much more to say about executing R code from D. You can execute any valid R code from D, and if there’s an error, it will be caught and printed to the screen. Graphical output is automatically captured in a PDF file. To work interactively with R, or if it’s sufficient to save the results to a text file and read them into D, this is all you need to know. The more interesting cases involve passing data between D and R, and for the times when there is no alternative, using the R interface to call directly into C, C++, or Fortran libraries.


## Passing Data Between D and R


A little background is needed to understand how to pass data between D and R. Everything in R is represented as a C struct named `SEXPREC`, and a pointer to a `SEXPREC` struct is called a `SEXP` in the R source code. Those names reflect R’s origin as a Scheme dialect, where code takes the form of s-expressions. In order to avoid misunderstanding, embedr uses the name `Robj` instead of `SEXP`.

It's necessary to let R allocate the memory for any data passed to R. For instance, you cannot tell D to allocate a `double[]` array and then pass a pointer to that array to R. You would instead do something like this:

```d
auto v = RVector(100);
foreach(ii; 0..100) {
  v[ii] = 1.5*ii;
}
v.toR("vv");
evalRQ(`print(vv)`);
```
The first line tells R to allocate a vector with room for 100 elements. `v` is a D struct holding a pointer to the memory allocated by R plus additional information that allows you to read and change the elements of the vector. Behind the scenes, the `RVector` struct protects the vector from R’s garbage collector. R is a garbage collected language, and if the only reference to the data is in your D program, there’s nothing to prevent the R garbage collector from freeing that memory. The RVector struct uses the reference counting mechanism described in [Adam Ruppe’s D Cookbook](https://www.packtpub.com/application-development/d-cookbook) to protect objects from R’s garbage collector and unprotect them when they’re no longer in use.

After filling in all 100 elements of `v`, the `toR` function creates a new variable in R called `vv`, and associates it with the vector held inside `v`. The final line tells R to print out the variable `vv`.

In practice, no data is ever passed between D and R. The only thing that’s passed around is a single pointer to the memory allocated by R. That means it’s practical to call R functions from D even for very large datasets.


## Calling the R API


[The R API](//cran.r-project.org/doc/manuals/r-release/R-exts.html#The-R-API)) provides a convenient (by C standards) interface to some of R’s functions and constants, including the numerical optimization routines underlying `optim`, distribution functions, and random number generators. This example shows how to solve an unconstrained nonlinear optimization problem using the Nelder-Mead algorithm, which is the default when calling `optim` in R.

The objective function is

```python
f = x^2 + y^2
```
We want to choose `x` and `y` to minimize `f`. The obvious solution is `x=0` and `y=0`.

Create a new project directory and initialize DUB from within R, with the one additional step to add the wrapper for R’s optimization libraries:

    library(embedr)
    dubNew()
    dubOptim()

`dubOptim()` adds the file `optim.d` to the `src/` directory. Create a file called `nelder.d` inside the `src` directory with the following program:

```d
import embedr.r, embedr.optim;
import std.stdio;

extern(C) {
  double f(int n, double * par, void * ex) {
    return par[0]*par[0] + par[1]*par[1];
  }
}

void main() {
  auto nm = NelderMead(&f);
  OptimSolution sol = nm.solve([3.5, -5.5]);
  sol.print;
}
```
First we define the objective function, `f`, using the C calling convention so it can be passed to various C functions. We then create a new struct called `NelderMead`, passing a pointer to `f` to its constructor. Finally, we call the `solve` method, using `[3.5, -5.5]` as the array of starting values, and print out the solution. You’ll want to confirm that the failure code in the output is false (implying the convergence criterion was met). The most common reason that Nelder-Mead will fail to converge is because it took too many iterations. To change the maximum number of iterations to 10,000, you’d add `nm.maxit = 10_000;` to your program before the call to `nm.solve`.

There’s no overhead associated with calling an interpreted language in this example. We’re calling a C shared library directly, and at no point does the R interpreter get involved. As in the previous example, since there’s no copying of data, this approach is efficient even for large datasets. Finally, if you’re not comfortable with garbage collection, the inner loops of the optimization are done entirely in C. We nonetheless do take advantage of the convenience and safety of D’s garbage collector when allocating the `nm` and `sol` structs, as the performance advantages of manual memory management (to the extent that there are any) are irrelevant.


## Calling R Interfaces from D


The purpose of many R packages is to provide a convenient interface to a C, C++, or Fortran library. The term “R interface” normally means one of two things. For modern C or C++ code, it’s a function taking `Robj` structs as arguments and returning one `Robj` struct as the output. For Fortran code and older C or C++ code, it’s a void function taking pointers as arguments. In either case, you can call the R interface directly from D code, meaning any library with an R interface also has a D interface.

An example of an R interface to Fortran code is found in [the popular glmnet package](//cran.r-project.org/web/packages/glmnet/index.html)).
Lasso estimation using the `elnet` function is done by passing 28 pointers to the function `elnet` in `libglmnet.so` with this interface:

```
.Fortran("elnet", ka, parm=alpha, nobs, nvars, as.double(x), y,
                  weights, jd, vp, cl, ne, nx, nlam, flmin, ulam, thresh,
                  isd, intr, maxit, lmu=integer(1), a0=double(nlam),
                  ca=double(nx*nlam), ia=integer(nx), nin=integer(nlam),
                  rsq=double(nlam), alm=double(nlam), nlp=integer(1),
                  jerr=integer(1), PACKAGE="glmnet")
```
You might want to work with the R interface directly if you’re calling `elnet` inside a loop in your D program. Most of the time it’s better to pass the data to R and then call the R function that calls `elnet`. Calling Fortran functions can be error-prone, leading to hard to debug segmentation faults.


## Conclusion


D was designed from the beginning to be compatible with the C ABI. The intention was to facilitate the integration of new D code into existing C code bases. The practical result has been that, due to C’s _lingua franca_ status, D can be used in combination with myriad languages. Data scientists looking for alternatives to C and C++ when working with R may find benefit in giving D a close look.

_Lance Bachmeier is an associate professor of economics at Kansas State University and co-editor of the journal _Energy Economics_. He does research on macroeconomics and energy economics. He has been using the D programming language in his research since 2013._
