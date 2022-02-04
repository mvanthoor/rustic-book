# Roadmap

A chess engine is quite a complex piece of software. It is not possible to
"just" write a chess engine off the bat, except if you've got lots of
experience in writing chess engines or similar software. (In that case,
you're probably reading the wrong book.) Therefore this chapter will
describe a roadmap to get your new engine to a playing state.

First, the engine needs to be designed. If necessary, reread the chapters
"concept" and "design", where Rustic's overall structure is explained. In
the roadmap below, the big parts of this architecture are going to be
unpacked into smaller parts which can be implemented one at a time.

The roadmap will describe how to get the engine to play somewhat decent
chess. From that point onward, it "just" becomes a matter of adding
features to make the engine stronger. If your overall design and
implementation is well done, adding newer features on top of the baseline
version is much less difficult than getting to the baseline itself.

Don't worry if you encounter terms in the roadmap you don't yet understand.
When we get to the design and implementation, these will be explained.

- Design and write the board representation
    - Represent the board
    - Represent the pieces
    - Read a position from an FEN-string
    - Keep the current state of the game
    - Keep a history of played moves
    - Implement Zobrist hashing
    - Create functions to control the board
    - Create functions to get information from the board
- Design and write the move generator
    - "Teach" the move generator how the pieces move
    - Create a move format
    - Generate moves for all the pieces when the MG is given a position
    - Take special moves into account (such as castling)
    - Adds generated moves to a move list and returns this when done
    - Can generate captures and silent moves separately
    - Can determine if a square is attacked
- Add perft (performance testing)
    - Add a perft function that runs on a given position.
    - Run through a perft suite containing "tricky" positions.

** **First milestone reached: move generator is bug-free** **

- Write the search functionality
    - Structs (information) needed by the search
    - Write the iterative deepening function
    - Write the Alpha-Beta function
    - Write the Quiescence search
    - Make fixed search possible (on ply, nodes, or time)
    - Implement time management for non-fixed searches
    - Helper functions such as determining if a position is draw
- Write the evaluation function
    - Material counting
    - Piece Square tables (PST or PSQT)
- Order the moves for more speed
    - MVV-LVA (Most Valuable Victim-Least Valuable Attacker)
- Design a communication interface ("IComm" in Rustic)
    - Write the UCI-protocol
    - Write the XBoard protocol (optional)
    - Make sure the engine understands the commands
    - Make sure the engine reacts correctly

** **Second milestone reached: baseline is done. 1500-1700 Elo** **

- Add more advanced functionality
    - Transposition table
    - TT move ordering
    - Principal Variation Search
    - Killer moves
    - Tapered Evaluation
    - Texel tuning

** **Third milestone reached: the engine should be >= 2000 Elo** **

- Keep adding more features to become stronger
    - History heuristics
    - Pruning capability
        - Null move pruning
        - Futility pruning
        - Mate pruning
        - ... etc ...
    - Evaluation
        - Mobility
        - King safety
        - Passed pawns
        - General knowledge
        - ... etc ...
    - Many other features

_Do not begin to extend the engine_ with new features until the baseline as
outlined above has proven to be bug-free. It should be able to complete a
perft suite without errors and it should be able to play thousands and
thousands of test games in a row without crashing, making illegal moves or
forfeiting on time.

Go back and read the above paragraph again. This is absolutely essential.
If your baseline engine is not bug-free, any and all features you add next
will either not bring you the improvement they should, or they won't work
at all. This also means you should only move on to the next feature if the
previous feature has proven to be working correctly.

Fixing bugs _afterwards_, in a feature you added weeks or even months ago
is difficult in a chess engine; much more so than it is in "normal"
software. The reason is that many features are synergetic; this means that
the behavior or effectiveness of one feature can be altered by another. If
you add a new feature and it doesn't work right, it could be because of a
bug in an older piece of code... but you won't know until you check _all_
the code.

After you hit the second milestone, you should have an engine that is on
par with Rustic Alpha 1 with regard to functionality. The engine has no
chess knowledge apart from PST's and material values. You can use Rustic
Alpha 1 versions to get started with engine to engine testing.

If your engine has no bugs, and you're using Rustic's PST's and material
values, you can expect an rating of 1500-1700 Elo after the engine is
tested in a list such as the CCRL 2m+1s Blitz list. Rustic Alpha 1 has a
rating of [1675
Elo](https://ccrl.chessdom.com/ccrl/404/cgi/engine_details.cgi?match_length=30&each_game=1&print=Details&each_game=1&eng=Rustic%20Alpha%201%2064-bit#Rustic_Alpha_1_64-bit).

In case your baseline engine reaches less than 1500 Elo, it's either very
slow, buggy, or both. First make sure there are no bugs, then try to
optimize for speed. This is the reason why you should choose a fast
programming language; if you don't, you could start out with a 200 Elo
disadvantage against engines with a similar feature set.

Should you be in the high 1600's, you _could_ possibly hit 1700 by
tinkering a lot with the material values, piece square tables, and time
management. It's probably not worth it, as all those functions will either
be improved or replaced in later versions, so don't spend too much time on
this.

(I believe 1700 on the CCRL Blitz list is the about the maximum
you could reach with the given baseline version that has only material
values, PST's, MVV-LVA move sorting, and no special search or move
generator optimizations.)

From this point onward, you can add features in any order you want, but I
suggest to follow the roadmap up to the third milestone. There are features
that depend on other features. To implement transposition table move
ordering, you obviously first need to have a working transposition table.

Let's move on to the first topic: board representation.