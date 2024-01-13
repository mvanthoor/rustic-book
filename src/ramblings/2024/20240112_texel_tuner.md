# January 12, 2024 - Writing a Texel Tuner

Hi there. Welcome to 2024. Another year, and you're possilby wondering if the Rustic project
is still alive. Not very suprising, because I said in March 2023 that I was
picking this up again (including building a new computer), but no new
versions have been released in the meantime.

Rest assured: the project isn't dead, and if you have been looking into the
GitHub repository from time to time, you may have seen that there are
commits now and again. First, I'm still maintaining versions Alpha 1, 2 and
3 to stay compatible with the latest Rust and library versions, and I've
been extensively refactoring Rustic Alpha 3 to be more Rust-like, and less
C-like with regard to coding style.

To be honest, I've been stuck at the point that should be the primary new
feature of Rustic 4 (yes, without the Alpha, because I consider that to be
the first 'complete' version): Being able to tune its own evaluation
parameters using Texel Tuning. I've been under the impression that this was
an absurdly difficult process to get done because every paper and tutorial
throws heaps of maths at you. Now, I'm not bad at maths, and I don't hate
it. However, I'm a software engineer that knows the maths he should know,
can learn most subjects without too many issues, but I don't have a math
background extensive enough to understand 3th and 4th year university level
maths.

Thus, I got stuck on sentences such as: "Create a vector of mutable
coefficients." A vector? Like in... something that points in a certain
direction? Or a vector (list) like a Rust or C++ vector? Mutable means that
I must be able to change it, but what is a coefficient? Where do I get that
from? An activation sigmoid function? How do I create that; can I just use
one from another engine or isn't that possible? So I got massively stuck on
that.

However, in my 2023 Christmas holiday, I took the time to re-read/re-watch
everything I collected about (Texel) Tuning over the years, and there where
two sentences in that material that made it all click for me:

- "Create a vector of mutable coefficients" => Make a list all the
  parameters you want to tune.
- Sigmoid Activation Function => Use a function to map values within a
  certain range. And yes, there are well-known Sigmoid functions to do
  this, and the original Texel Tuning uses a slightly altered version of
  one of them.

Having finally understood this, I started the work to write a Texel Tuner.
The first steps have already been done:

- Write a file reader that reads EPD (extended position diagrams), in which
  FEN-strings are labeled with the outcome of the game.
- Convert each EPD-line into a datapoint, containing the FEn-string and the
  result of the game.
- Refactor the evaluation function so it doesn't directly use constants,
  but can get its evaluation parameters passed into it. (We need this
  because the tuner will constantly change parameters and then run the
  evaluation with that new set of parameters).

Next steps are going to be:
- Write a function to convert my evaluation parameters into a single list (The "Vector of
  mutable coefficients").
- Write a function to convert this list back to the struct my evaluation
  function expects (I'm NOT going to change my evaluation to directly index
  into a single massive array to get at the parameters).
- Write a function that dumps the struct to the hard disk after each Texel
  run, so I can just copy/paste the values into the chess engine and
  re-compile it for testing.
- And, of course, write the function that does the actual tuning.

So, there's still quite some work to be done, but fortunately, we're
underway. Just like Magic Bitboards, the actual implementation seems much
more doable and understandable than I thought at first. After the tuner is
finished, I'll obviously describe the entire process in this book before I
forget everything again. (Fortunately, I keep extensive comments in the
code, just as I did with the Magic Bitboards.)

Have a nice 2024, and I hope to see you around when Rustic 4 releases.
