+++
title = "Docs"
date = 2024-01-06
+++

I have a few projects. Here I talk about how they came about and what I learned
as I did them. Not a particularly technical segment, so refer to the sidebar
for specific project documentation.

## My software engineering

I would not really consider myself a software engineer. It is true that me
being able to create a software project that solves a non-trivial problem, such
as a game of blackjack, and being able to compile the project, puts me above
many people in terms of software engineering skill. People who overwhelmingly
do not know what a compiler is, what a programming language is, or even how the
binary numeric system works. At the same time, my relationship to software is
largely instrumentative. I am interested in software so far as it is able to
solve issues for me, not because I care about software engineering in itself.

At the same time, software is indistinguishable from magic, and its usefulness
cannot be overstated. I cannot help but be somewhat passionate when I see it
being abused for the most idiotic of purposes, such as Electron or the GNOME
desktop environment. Just because I have 32 GB of RAM on my computer does not
mean your software is now allowed to use all of it. I still expect the software
on my computer to adhere to the UNIX philosophy. Keep it simple, and you shall
solve many problems. Electron does absolutely nothing except bloat your bad
program that is just as bad, if not worse, than programs before it.

In summary, I think you could say that I am a software engineer so far as I am
a computer user. I have work to do, I have a problem, and I need the right
tools to solve the problem. Software that does not adhere to the UNIX
philosophy is generally just bad software. This is why I care about software.

## nimjack and nimchain

These were my first ever software projects that I would consider complete
projects. This was by no means my first encounter with programming, but I never
did anything that could be considered in any way a minimum viable product.
Granted, nimchain does not have the essential component of networking, but from
a design perspective it would not be difficult to plug into what I created,
especially with Nim.

*nimjack* and *nimchain* were inspired by Nim. What I mean by that is that Nim
opened new ways for me to think about programming. Before, many, many years
ago, I would start with PHP, then move onto other "easy" programming languages
like Python. However, I never really got the languages. Nothing ever stuck to
me. I could never look at a problem and know exactly how to solve it. It would
always be a matter of trial and error. This is because I never fundamentally
understood key concepts of programming that are required to truly understand
and appreciate programming.

Nim is amazing, because it has all the niceties of modern programming
languages, with a syntax that doesn't make you feel like you are reading with a
needle stuck in your eye. At the same time, it is a statically typed language,
where you have to reason about what you are doing in terms of type. Once I
understood types, everything started becoming clear for me. I now could
actually start programming, rather than fumble my way through and hope that one
of the things I try succeeds. Consequently, I wrote [some thoughts about
programming languages](https://maxwelljensen.no/programming-langs/), if you are
interested in seeing my perspective.

## bibel

This one was my first big project in Rust. It is not really a particularly good
program. It was in fact first written in Go, but then I went onto Rust, and
went all-out on trying some difficult programming problems to solve. The
biggest one of those was implementing parallelism in the program, although that
was accomplished using the Rayon library. It makes it very easy to do loops and
many other routines as multiple threads, but you still need to reason about
when and how that can be useful. It was an excellent learning experience.

Programming in Rust also taught me even more about what goes on in a program at
a low level. I think Rust is the real C++, even if people give it flack for
having a massive toolchain and compiling times. I always found it to be a weird
criticism, considering that you would use neither Rust nor C++ to make small
programs. You use either of those to make huge, complicated programs, but only
one of those is really suitable without making it an incredibly costly and
risky endeavour. I know at least I learned a lot from the experience either
way.
