# Playing Strength

As mentioned in the preface of this book, writing a chess engine can be a
rewarding endavour. It takes a long time to write  strong engine,
especially if you are just starting out. It takes lots of knowledge, time,
and often, perseverance, to get the first basic version up and running.
After this version works and plays legal and somewhat decent chess, it can
be improved incrementally. Each new feature adds strength to the engine, at
least, if it's done correctly.

This is no different with Rustic. The first basic version is called Rustic
Alpha 1. This is the base-line version. It only has the minimal amount of
features to play legal, but decent chess:

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
playing strength they provide. Please note that this is different from
other engines: adding the features in a different order, will give
different results. Some features work so well together, that it's almost
illogical to add one without the other. In that case, the playing strength
is determined as if they were a a single feature.

