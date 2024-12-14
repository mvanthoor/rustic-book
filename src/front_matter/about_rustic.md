# About Rustic

The chess engine discussed in this book is called Rustic. It is written in the
Rust programming language. The engine is an original work, but it
implements many well-known concepts, both from computer science in general,
and chess programming in particular.

Rustic is an open-source engine licensed under GPL v3, so anyone can
download the source code and change it to their liking, and compile it for
their particular platform or CPU. Creating derivative works is possible,
within the rules stated by GPL v3.

The feature-set for Rustic Alpha 3.0.0 is:

- Engine:
  - Bitboard board representation
  - Fancy Magic bitboard move generator
  - Transposition Table
  - UCI-protocol
- Search
  - Alpha/Beta search
  - Quiescence search
  - Check extension
  - PVS
- Move ordering
  - TT Move priority
  - MVV-LVA
  - Killer moves
- Evaluation
  - Material counting
  - Piece-Square Tables

Obviously the feature list will grow longer as the engine gets developed
further. The book will describe how each feature works and how it is
implemented. Example code will be in Rust, coming straight from a working
version of the engine, but there will be enough explanation so that each
feature can be rewritten in any desired programming language.

The programming language obviously needs the right capabilities, so please
note that something like
[WhiteSpace](https://en.wikipedia.org/wiki/Whitespace_(programming_language))
is probably out. Even _if_ the language you choose has enough capabilities
to write a non-trivial project in it, the language still may not be a good
fit. The programming language choice is your own responsibility. Any and
all suffering will be of your own making. Choose your language wisely.
