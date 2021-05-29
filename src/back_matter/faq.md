# Frequently asked questions

## Why Rust? What's wrong with C and C++?

Nothing... and a lot, both at the same time. I actually like C and C++;
especially C, because it is such a small language, very easy to learn, and
it basically runs on every computer platform ever created. C gives me the
same sort of thrill as I get from handling a live piece of dynamite. No,
really. I love C to bits. (I should stop making word jokes, I know.) If
Rust hadn't existed, my chess engine may have been written in C and in that
case would probably have had a different name.

These languages are the 800 pound gorilla's in the room with regard to
systems programming, embedded software engineering, and also, chess
programming. The reason is that these languages compile down to native
machine code which runs at lightning speed, so they can be used to obtain
maximum performance. Also, these languages have also been around for a long
time. They are well known, and a lot of open source code is floating around
the internet, ready to be used. While all of this can be an advantage,
there are also some drawbacks.

Both C and C++ are inherently unsafe, because the programmer is allowed to
do anything he or she wants. These languages are fast, but also dangerous.
Even though C is easy to learn, it has a lot of pitfalls one should be
careful about. Managing memory and resources correctly is hard, and has to
be exactly right. If they aren't, it's easy to create memory leaks, subtle
and really hard to find bugs, race conditions and deadlocks when
programming multi-threaded. You could also crash the program.

There are other problems I could list, but most of them come down to the
fact that C, as of 2020, was designed 47 years ago. Think about that. In
the world of computers, 47 years ago is akin to prehistory. Even though
newer versions of C and C++ are adding more modern features, they can never
escape their (unsafe) roots completely.

One of the main reasons to avoid C or C++ when writing a *chess engine*
specifically, isn't technical at all. In my case, it's personal.
*Everybody* has a chess engine written in C or C++; using a different
language is more the exception than the rule. I want mine to be written in
a different language.

Preferably, that would be a language as fast as C or C++, but avoiding most
or even all the pitfalls of those languages. I've looked at different
options such as C#, Go, Ada, and others. In the end I chose Rust. Even
though Rust itself is not a perfect language and it can't solve _all_ the
problems (it's still possible to create subtle bugs or crash the program),
it goes a long way in making memory management and threading a lot easier.

## As fast as C, but safe? I wanna know!

[See the Rust website](https://www.rust-lang.org/)

In short, the Rust programming language can be summed up as such:

- Statically typed with type inference. If a variable has a certain type,
  the compiler will enforce that type. If you use this type in combination
  with functions and other types, the compiler will often know by inference
  what the type of other variables should be, and enforces that too.
- Safe. The compiler statically checks the safety of the code, so you
  can concentrate on writing the software, instead of having to concentrate
  on safe coding. This component is called the "borrow-checker."
- Because of point 1, the following happens:
    - No uninitialized variables... ever.
    - No memory leaks.
    - No race conditions.
    - Multi-threading actually becomes fairly easy!
- No dependency problems: if you know and like any package manager
   (apt-get, yum, npm, pip), you'll love cargo. (However, if you hate all
   package managers, you'll hate this one too.)
- Truly cross platform: one code-base compiles and runs everywhere. You
   only need conditional compiles if you use specific features of a
   specific language.
- Easy building of the software. If you have Rust installed, the only
   thing you have to do is "cargo build --release", and cargo will download
   all the dependencies, compile them, and create the executable. If your
   Rust version is not too old, it will work.
- Completely modularized: no header files.
- Zero-cost abstractions.
- Great tooling (works with C/C++ debuggers and profilers).
- Last but not least: Speed.
- And many other features that make it a nice language to use.


## What about this site?

Rustic-chess.org is Rustic's home on the internet. This site will document
the entire development of Rustic, starting from the very beginning, and its
progress after each new feature is added. As such, it will be possible to
write one's own chess engine by following Rustic's development steps.

## It looks weird. What software does it use?

Yeah. This might come as a shock, but it's not Wordpress. This site is
written using MD Book. This is a commandline utility written in Rust. It
creates websites as books out of Markdown files. It's open source, written
by and for the Rust language community, and it's used to create the Rust
language documentation. I am using it because it allows me to write
documentation which is then converted into a static website, so I don't
have to maintain or update a content management system. Rustic-chess.org is
purely informative. It is going to contain a lot of information, so it's
priority is to be easy to navigate, very readable, and easy to maintain.
The less work needs to be done to maintain it, the more I can either
develop Rustic, or write its documentation.

## Hasn't this been done before?

Of course, this has been done before. There are some well known tutorial
engines around. Most of them do have some issues, for example:

* The engines and tutorials are getting older. The source code may be old
  enough to not compile cleanly using a current-day compiler.
* These engines mainly use older techniques, often for simplicity's sake.
  Those techniques work well, but because of speed limitations, the engine
  will always be at a disadvantage to an engine using more modern
  techniques if everything else is equal.
* Even though the engines may be available with (some) documentation in the
  source code, in some cases the accompanying website has been gone for a
  long time.
* The code isn't always as readable as it could have been.
* Most if not all engines are written in C or C++, and many of these
  engines use particular features (or even loopholes!) in those languages
  to get certain things done. That is not good programming practice.

Even if this has been done before, I still get fun out of doing it myself,
in exactly the way I want to be done and in a programming language of my
choice. Even though I'll be standing on the shoulders of giants that have
shared information before me, this project is my own. Having my own chess
engine makes it worthwhile to me. I hope, in the future, other people
starting out to write a chess engine will find the information contained in
this site useful.
