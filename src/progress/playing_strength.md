# Playing Strength

As mentioned in the preface of this book, writing a chess engine can be a
rewarding endavour. It takes a long time to write a very strong engine,
especially if you are just starting out. It takes lots of knowledge, time,
and often, perseverance, to get the first basic version up and running.
After this version works and plays legal and somewhat decent chess, it can
be improved incrementally. If done correctly, each new feature adds playing
strength to the engine.

This is no different with Rustic. The first version, Alpha 1, is the
baseline version. It only has the minimal amount of features to play legal,
but decent chess:

- The board representation (Bitboards)
- Move generator (Fancy Magic Bitboards)
- Make/Unmake move on the board
- Alpha-Beta search
- Quiescence search
- Check extension
- MVV-LVA move sorting
- Evaluation (Material counting and PST's)
- UCI communication protocol

All other versions build on top of the previous version. The table below
provides an overview of the added features per version, and the gain in
playing strength they provide. Please note that this will be different from
other engines: adding the features in a different order, will give
different results. Some features work so well together, that it's almost
illogical to add one without the other. In that case, the playing strength
is determined as if they were a single feature.

Reading the table works like this: Rustic Alpha 1 is the baseline version.
On top of that, the transposition table is added, and tested (+105 Elo),
and TT Move ordering is added after that. The result is Rustic Alpha 2,
which contains both of the features on top of the baseline version.

## Progress per feature and version

| Version        | Feature                     | Improvement | Strength |
|----------------|-----------------------------|-------------|----------|
| Rustic Alpha 1 | Baseline version            |             | 1675     |
|                | Transposition Table         | 105         |          |
|                | TT Move sorting             | 65          |          |
| Rustic Alpha 2 |                             |             | 1845     |
|                | Killer + History Heuristics | ?           |          |
|                | Aspiration Window + PVS     | ?           |          |

## Determining progression and playing strength

It is impossible to define the strength of a chess engine, or a human
player for that matter, by an exact measure. This is because of how the
Elo-rating system works. The rating system works with a pool of players,
and it determines their relative strength, from one player to another. Not
every player can play every opening or time control equally well. It also
happens that a certain player A consistently performs better against B than
expected, but also consistently plays worse than expected against player C.

The list above is created by running my own tournaments, where different
Rustic versions play against a set of other engines. This results in a
rating list where Rustic Alpha 1 is set to strength 0. To make this list
easier to relate to, it is calibrated against the CCRL Blitz list, in which
Rustic Alpha 1 performs around 1675-1680 Elo.

Because the list above and the CCRL Blitz list (or any other rating lists)
use different opponents, time controls and opening books, the results in
different lists cannot be compared directly. A Rustic version in one list
can have a different rating from the same version in another list.



