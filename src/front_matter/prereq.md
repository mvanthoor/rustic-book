
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Prerequisites](#prerequisites)
  - [This is gonna take a while](#this-is-gonna-take-a-while)
  - [Not for complete beginners](#not-for-complete-beginners)
  - [Mind your language](#mind-your-language)
  - [Be happy with your environment](#be-happy-with-your-environment)
  - [Git those versions under control](#git-those-versions-under-control)
  - [Know something about chess](#know-something-about-chess)

<!-- /code_chunk_output -->


# Prerequisites

There's a few more things we need to discuss before you can start on this
journey. Those are the... dreaded... _prerequisites_. Yeah, I don't like
them either, but they're really important to get right so you can avoid
disappointments during this project.

## This is gonna take a while

_First:_ Let's set some expectations first. This is not a weekend project
(for most people) if you want to do it well. Writing a strong chess engine
takes time; not only for writing it, but also researching, debugging, and
testing. Be prepared: you're going to be at this for some time before you
see the first results. That would be having your engine play a non-trivial
game against another engine, and _win it_.

## Not for complete beginners

_Second:_ This is not a project for someone who is just starting out with
computers, computer science, or programming. You certainly don't need a
Ph.D. to understand the information in this book, but you _will_ need to
have strong intermediate knowledge about programming concepts (variables,
constants, functions, data structures) to be able to get this project off
the ground.

If there are two topics I would recommend to brush up on, then it would be
the binary system and bitwise operations. There are two chapters in the
appendix that describe them:

[The binary system](../appendix/binary_system.md)<br />
[Bitwise operations](../appendix/bitwise_operations.md)

If you are not already well versed in these topics then I highly urge you
to study these first, thoroughly. Practice with this until you get it down
without having to think too much. Feel free to find more information about
this on the internet or YouTube, if the explanations in the above two
chapters are not enough.

If you have a strong background in low(er) level programming languages such
as C, C++, Rust, or even classic Pascal, this book and the subject of chess
programming will be easier for you. If you are coming from the
web-development world, used to Javascript, Python or PHP, it will
(probably) be harder, but certainly not impossible. (You can write chess
engines in Javascript and Python, but those languages wouldn't be my
personal choice.) If you just learned to write "Hello World", this probably
will not be the project for you... yet. Of course, you're invited to come
back later.

## Mind your language

_Third:_ If you are not experienced in several different programming
languages of different styles and/or low level programming, then it would
be best to pick a language you know well. This will spare you the burden of
learning chess programming _and_ a new programming language at the same
time. Only pick a language you don't know because you want to use this
project to learn it, like I did with Rust. Even so, you should have
intermediate to early-advanced programming skills in at least one other
language, so you don't need to learn all the basics from scratch. Be
prepared for a steep learning curve and some rewrites, in case you pick a
language you don't know, or don't know well.

## Be happy with your environment

_Four:_ Make sure your development environment is set up correctly. This is
a topic which is not discussed in this book, apart from the chapter where
instructions are given on how to build Rustic for different operating
systems. It's impossible to give advice here, because there are so many
IDE's (Integrated Development Environment), compilers, debuggers, operating
systems, and programming languages to choose from.

Make sure you are able to compile/run "Hello World" in your programming
language of choice. Make sure your editor/IDE has a debugger configured
correctly, so you can set breakpoints and halt the program at that point.
This lets you inspect variables and debug the program. If you haven't got a
working development environment already, search the internet to find out
how to set one up. My personal preference is to use Visual Studio Code as
an IDE, Rust as a programming language (obviously), and the VSCode plugin
_rust_analyzer_, which gives you code completion and linting for Rust in
VSCode.

## Git those versions under control

_Five:_ This is not strictly necessary per se, but I strongly recommend
using the a version control system [such as Git](https://git-scm.com/),
even if you are working alone. Versioning your code makes the development
process a lot easier, not to mention safer. Safer? Yes. You can just create
a branch to write some experimental code. If it doesn't work, you just
throw it away, instead of trying to undo everything you did.

I mean it. Don't skimp on this. Many IDE's and editors have provisions for
Git and other version control systems built in, or they provide plugins or
extensions. If you don't like built-in version control or your favorite
eidtor doesn't support it, then there are many GUI-applications to be
found; open-source, freemium and paid. If you are really a contrarian, you
can also use choose a different version control system. It doesn't matter,
as long as you use _something_.

## Know something about chess

_Six:_ Also not strictly necessary, but it would be very helpful if you
know _all_ the chess rules already. (If not, you can download and read the
[FIDE Laws of Chess
Handbook](https://www.fide.com/FIDE/handbook/LawsOfChess.pdf).) As with
other programming projects, it is good to know the basics of the domain
you're working in. Think about the movement of the pieces, castling,
en-passant, and the different draw rules. You could just find all the rules
and implement them, but you wouldn't be able to understand what the engine
is doing, and finding bugs is going to be harder. It would even be better
if you could play some decent chess already. Even beginner level between
1000-1200 Elo should be sufficient. At that point you're probably not going
to make any mistakes against the rules anymore, so there's less chance
you'll implement the rules the wrong way.

Still here? Great. Now we start. Good luck.