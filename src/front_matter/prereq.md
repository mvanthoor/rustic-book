# Prerequisites

There's one more thing we need to discuss before you can start on this
journey. Those are the... dreaded... _prerequisites_. Yeah, I don't like
them either, but they're really important to get right as to avoid
disappointments in the future.

_First_. Let's set some expectations first. This is not a weekend project
(for most people) if you want to do it well. It certainly isn't a weekend
project if you have a full time job. Writing a strong chess engine takes
time; not only for writing it, but also researching, debugging, and
testing. Be prepared that you're going to be at this for some time before
you see the first results i.e., have the thing play a non-trivial game
against another engine, and win it.

_Second_. This is not a project for someone who is just starting out with
computers, computer science, or programming. You certainly don't need a
Ph.D. in computer science, but you _will_ need to have intermediate
knowledge of computers and about programming. If there's one thing I would
recommend  to brush up on, then it would be bitwise operations. If you are
not already well versed in that, go study it first: thoroughly. If you have
a strong background in low(er) level programming languages such as C, C++,
Rust, or even classic Pascal, this book and the subject of chess
programming will be easier for you. If you are coming from the
web-development world, used to Javascript, Python or PHP, it will
(probably) be harder, but certainly not impossible. (You can actually write
chess engines in Javascript and Python, though.) If you just learned to
write "Hello World", this probably will not be the project for you...
sorry. However, you're invited to come back later.

_Third_ If you are not experienced in several different programming
languages of different styles, and/or low level programming, then pick a
language you know well, so it will spare you the burden of learning chess
programming _AND_ the programming language at the same time. Only pick a
programming language you don't know because you want to use this project to
learn it, like I did with Rust. Be prepared for a steep learning curve and
some rewrites of parts of your code.

_Four_ Make sure your development environment is set up correctly. This is
a topic that is not discussed in this book, apart from the chapter where
instructions are given on how to compile Rustic for different operating
systems. It's impossible to give advice here, because there are so many
IDE's, compilers, debuggers, operating systems, and programming languages
to choose from, that I would never get around to writing the actual book.
Make sure you are able to compile/run "Hello World" in your programming
language. Make sure your editor/IDE has a debugger configured correctly,
because you can set breakpoints and halt the program at that point, and
debug it.

_Five_ This is not strictly necessary per se, but I recommend using the a
version control system such as Git, even if you are working alone.
Versioning your code makes the development process _a lot easier_. I mean
it. Don't skimp on this. Many IDE's and editors have provisions for Git and
other version control systems built in, or they provide plugins or
extensions. If you don't like these, then there are many GUI-applications
to be found to use Git. If you are really a contrarian, you can also use
choose a different verison control system... as long as you use
_something_.

Still here? Great. Now we start. Good luck.