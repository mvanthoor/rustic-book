
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Piece-Square Tables](#piece-square-tables)
  - [Explanation](#explanation)
  - [Caveats](#caveats)
  - [Implementation](#implementation)

<!-- /code_chunk_output -->

# Piece-Square Tables

## Explanation

Piece-Square Tables (henceforth PST, often also called PSQT) are the most
fundamental part of an engine's evaluation function. Without PST's it's
very hard to get the engine to play a decent game of chess. They have been
present since the very first version of the engine. After counting
material, this is the first evaluation function to implement.

PST's are exactly what the name implies: they are tables that indicate
which piece goes where. A PST's value indicates if a square is a good
square for a piece, or it isn't. Because you only have an empty board and
the piece itself available to evaluate if a square is good or not, you can
only look at the piece's mobility. On top of that, you can encode a tiny
bit of positional knowledge into the PST's.

Let's take a look at how it's done. This is a PST for a rook, from white's
point of view (a1 is on the lower left, to make the table easy to read and
edit):

```csharp
const ROOK_MG: Psqt = [
    0,   0,   0,   0,   0,   0,   0,   0,
   15,  15,  15,  20,  20,  15,  15,  15,
    0,   0,   0,   0,   0,   0,   0,   0,
    0,   0,   0,   0,   0,   0,   0,   0,
    0,   0,   0,   0,   0,   0,   0,   0,
    0,   0,   0,   0,   0,   0,   0,   0,
    0,   0,   0,   0,   0,   0,   0,   0,
    0,   0,   0,  10,  10,  10,   0,   0
];
```

The rook has 14 squares available to move to, from any square on the board.
Given only that criterion, it doesn't matter where the rook is. It always
has 14 squares available on an empty board. Therefore, most of the values
in the table are 0.

Now we also see a tiny bit of the mentioned positional knowledge: we _know_
that the rook is strong on the 7th rank, so all squares on that rank get a
small bonus. We also know the rook is _often_ strong on the two middle
files, so they get a bonus as well. _F1_ gets a bonus as an extra
encouragement for the king to castle, as the rook is better on _f1_ than it
is on _h1_.

A somewhat busier PST is the one from the knight:

```csharp
const KNIGHT_MG: Psqt = [
    -20, -10,  -10,  -10,  -10,  -10,  -10,  -20,
    -10,  -5,   -5,   -5,   -5,   -5,   -5,  -10,
    -10,  -5,   15,   15,   15,   15,   -5,  -10,
    -10,  -5,   15,   15,   15,   15,   -5,  -10,
    -10,  -5,   15,   15,   15,   15,   -5,  -10,
    -10,  -5,   10,   15,   15,   15,   -5,  -10,
    -10,  -5,   -5,   -5,   -5,   -5,   -5,  -10,
    -20,   0,  -10,  -10,  -10,  -10,    0,  -20
];
```

There are many squares that give negative values for the knight. The closer
to the edge and corners it is, the greater the negative impact. In the
middle 16 squares, the knight gets a bonus. The reason is that the knight,
as opposed to the rook, loses mobility as it is closer to the edge and
corners. So, it is better in the middle of the board.

We do not encode any other positional knowledge into the knight's PST; we
can't, because what is _really_ the best location for a knight, depends on
the placement and interaction of all the other pieces.

We repeat this for all other pieces, so we end up with 6 PST's: one for the
king, queen, rook, bishop, knight, and pawn.

## Caveats

You may be wondering: but it's not a given that THIS piece always needs to
be on THAT square. It changes during the game. In the endgame, the pieces
should go on different squares than they were on in the opening and the
middle game.

That's true; it's the first caveat. The PST's are only a very rudimentary
guideline for the engine. Imagine a beginner, who has just learned the
rules. He doesn't have an inkling of where the pieces should be placed. As
a general rule, you can tell him: King castled, rooks on the two middle
files, knights in the middle, bishops on long distance, targeting the
center or the position of the enemy king. The beginner player then has some
notion of what he has to do to get the game going. The PST's do the same
thing for the chess engine.

In a later chapter we will discuss the so called _tapered evaluation_,
which gives the engine _two_ sets of PST's, one set for the
opening/middle-game, and one for the endgame. Then the engine can gradually
"glide" from the opening/middle-game PST into the endgame PST as the game
progresses. (The values are "tapered", or interpolated, between the
opening/middle-game and endgame values.)

After that, we will also write an automatic tuner, which populates the
PST's with values that give good results; often better than what you will
be able to do by hand. (Not to mention that tweaking 12 PST's by hand is
boring and it has to be re-done after you change _anything_ in either the
search or the evaluation!)

Tapering and tuning the PST's is a major strength boost for most engines.

(You can see the _MG postfix for "middle-game" in the PST names already, to
take into account that the engine is going to have a tapered and tuned
evaluation at some point.)

The second caveat is that PST's, by themselves, can't take the dynamics of
the game into account. A white knight might generally be great on _e5_, but
there are many reasons to come up with why it would be better on _d6_, or
_c4_, or any other square, depending on the placement of the other pieces.

For this, we need the _evaluation terms_ that take the dynamics of the game
into account, and which modify the evaluation score after the PST's have
had their say. The more evaluation terms we have, the less important the
PST's become, but together with the material count, they always remain the
basis; the starting point to get going.

## Implementation

Implementing the PST's is not hard, but it requires a bit of thought.

