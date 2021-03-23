# About Rustic

The chess engine discussed in this book is called Rustic, which is written
in the Rust programming language. The engine is an original work, but it
implements many well-known concepts, both from computer science in general,
and chess programming in particular.

Rustic is an open-source engine licensed under GPL version 3, so anyone can
download the source code and change it to their liking, and compile it for
their particular platform or CPU. Creating derivative works is possible,
within the rules stated by GPL v3.

At the time of writing, March 23rd of 2021, the engine's version is Rustic
Alpha 2, with the following features:

* Board representation
* Fancy magic bitboards
* Move generator
* Make/Unmake move
* Alpha-Beta search
* Transposition Table
* MVV-LVA move sorting
* Transposition table move sorting
* Material count + PST evaluation
* Simple time management
* UCI protocol support

Obviously the feature list will grow longer as the engine gets developed
further. The book will chronicle those developments and descibe how each
feature is implemented. Example code will be in Rust, coming straight from
a working version of the engine, but there will be enough explanation so
that each feature can be rewritten in any desired programming language.

The programming language obviously needs the right capabilities, so please
note that something like
[WhiteSpace](https://en.wikipedia.org/wiki/Whitespace_(programming_language))
is probably out; and even if the language can do it, it's your own
responsibility. Any and all suffering will be of your own making. Choose
your language wisely.