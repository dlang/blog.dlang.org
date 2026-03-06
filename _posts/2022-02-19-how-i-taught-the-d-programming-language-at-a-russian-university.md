---
author: GeorgyMarkov
comments: false
date: 2022-02-19 15:02:41+00:00
layout: post
link: https://dlang.org/blog/2022/02/19/how-i-taught-the-d-programming-language-at-a-russian-university/
slug: how-i-taught-the-d-programming-language-at-a-russian-university
title: How I Taught the D Programming Language at a Russian University
wordpress_id: 3060
categories:
- Education
- Guest Posts
- User Stories
permalink: /how-i-taught-the-d-programming-language-at-a-russian-university/
redirect_from: /2022/02/19/how-i-taught-the-d-programming-language-at-a-russian-university/
---

_This article was [originally published in Russian](https://habr.com/ru/post/522002/) by Grigorii Smorkalov. It was translated to English for the D Blog by Georgy Markov and lightly revised from the original by Michael Parker._

![]({{ '/assets/images/how-i-taught-the-d-programming-language-at-a-russian-university/gettyimages-1249804112-612x612-1-200x200.jpg' | relative_url }})

This is the fourth year I'm teaching my D Programming Language course at a very real university in Russia. It's a full-term course with lectures, practical lessons, and exams, although it's all remote now. This is the story about how I got there, the challenges I encountered, and how students sometimes surpass their teachers.



## What's in D for university students



The job market for D is very small. As I always say during the first lesson, it's unlikely that students are going to write D code for a salary, but that doesn't mean that learning D is useless. Firstly, it's much easier to learn how to program with D than with C or C++. This is important because many students don't know how to program even after a full C/C++ course. Secondly, a broader outlook makes for better code. My familiarity with D improved my C++ skills and made it much easier to learn Python, especially its iterators. Most importantly, D is the future--of C++ and beyond. Many C++11/17/20 novelties were first battle-tested in D, and even today D is a much more modern and feature-rich language than C++.



## Who am I and what's in D for me



For simplicity, I would say that I am a C++ programmer. This is my main line of work. Before this course, I'd never been a teacher of any sort. Even on my job, I'm not involved much in mentoring. After earning my master's degree in Computational Mathematics and Cybernetics at UNN (Lobachevsky State University of Nizhny Novgorod) in 2014, I had no relation to academics at all.

In my first months of university, I entertained the thought of being a school teacher. Now I realize that this was just a call for justice of sorts. There is a stark difference between a high school and a university, and for me, the latter was a much better experience. It felt like in just one month at university I learned more than in one term in high school, and that was so cool. _Why couldn't they explain it **this way** back in high school_, was my common thought. If only educators applied the same educational methods in a regular school, it would make the experience much better. My desire to become a teacher vanished the second I received my first paycheck as a programmer (in Russia, 'programmer' is a very well-paid job and 'school teacher' is the total opposite), but some memory of that desire remained.

This was also the time when I became interested in D. Compared to C++ it looked like a perfect programming language. You can write code that would be as fast, but without all those C atavisms. I used D for my master's thesis, and I loved it. My program was twice as small and simple as the older C++ version while performing better. Implementing complex and more efficient algorithms in D was much easier; doing the same in C++ would be too much work, and, like any student, I always struggled with my deadlines.

Since then I've followed D and used it for my little pet projects. I've always wanted to help the D community in some way, but I couldn't write any useful library, nor could I find the motivation for contributing to open source.



## Beginnings



In 2018, an unusual offer surfaced on the D mailing list: does anyone by any chance want to teach students in Moscow, Russia?

The initiator was Dmitry Olshansky, a well-known member of the D community, the creator of `std.regex` and more. I contacted him and said I was interested. I didn't see myself as a full-fledged lecturer and expected to be just an assistant. At the beginning that was the plan, with someone else acting as a lecturer. He was going to give lectures remotely via Skype, and I would assist him on site.

To my surprise, the university in question was RSUH, Russian State University for the _Humanities_. As it turned out, they do have technical faculty there, and the students do actually code. I checked their program: D was introduced for third-term students, and during the first two terms they learned C, C++, Prolog, and even some Lisp, I think (a bit too much, but why not). Their math course was solid, too (yes, I am among those who think that math is important for programmers).



## Preparations



I was introduced to the department staff. They explained everyone's responsibilities and even offered the opportunity to join in scientific work as a programmer. We started working on the course program, although I barely included myself in that “we”. That was a mistake. With one month left until the classes started, the lecturer was suddenly leaving us. The news took me by surprise, but… there was still plenty of time, right?

This was happening when it was time for me to complete all the formalities and start work. Though I knew they were hiring only me, for some reason I was still under the impression that I wouldn't be alone. I can't say why I thought so. Everything was saying that there would be no help and that I had to do the whole course by myself, but my impression was hard to shake. The grave realization only came one week before Day One. Only then did I start to prepare for real.



## Bureaucracy



This is supposed to be the part about the trials and tribulations of the endless bureaucracy awaiting a poor programmer's soul. The amount of paperwork required to sign the contract was indeed an entertaining story to tell to my peers. And everyone was, as they say, “rolling on the floor laughing” when I described applying for a Mir payroll card (Mir is the Russian national payment system mandated in the state-funded sector). But that's it, actually.

The next bureaucratic task was composing the formal program, an official paper including the course program and the materials. There were indeed a lot of formalities there, but I got some help: they showed me the paper for a very similar course. At the end of the day, it was easier than I expected. For this, I should thank the university staff. It was a one-year contract. Renewing it every year only takes one piece of paper and a couple of pen strokes.



## The first class



The schedule was set up such that my first class happened to be a seminar (a.k.a. a practical lesson) instead of a lecture. This is only on paper, though with only one group consisting of just 14 people it didn't make any difference. The schedule was adjusted later. I got two classes in a row and could decide which was a lecture and which was a seminar. I came up with the following arrangement: in the first class, I would lecture and answer general questions, and in the second, the students would program and ask practical questions.

For the first lesson, I prepared a brief description of the language, a syntax overview, a compiler, and several problems to solve: calculating the area of a triangle, solving a quadratic equation, and similar problems that yield a simple output for simple input. The idea was to immerse students in the language and immediately give them something to do with it. The plan was a success. By the end of the class, most of the students had gotten the hang of using a command-line compiler, written some code in a text editor, and solved some problems.



## Notepad and the command line



I bet many people would take issue with the command line and text editor part. Seriously? No IDE? IDEs for D exist, but I left it up to the students if they wanted to use them. The reason was my own experience in learning C++ and programming in general. Knowing how a compiler works and how to link several files into a project is integral to understanding the language as a whole.

This is especially true for C and C++. Things like the difference between declaration and definition lose their meaning without understanding the build process. In D, such nuances are fewer. For example, header files are not required. Still, I meant to teach how to program in D, not how to use an IDE. Things like the principles of `import` are easier to grok when using a command-line compiler rather than some “intuitive interface”. And you learn the syntax faster if you write the code by hand without autocomplete doing it for you.



## Further development



Lifted by the success of the first seminar, I was slammed to the ground by the first lecture. It contained the full theoretical explanation of type systems and their various types, compared D with other languages, brought out the problems of C and C++, and demonstrated what makes D different. It was a total failure.

I expected the lecture to take the whole 80-minute class, including questions, but it finished in less than an hour without a single question during or after the speech. I even asked if the students couldn't hear or understand me, or if there was a problem with my diction. But the problem was with the lecture itself. First, it was too much for one class. Second, the lecture was based on the talks I did for my job that were intended for seasoned programmers. I realized that everything that I'd prepared for my future lectures must be tossed aside and rewritten from scratch.



## The new program



The first candidate for simplification, and by that I mean expunging, was metaprogramming. Getting rid of templates was impossible since even the most basic language features and algorithms are tied to them, but code generation of all sorts was the first to be removed. Following that was anything that required external libraries. What was left were things that D code can't be written without.

Since I had better success with live communication during actual programming, I decided to focus on that. Rewriting the course on the fly was tricky, but I came up with a solid plan: throughout the semester we would write a complex calculating program, during lectures we would study language features required for a given task, and then during seminars the ideas would be transformed into code.



## The semester assignment



If you’re a nerd like me, you’ve probably heard of [the 10,958 problem](https://youtu.be/-ruC5A9EzzE). The gist of it is that you need to put signs and parentheses between numbers composed of the digits 1 to 9 so that the result would be exactly 10,958. The solution exists for any number up to 11,000, except 10,958. There's no proof that it doesn't exist either, hence the 10,958 problem.

I gave the students an assignment to write a program that would find the solution, brute-forcing any possible combination of signs and parenthesis. All calculations were done with `double` instead of some sort of `bignum`, so it's not a real solution, but it's simpler this way.

I had several reasons to think that this was a well-fitting problem. First, I could write a solution I expected to see from the students in five hours or two evenings. Compensating for the students' level, it looked like a good project for a semester-long assignment. Second, the solution isn't too straightforward. Simple brute force would take too much time so you need to cut off the equal variants in the beginning. And third, I was fascinated by this problem myself, so I thought the students would feel the same. I couldn't be more wrong.

Not only were a majority of students not into such mathematics, most of them couldn't even grasp what the problem was about and what this 10,958 was for. I failed to get them interested.

When I realized the problem, it was too late to change anything. So the first semester wasn't very engaging. Only a few students could finish the assignment to its fullest. For the rest, it was impossible.



## The second semester



Since it was a full-term course, I had time to make up for it. For the second semester, I tried to come up with something practical and interactive. I gave the students an assignment to write a game. They were learning networking, so it was to be a multiplayer game. I recalled playing “tic-tac-toe on an infinite field” with my peers during boring lectures. The smarter name for this game is Gomoku. Two players can play online, and the game logic is simple. I was hoping to spark students' interest. This assignment turned out much better than the previous one, but I still can't call it a full success.

I really wanted to show them how coroutines (fibers, as they are called in D) make async programming much more manageable, how nice [the vibe.d framework](https://vibed.org/) is, and how easy it is to use external libraries with [the dub package manager](https://code.dlang.org/). Lesson One: don't try to sell coroutines to those who don't know what callback hell is. Lesson Two: always keep hardware limitations in mind.



## Problems out of nowhere



I would never have thought that single-threaded compilation could be thwarted by the memory limit. It happened because the computers at the university were equipped with only 2 GB of memory, and some students' netbooks worked on a mere 1 GB. Simple programs build just fine even on weak machines--D is much better than C++ in terms of compilation time--but a large framework like vibe.d requires a lot of memory for its compilation. Now imagine, I was just telling them how everything is so easy-peasy and then half the students couldn't even compile a networked version of Hello World.

In my defense, I checked everything in advance. I set up a virtual machine with 2 GB of memory and made sure that it worked with the special compiler flags. Theoretically, I was prepared. But dealing with a new library _and_ a new build system _and_ new compiler flags all at once was just too much for the students. Their brains were getting DoS’d and shut down. So even though I demonstrated how to compile a program on a low-memory machine, I still had to explain to them individually why the usual method of compilation wasn't working. For those who had only 1 GB of memory, I didn't even have a ready solution.

But still, the results of the second semester were much better: absolutely everyone could write the client side of the game. Some had problems with coroutines on the server side, but in general, they got this part just fine. For me, that wasn't enough. So I gave them a supertask: write a simple AI for the game. This would help them understand the advantages of the client-server architecture. They had to realize that an AI is just another client, so the server code should be left as it is, and on the client side, the only required modification was move polling. This was a good problem on architecture design, suitable for students who already had gotten a grip on programming.

We also had a little contest. I invited the students to play some code golf, solving a simple problem with the smallest program possible. For encouragement, I promised to free the best-achieving students from writing a semester report. And if anyone could beat my solution, they would pass the semester examination automatically (in Russian universities, teachers are allowed to set arbitrary conditions, like participating in side projects, for passing the semester without the usual examination).

As I expected, nobody could beat me, though a couple of students came up with some interesting tricks, like writing `10` instead of `"\0"` to save two symbols. Some would say this is an abuse of the type system, but clever hacks like this speak to a student's knowledge.



## The results of the first year



I was asking too much for a single semester; only one student could write an AI that kind of worked (she said that the first time it made a sensible move, it made her really happy). Another student who complained the whole year that programming was not her thing and that she couldn't understand anything managed to write her own client and server. At the end of the day, all was not for nothing.



## The second try



Everything described until this point happened during the 2018/2019 academic year. The higher-ups had no issues with the course, and I was able to continue on the following year. New students, new possibilities, a new program. This time I was much better prepared. I had materials for most of the lectures, no more need to redo everything on the fly, and during the summer I had time to fix the problems with the course.

The 10,958 problem was expelled on the charge of being boring. Instead, the Gomoku game was expanded, now lasting _almost_ the whole term. "Almost" because the first assignment was a problem regarding OOP: students had to implement various geometrical shapes as classes and draw them on screen with some customization. I made this the first step after Hello World so everyone could learn how to use the tools, and so I could judge their level.

It's not surprising that students are very different. Some are already working as programmers and attend the course out of curiosity, while some still struggle with variables and loops. From the beginning, I decided that my course would be not just for everyone, but for those who are interested. However, disregarding the others would be wrong, so we needed assignments for different levels.

To jump-start the network study, I set up a server with the reference implementation, and everyone could connect to it and play. Even telnet would do. Allotting more time for an assignment and providing better explanations served students well. Many could finish the supertask. We even had a little contest between AIs. A human could still beat them with ease, but it was still a big improvement over the previous year.



## A pleasant surprise



I had the same code golf contest again. And again, I promised to give a free pass to whoever could beat me. I'm sure you can see where this is going, but first I must explain the problem in detail. The task was to implement a function, `challenge`, defined as follows:


```d
import std;

/**
* Params:
* s = Multiline string, each line containing positive integers separated by whitespace.
* Returns: An array containing the sums of each line and the grand total as the last element.
*/
uint[] challenge(string s);

unittest {
    assert(challenge("0") == [0, 0]);
    assert(challenge("1\n1") == [1, 1, 2]);
    assert(challenge("2\n2\n3") == [2, 2, 3, 7]);
    assert(challenge("2\n2 0 3\n3 1 1 4") == [2, 5, 9, 16]);
}
```
Only the symbols inside the function body count. Using any Phobos functionality is allowed, but no renamed imports.

My solution from the previous year was pretty straightforward:


```d
auto r=s.split('\n').map!(a=>a.split.map!(i=>i.to!uint).sum); return r.array~r.sum; // 84 characters
```
I didn't show them my solution, I just told them its character count. I was absolutely sure that, just like the previous year, nobody could beat it. Imagine my shock when I was outdone in a mere week:


```d
auto r=split(src,"\n").map!"sum(map!(to!uint)(a.split))".array;return r~[sum(r)]; // 82 characters
```
Even with the unneeded brackets, this solution was better, thanks to the shortened `map` syntax: passing a lambda as a string in the first case and passing `to` directly in the second. The latter was just my oversight, but the former was a typical teacher's mistake. You see, back when I first wrote my solution, it wasn't possible to pass a string to `map` like that. I don't remember well what the problem was, something about scopes. I missed that it was fixed at some point. That's how in just one year you become an old geezer teacher who can't keep up with the times.

I stayed true to my word and granted a free pass to the resourceful student at the end of the semester. I was concerned that securing the pass in the middle of the semester would demotivate him to attend classes, but fortunately, I was wrong.



## COVID-19 strikes



Of course, I can't omit how the pandemic impacted the educational system. In the spring of 2020, all classes were moved online. There are some pros and cons to this.

Giving lectures remotely turned out to be very convenient. You can mutually agree on a suitable time, and you don't need to find a free lecture room. Another very good thing is that students can either ask questions vocally--as when they raise their hands offline--or write them in the chat so that the teacher can answer them when it's most appropriate. I really think that this system works very well.

Practical lessons are problematic though, especially with those students who aren't involved enough. When they sit in a classroom they at least do something. Why show up at all if you're not going to do anything? When it's online, they just skip class. The chemistry of a group coding session with a teacher ready to help won't kick in. I encouraged them to ask questions not only during class but at any time, hoping that this would allow me to assist them whenever they are coding. But this only worked for a couple of people.



## Lessons of the second year



The second year was better than the first but still less than ideal. I saw no need to modify the course, but its presentation had to be addressed.

We need to tear down this wall in communication. Mutual trust between the teacher and the student makes for a better educational process. It's difficult to work with those who are wary of the teacher even when doing alright. I don't yet know a robust solution; each case is tackled individually.

Even those who are doing well need some control. I used to think that attendance scrutiny is for those students who don't really want to learn, to intimidate them to show up. When I was a student, the best teachers never bothered about absentees. So I too was liberal with these things, even when half the group was missing. But everything has its limits. Bad students will skip classes anyway, but lazy B-graders could benefit from a little scolding.



## Fast forward to the present



I don't have much to say about the last year-and-a-half. Teaching D is now part of my life. Due to the pandemic, we've had to move online completely. I couldn't even meet my students face-to-face. Aside from what I said earlier about how this affects the educational process, this also disrupted one of the ideas I had.

I need some way to track students' progress to be sure that they actually follow the course and don't just show up for classes. The way I handled this during the previous two years worked for some students: if they had a problem, they asked questions during a lecture or a practical lesson. But some students just keep quiet when they have problems, so I need some means to identify them. I couldn't do written tests for programming; that would be nonsense. So during the summer break, I came up with the idea of doing some small quizzes if COVID restrictions were to be lifted. These quizzes would affect the final grade, but not much, as the intention behind them was to help students who are having problems, not to be the final straw that breaks the camel's back. Unfortunately, remote lessons made this nearly impossible.

My students keep surprising me, coming up with newer and better solutions to the code golf puzzle. Unfortunately, I don't have the exact code of the student who won in the third year (it appears that she deleted her Github account), but it was something like:


```d
auto r=s.split('\n').map!(a=>a.split.to!(uint[]).sum); return r.array~r.sum; // 74 characters 
```
Again, it was a feature I didn't know about: `to` converts between arrays, too.

I thought that this problem was done and that there was no room for further improvement. Oh, how wrong I was. This is what they've brought me this year:


```d
return(s.split('\n')~s).map!(a=>a.split.to!(uint[]).sum).array; // 63 characters
```
This completely destroys my solution with all the improvements! Note that the trick is not in the syntax, but in the logic. This works by concatenating the split lines with the original string, making it unnecessary to declare an intermediate array, which allows implementing this function as a single statement. I would never have thought to do that! Algorithms win over micro-optimizations, even in code golf.

The biggest change this year is a new, formalized evaluation system. In the semester, students earn points for doing assignments and writing reports on them. The first and simplest assignment is worth 5 points, and the last and hardest is worth 30. The maximum number of points a student can score without participating in code golf is 100. Code golf is scored by the formula `110 − length`. The code golf winner this year got 47 points for his solution which earned him an exemption from writing any reports. We have a table listing every student's points so everybody knows how many points they need to score. Everything is very transparent, so I don't need to worry about not being objective when evaluating students.
