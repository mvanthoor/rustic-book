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
where removed (edit in 2024: and are being removed) and replaced by Rust
features. Consider for example the printing of a struct's values to the
screen in a pretty way, or converting such a struct into a string, so it
can be appended to the end of another string. To achieve this, I
implemented ".print()" and ".to_string()" for some structs. Sometimes there
were even completely separate functions for printing, carried over from the
very first versions of the engine.

The idiomatic way to do this is to implement the trait "Display". This has
the advantage that the struct can be printed, but it also automatically
gets you the ".to_string()" function for free.

For Rustic 4, a big push is being made to make the engine as idiomatic as
possible without compromising its speed, readability or maintainability. I
can hear you saying: "Make it as idiomatic as possible? Why not make
everything idiomatic, and why would that harm the engine's speed,
readability and maintainability?" Allow me to try and explain with an
example.

For example, one place where you will encounter an "old-fashioned" way of doing
things is when Rustic iterates over the move list. It uses a for-loop,
instead of an iterator. The reason is that the this list is an array backed
with a counter, so it can be stored on the stack. (In case of a Vec, it
would be stored on the heap, which is slower.) When I implemented Iterator
for MoveList the engine's speed dropped by 10% because of the Some/None
check that comes with an iterator. Therefore I just use a for-loop, so the
engine doesn't lose 10% speed. Should I find a way to implement an Iterator
in such a way that it is just as fast as the for loop, then I'll obviously
do so because it is the better and safer way.

I'm always open for suggestions to improve the engine in such a way that it
adheres more to idiomatic Rust, as long as it doesn't slow down the
execution speed and doesn't hurt readability, or maintainability. Please
take into account that, because of these requirements, Rustic sometimes
doesn't follow idiomatic Rust _by choice_.
