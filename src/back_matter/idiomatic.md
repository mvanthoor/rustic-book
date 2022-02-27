# On idiomatic Rust

I'd like to say a few words about writing idiomatic code. Every programming
language has its own way of doing things. Some programming patterns occur
over and over again and are considered the 'correct way.' This is called
"idiomatic code", and most programmers try to follow the idioms of the
language they are writing in.

So it is with Rustic. When I started this engine, I didn't know a lot about
Rust, but I was well versed in C. Therefore if you look at early versions
of Rustic, you can see many things of which more experienced Rust
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
gets you the ".to_string()" function for free.

For Rustic 4, a big push has been made to make the engine as idiomatic as
possible without compromising its speed. I can hear you saying: "Make it as
idiomatic as possible? Why not make everything idiomatic, and why would
that harm the engine's speed?" Allow me to try and explain with an example.

Rust is statically typed and has great type saftey. This means that you
cannot assign a value of type X to a variable that is of type Y, without
casting from X to Y first. Rustic tries to take advantage of this as much
as possible. However, in a chess program, both pieces and squares are often
(but not always) represented by integers. This means that these can be
swapped around when calling a function. They can even be swapped if you use
a type alias, because a type alias is just that: another name for the same
thing. The compiler does not make any difference between two aliases such
as Square or Piece which are the same thing underneath.

The proper way to create new types in Rust is the "NewType" pattern, where
types are created by wrapping simple types into a struct or enum. It is
possible to do this with squares and pieces. However, it will later turn
out that bitwise operations need to be done on these values, constantly, in
many different places. Because we wrapped the integer into a struct or an
enum, we would need to implement all the bitwise operators for these new
types, so they can be used as if they were integeres. What we are basically
doing is re-implementing one of the integer types from scratch.

This makes the code very verbose, hard to understand and harder to
maintain, because it is an extra struct/enum layer on top of the integer
type we need. Therefore I did not do any of these wrappings to keep all the
u8 types seperated and chose to just keep my eyes open and make sure I
don't swap (for example) squares and pieces.

One place where you will encounter an "old-fashioned" way of doing things
is when Rustic iterates over the move list. It uses a for-loop, instead of
an iterator. The reason is that the this list is an array backed with a
counter, so it can be stored on the stack. (In case of a Vec, it would be
stored on the heap, which is slower.) When I implemeted Iterator for
MoveList the engine's speed dropped by 10% because of the Some/None check
that comes with an iterator. Therefore I just use a for-loop.

I'm always open for suggestions to improve the engine in such a way that it
adheres more to idiomatic Rust, as long as it doesn't slow down the
execution speed, doesn't hurt readability, or maintainability. Please take
into account that, because of these requirements, Rustic sometimes doesn't
follow idiomatic Rust by choice.
