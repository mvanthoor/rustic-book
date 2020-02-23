![Rustic Logo](./img/rustic-logo-web.jpg)

# About Rustic

Rustic is a chess engine, written in the Rust programming language. This is an original work, not derived from any others that came before it, although inspiration and knowledge was collected from many websites and some open source engines.

The main features this engine strives to achieve are:

* Written completely in the Rust programming language.
* Using only safe code: so no _unsafe {}_ blocks, if at all possible.
* Simple, easy to read code (simplicity over cleverness or brevity).
* Complete documentation regarding its development and progression.
* Open source (GPLv3), to give something back to the community.

Uhm... wait. No list of features with regard to playing chess? Of course there will be, but at this time, the engine is still in development, and the code is still private. The very first version released will have the following features:

* Board representation
* Fancy magic bitboards
* Move generator
* Make/Unmake move
* Alpha-Beta search
* Simple Evaluation function
* Simple time management
* UCI protocol support

This will be the bare minimum needed to make the engine run and be able to play games against other engines. At this point, a base ELO can be determined, and the engine will be released as Rustic 0.1, as open source under the GPLv3 license. As development continues, new features will be added. As soon as I deem the engine strong enough, that version will become Rustic 1.0.
