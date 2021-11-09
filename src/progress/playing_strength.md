
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Playing Strength](#playing-strength)
  - [Progress per feature and version](#progress-per-feature-and-version)
  - [Determining progression in actual playing strength](#determining-progression-in-actual-playing-strength)

<!-- /code_chunk_output -->
# Playing Strength

As mentioned in the preface of this book, writing a chess engine can be a
rewarding endeavour. It takes a long time to write a very strong engine,
especially if you are just starting out. It takes some knowledge about
programming and chess, lots of time, and often, perseverance, to get the
first basic version up and running. After this version works and plays
legal and somewhat decent chess, it can be improved incrementally. If done
correctly, each new feature will add playing strength.

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
provides an overview of the added features per version and the gain in
playing strength they provide. The strength gain is measured by playing the
new Rustic version against the previous version. Please note that the
results obtained in my tests will be different compared to your own,
because each engine is different. Also, adding features in a different
order will give different results.

Also take into account that results by self-play are inflated. Because one
engine has a feature the other doesn't have, with that feature being the
only difference, the newer engine will (ab)use this feature constantly. In
the end the real increase in playing strength can only be measured in large
tournaments. Self-play is used to prove that the newer engine is stronger
than the previous version, not to obtain a rating. As a rule of thumb, at
least for Rustic, it seems that 60% of the rating improvement obtained in
self-play will 'stick' when running in a tournament.

> **Side note:** A notable difference seems to be the tapered and tuned
> evaluation, and the refactors and optimizations done thereafter. They
> provide an improvement of 300 Elo in self-play, and the entire gain seems
> to carry over into gauntlets and tournaments, instead of the expected
> 60%. Hopefully the improvement will also carry over when playing more
> different engines for the CCRL rating list.

In the table below, we start by writing the baseline version, which is then
known as version Alpha 1. CCRL tested this version with a result of 1675
ELo in their Blitz list.

The feature "TT cuts only" was built on top of Alpha 1. The rating increase
in self-play against Alpha 1 was +42 Elo. Then the "TT Move Ordering"
feature was built on top of the "TT Cuts Only" version, and this gained
another +100 Elo in self-play. This completes the transposition table. This
version became Alpha 2, which was tested in the CCRL list at 1815 Elo. Then
"Killer moves" were built on top of Alpha 2... and so on.

## Progress per feature and version

| Version       | Feature                   | Improvement | CCRL   |
|---------------|---------------------------|-------------|--------|
|               | Writing baseline...       |             |        |
| Alpha 1       | Baseline                  |             | 1675   |
|               | TT cuts only              | 42          |        |
|               | TT Move sorting           | 103         |        |
| Alpha 2       |                           |             | 1815   |
|               | Killer Moves              | 56          |        |
|               | PVS                       | 55          |        |
| Alpha 3.0.0   |                           |             | 1865   |
| &nbsp;        | &nbsp;                    | &nbsp;      | &nbsp; |
| Alpha 3.1.112 |                           |             |        |
|               | Tapered & tuned eval      | 248         |        |
|               | Refactor / Optimize / Fix | 52          |        |
| Later...      | &nbsp;                    | &nbsp;      |        |
|               | Aspiration Window         | ?           |        |
|               | History Heuristic         | ?           |        |
|               | Null move pruning         | ?           |        |
|               | ... ?                     | ?           |        |


## Determining progression in actual playing strength

It is impossible to define the strength of a chess engine, or a human
player for that matter, by an exact number. This is because of how the
Elo-rating system works. The rating system works with a pool of players,
and it determines their relative strength, from one player to another. Not
every player can play every opening or time control equally well. It also
happens that a certain player A consistently performs better against B than
expected, but also consistently plays worse than expected against player C.

Chess engine testing is  also not transitive, especially if the tested
engines are fairly close in strength. This means that if engine A almost
always wins matches against engine B, while engine B consistently wins
matches against engine C, it is *NOT* the case that engine A will for
certain consistently win against engine C.

If you test engines in your own gauntlets, it is therefore impossible and
unadvisable to compare the results of your gauntlet against something like
the CCRL rating list. The time control is different (and even if it's
exactly the same, your computer is different), the opening books are
different, and you are likely choosing different opponents. Your pool of
engines to test is also smaller. All these factors will cause your gauntlet
to have different ratings for each engine, including your own.

The only thing you can use your gauntlet for is to determine if "new
engine" is stronger than "old engine", if both "new" and "old" play the
same opponents, under the same time controls, with the same opening book,
on the same computer, and with the same settings. The one conjecture you
can make is that if "new" is 100 Elo stronger than "old" in your own
gauntlets, then "new" will probably (but _not_ certainly) be roughly 100
Elo stronger than "old" in a CCRL test. But they might accidentally choose
an opponent against which your engine fares exceptionally well or very
badly, while you did not have that opponent in your pool. This will lead to
a higher or lower rating than expected.
