
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Piece-Square Tables](#piece-square-tables)
  - [Explanation](#explanation)
  - [Caveats](#caveats)
  - [Implementation](#implementation)

<!-- /code_chunk_output -->

# Piece-Square Tables

>__Sidenote:__ In this chapter, PSQT's that already contain the material
>values will be used. Take note that you _either_ implement material
>counting + PSQT's starting from 0, _or_ PSQT's that include the material
>count. Don't implement material counting _and_ include the material in the
>PSQT's as well, because you will bee keeping and calculating redundant
>information. This chapter uses PSQT's that already contain the base
>material value.

## Explanation

Piece-Square Tables (henceforth PSQT, also called PST) are the most
fundamental part of an engine's evaluation function. Without PSQT's it's
very hard to get the engine to play a decent game of chess. They have been
present since the very first version Rustic, and most other engines also
include them in some way or another. After counting material, this is the
first evaluation function to implement. (And then, if desired, combine the
material count with the PSQT's to save on calculations.)

PSQT's are exactly what the name implies: they are tables that indicate
which piece goes where. A PSQT's value indicates if a square is a good
square for a piece, or it isn't. Because you only have an empty board and
the piece itself available to evaluate if a square is good or not, you can
only look at the piece's mobility. On top of that, you can encode a tiny
bit of positional knowledge into the PSQT's.

Let's take a look at how it's done. This is a PSQT for a rook, from white's
point of view. A1 is on the lower left, to make the table easy to read and
edit:

```rust,ignore
const ROOK: Psqt = [
    500,   500,   500,   500,   500,   500,   500,   500,
    520,   520,   520,   520,   520,   520,   520,   520,
    500,   500,   500,   500,   500,   500,   500,   500,
    500,   500,   500,   500,   500,   500,   500,   500,
    500,   500,   500,   500,   500,   500,   500,   500,
    500,   500,   500,   500,   500,   500,   500,   500,
    500,   500,   500,   500,   500,   500,   500,   500,
    500,   500,   500,   510,   510,   505,   500,   500
];
```

The rook has 14 squares available to move to, from any square on the board.
Given only that criterion, it doesn't matter where the rook is. It always
has 14 squares available on an empty board. Therefore, most of the values
in the table are 500, which is the base value of the rook.

Now we also see a tiny bit of the mentioned positional knowledge: we _know_
that the rook is strong on the 7th rank, so all squares on that rank get a
small bonus. We also know the rook is _often_ strong on the two middle
files, so they get a bonus as well. _F1_ gets a bonus as an extra
encouragement for the king to castle, as the rook is better on _f1_ than it
is on _h1_.

A somewhat busier PSQT is the one from the knight:

```rust,ignore
const KNIGHT: Psqt = [
    290, 300, 300, 300, 300, 300, 300, 290,
    300, 305, 305, 305, 305, 305, 305, 300,
    300, 305, 325, 325, 325, 325, 305, 300,
    300, 305, 325, 325, 325, 325, 305, 300,
    300, 305, 325, 325, 325, 325, 305, 300,
    300, 305, 320, 325, 325, 325, 305, 300,
    300, 305, 305, 305, 305, 305, 305, 300,
    290, 310, 300, 300, 300, 300, 310, 290
];
```

Some squares have the base value of a knight, some squares are higher, but
also there are many squares with a lower value. The closer to the edge and
corners it is, the greater the negative impact. In the middle 16 squares,
the knight gets a large bonus. The reason is that the knight, as opposed to
the rook, loses mobility as it is closer to the edge and corners. So, it is
better in the middle of the board.

We do not encode any other positional knowledge into the knight's PSQT; we
can't, because what is _really_ the best location for a knight, depends on
the placement and interaction of all the other pieces.

We repeat this for all other pieces, so we end up with 6 PSQT's: one for the
king, queen, rook, bishop, knight, and pawn.

## Caveats

You may be wondering: but it's not a given that THIS piece always needs to
be on THAT square. It changes during the game. In the endgame, the pieces
should go on different squares than they were on in the opening and the
middle game.

That's true; this is the first caveat. The PSQT's are only a very
rudimentary guideline for the engine. Imagine a beginner who has just
learned the rules. He doesn't have any idea of where the pieces should be
placed. As a general rule, you can tell him: King castled, rooks on the two
middle files, knights in the middle, bishops on long distance, targeting
the center or the position of the enemy king. The beginner player then has
some notion of what he has to do to get the game going. The PSQT's do the
same thing for the chess engine.

In a later chapter we will discuss the so called _tapered evaluation_,
which gives the engine _two_ values for each square in the PSQT, one for
the opening/middle-game, and one for the endgame. Then the engine can
gradually "glide" from the opening/middle-game value into the endgame value
as the game progresses. The values are are interpolated, between the
opening/middle-game and endgame values. This is called a "tapered" PSQT,
because the value tapers off from the opening value into the endgame value.

After that, we will also write an automatic tuner, which populates the
PSQT's with values that give good results; often better than what you will
be able to do by hand. Not to mention that tweaking 6 PSQT's holding 128
values by hand is boring and it has to be re-done after you change
_anything_ in the evaluation! Add a term... retune. Better automate that.

Tapering and tuning the PSQT's is a huge strength boost for most engines.

The second caveat is that PSQT's, by themselves, can't take the dynamics of
the game into account. A white knight might generally be great on _e5_, but
there are many reasons why it would be better on _g5_, or
_b4_, even though the PSQT says otherwise.

For this, we need the _evaluation terms_ that take the dynamics of the game
into account, and which modify the evaluation score after the PSQT's have
had their say. The more evaluation terms we have and the more accurate they
are in encoding the position's dynamics, the less important the PSQT's
become. However, they always remain the basis; the starting point to get
going.

## Implementation

Implementing the PSQT's is not hard, but it requires a bit of thought.

