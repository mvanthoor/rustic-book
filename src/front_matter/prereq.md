
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Prerequisites](#prerequisites)
  - [This is gonna take a while](#this-is-gonna-take-a-while)
  - [Not for total beginners](#not-for-total-beginners)
  - [Mind your language](#mind-your-language)
  - [Be happy with your environment](#be-happy-with-your-environment)
  - [Git those versions under control](#git-those-versions-under-control)
  - [Actually know something about chess](#actually-know-something-about-chess)

<!-- /code_chunk_output -->


# Prerequisites

There's one more thing we need to discuss before you can start on this
journey. Those are the... dreaded... _prerequisites_. Yeah, I don't like
them either, but they're really important to get right as to avoid
disappointments in the future.

## This is gonna take a while

_First:_ Let's set some expectations first. This is not a weekend project
(for most people) if you want to do it well. It certainly isn't a weekend
project. Writing a strong chess engine takes time; not only for writing it,
but also researching, debugging, and testing. Be prepared you're going to
be at this for some time before you see the first results i.e., have the
thing play a non-trivial game against another engine, and win it.

## Not for total beginners

_Second:_ This is not a project for someone who is just starting out with
computers, computer science, or programming. You certainly don't need a
Ph.D. in computer science to understand the information in this book, but
you _will_ need to have strong intermediate knowledge of computers and
about programming. If there's one thing I would recommend to brush up on,
then it would be bitwise operations. If you are not already well versed in
those, go study this topic first: thoroughly. If you have a strong
background in low(er) level programming languages such as C, C++, Rust, or
even classic Pascal, this book and the subject of chess programming will be
easier for you. If you are coming from the web-development world, used to
Javascript, Python or PHP, it will (probably) be harder, but certainly not
impossible. (You can actually write chess engines in Javascript and Python,
but those languages wouldn't be my personal choice.) If you just learned to
write "Hello World", this probably will not be the project for you...
sorry. However, you're invited to come back later.

## Mind your language

_Third:_ If you are not experienced in several different programming
languages of different styles, and/or low level programming, then pick a
language you know well. This will spare you the burden of learning chess
programming _and_ a new programming language at the same time. Only pick a
language you don't know because you want to use this project to learn it,
like I did with Rust. Even so, you should have intermediate to
early-advanced programming skills in at least one other language, so you
don't need to learn all the basics from scratch. Be prepared for a steep
learning curve and some rewrites of parts of your code.

## Be happy with your environment

_Four:_ Make sure your development environment is set up correctly. This is
a topic which is not discussed in this book, apart from the chapter where
instructions are given on how to build Rustic for different operating
systems. It's impossible to give advice here, because there are so many
IDE's, compilers, debuggers, operating systems, and programming languages
to choose from. Make sure you are able to compile/run "Hello World" in your
programming language. Make sure your editor/IDE has a debugger configured
correctly, because you can set breakpoints and halt the program at that
point, so you can watch variables and debug the program. If you haven't
already got a working development environment, search the internet to find
out how to set one up. My personal preference is to use Visual Studio Code
as an IDE, Rust as a programming language (obviously), and the VSCode
plugin _rust_analyzer_ (which gives you code completion and linting for
Rust in VSCode) as the basis.

## Git those versions under control

_Five:_ This is not strictly necessary per se, but I recommend using the a
version control system such as Git, even if you are working alone.
Versioning your code makes the development process _a lot easier_. I mean
it. Don't skimp on this. Many IDE's and editors have provisions for Git and
other version control systems built in, or they provide plugins or
extensions. If you don't like these, then there are many GUI-applications
to be found to use Git. If you are really a contrarian, you can also use
choose a different verison control system... as long as you use
_something_.

## Actually know something about chess

_Six:_ Also not strictly necessary, but it would be very helpful if you
know _all_ the chess rules already. As with other programming projects, it
is good to know the basics of the domain you're working in. Think about the
movement of the pieces, castling, en-passant, and the different draw rules.
You could just find all the rules and implement them, but you wouldn't be
able to understand what the engine is doing, and finding bugs is going to
be harder. It would even be better if you could play some decent chess
already. Even beginner level between 1000-1200 Elo should be sufficient. At
that point you're probably not going to make any mistakes against the rules
anymore, so there's less chance you'll implement the rules the wrong way.

Still here? Great. Now we start. Good luck.