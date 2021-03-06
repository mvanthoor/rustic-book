# Frequently asked questions

## Why Rust? What's wrong with C and C++?

Nothing. I like C and C++; especially C. They give me the same sort of thrill as I'd get from handling a live piece of dynamite :P No, really. I love C to bits. If Rust hadn't existed, my chess engine would have been written in C and would (probably) have a different name.

These languages are the 800 pound gorilla's in the room with regard to systems programming, embedded software engineering, and also, chess engines. The reason is that these languages compile down to superfast native machine code, so they can be used to obtain maximum performance. At the same time, these languages have also been around for a long time.  They are well known, and a lot of open source code is floating around the internet, ready to be used. While all of this can be an advantage, there are also some drawbacks.

Both C and C++ are inherently unsafe. Fast, but also dangerous. One has to be very precise in managing memory and resources. If one isn't, it's easy to create memory leaks, subtle and hard to find bugs, race conditions and deadlocks when programming multi-threaded, or you could have the program crash. There are _many_ C and C++ compilers. If a program is written and compiles using one compiler, it won't necessarily compile using another. It can be necessary to make changes.

There are other problems I could list, but most of them come down to the fact that C, as of 2020, was designed 47 years ago. Think about that. In the world of computers, 47 years ago is akin to prehistory. Even though newer versions of C and C++ are adding more modern features, they can never escape their (unsafe) roots completely.

And, one of the main reasons to avoid C or C++ when writing a *chess engine* specifically, isn't technical at all. In my case, it's personal. Everybody and their grandma, dog _and_ cat has a chess engine in C or C++ nowadays. I want mine to be written in a different language.

Preferably, that would be a language as fast as C or C++, but avoiding most or even all of the memory and thread unsafety issues. I've looked at different compiled languages such as C#, Go, Ada, and others. In the end I've chosen Rust.

## As fast as C, but completely safe? I wanna know!

[Rust Language website](https://www.rust-lang.org/)

... to do ...

## What about this site?

Rustic-chess.org is Rustic's home on the internet. This site will document the entire development of Rustic, starting from the very beginning, and its progress after each new feature is added. It will also collect all the information that was found around the internet. As such, it will be possible to write one's own chess engine by following Rustic's development steps.

## It looks weird. What software does it use?

Yeah. This might come as a shock, but it's not Wordpress. This site is written using MD Book. This is a commandline utility written in Rust. It creates websites as books out of Markdown files. It's open source, written by and for the Rust language community, and it's used to create the Rust language documentation. I am using it because it allows me to write documentation which is then converted into a static website, so I don't have to maintain or update a content management system such as Wordpress. Rustic-chess.org is purely informative. It is going to contain a lot of information, so it's priority is to be easy to navigate, very readable, and easy to maintain. The less work needs to be done to maintain it, the more I can either develop Rustic, or write its documentation.

## Hasn't this been done before?

Of course, this has been done before. There are some well known tutorial engines around. Most of them do have some issues, for example:

* The engines and tutorials are getting older.
* The source code may be old enough to not compile cleanly using a current-day compiler.
* These engines mainly use older techniques (for example, not bitboards) for simplicity's sake. Those techniques work well to learn the basics of chess engine programming, but because of speed limitations, the engine will always be at a disadvantage to an engine using bitboards if everything else is equal.
* Even though the engines may be available with (some) documentation in the source code, in some cases the accompanying website has been gone for a long time.
* The code isn't always as readable as could be.
* Most if not all engines are written in C or C++.

Even if this has been done before, I still get fun out of doing this myself, in exactly the way I want to be done and in a programming language of my choice. Even though I'll be standing on the shoulders of giants that have shared information before me, this project is my own. It makes it worthwhile to me. I hope, in the future, other people starting out to write a chess engine will find it useful.