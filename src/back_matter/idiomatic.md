# On idiomatic Rust

Before we start, I'd like to talk a bit about writing idiomatic code. Every
programming language has its own way of doing things. Some programming
patterns occur over and over again and are considered the 'correct way.'
This is called "idiomatic", and most programmers try to follow the idioms
of the language they are writing in.

So it is with Rustic. When I started this engine, I didn't know a lot about
Rust, but I was well versed in C. Therefore if you look at early versions
of Rustic, you may see many things of which more experienced Rust
programmers would say: "This is not idiomatic Rust; this looks like C."
They would be correct.

Over time, Rustic became more idiomatic: more and more C-like constructs
where removed and replaced by Rust features. Consider for example the
printing of a struct's values to the screen in a pretty way, or converting
them into a string, so they can be appended to the end of another string.
To achieve this, I implemented ".print()" and ".to_string()" for some
structs. Sometimes there were even completely seperate functions for
printing, carried over from the very first versions of the engine.

The idiomatic way to do this, is to implement the trait Display. This has
the advantage that the struct can be printed, but it also automatically
gets the ".to_string()" function for free.

For Rustic 4,a big push has been made to make the engine as idiomatic as
possible without compromising readability and maintainability. I can hear
you saying: "Make it as idiomatic as possible? Why not make everything
idiomatic, and why would that harm readability and maintainability?" Allow
me to try and explain with an example.

Rust is statically typed and has great type saftey. This means that you
cannot assign a value of type X to a variable that is of type Y, without
casting from X to Y first. Rustic tries to take advantage of this as much
as possible. However, in a chess program, both pieces and squares are often
(but not always) represented as 8-bit integers. This means that these can
accidentally be swapped around when calling a function. (Even if you use a
type alias.)

It is possible to create new types for squares and pieces by wrapping them
into a struct or an enum. However, it will later turn out that bitwise
operations need to be done on these values, constantly, in many different
places. Because we wrapped the u8 into a struct or an enum, we would need
to implement all the bitwise operators for these new types, so they can be
used as if they were integeres. What we are basically doing is
re-implementing the u8 type from scratch, but wrapped into a struct or
enum, just to create a different type.

This makes the code very verbose, hard to understand and harder to
maintain, because it is an extra struct/enum layer on top of the u8 type we
need. Therefore I did not do any of these wrappings to keep all the u8
types seperated and chose to just keep my eyes open and make sure I don't
swap (for example) squares and pieces.

One place where you may encounter an "old-fashioned" way of doing things is
when Rustic iterates over the move list. It uses a for-loop, instead of an
iterator. The reason is that the this list is an array backed with a
counter, so it can be stored on the stack. (In case of a Vec, it would be
stored on the heap, which is slower.) When I implemeted Iterator for
MoveList the engine's speed dropped by 10% because of the Some/None check
that comes with an iterator. Therefore I just use a for-loop.

I'm always open for suggestions that can improve the engine to adhere to
idiomatic Rust, but please take into account that it is possible that it
doesn't follow idiomatic style by choice, because it has some sort of
disadvantage in the specific case of a chess engine: either speed,
maintainability or readability.
