
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Piece-Square Tables](#piece-square-tables)
  - [Explanation](#explanation)
  - [Caveats](#caveats)
  - [Implementation](#implementation)
    - [Encoding](#encoding)
    - [Functions](#functions)

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

### Encoding

In essence, implementing PSQT's is similar to implementing the material
count from the previous chapter. The only difference is that a PSQT has 64
values instead of 6. Rustic Alpha 3 does not yet have tapered PSQT's;
therefore it only has one value per piece/square combination for the entire
game, instead of two values for the opening and endgame. For simplicity's
sake the implementation of PSQT's with one value will be described. If you
understand this, switching to a tapered PSQT will be trivial. Implementing
the PSQT's is not hard, but it requires a bit of thought, especially in the
case of the so-called flip-table. 

First, we create the types for the PSQT's and list all of them:

```rust,ignore
type Psqt = [i8; NrOf::SQUARES];
type PsqtSet = [Psqt; NrOf::PIECE_TYPES];

const KING: Psqt = [ /*values here */ ];
const QUEEN: Psqt = [ /* values here */ ];
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
const BISHOP: Psqt = [ /* values here */ ];
const KNIGHT: Psqt = [ /* values here */ ];
const PAWN: Psqt = [ /* values here */ ];

pub const PSQT_SET: PsqtSet = [KING, QUEEN, ROOK, BISHOP, KNIGHT, PAWN];
```

There are also two special tables we need. These are the KING_EDGE and FLIP
tables, which have a special purpose within the evaluation function.

The KING_EDGE table:
```rust,ignore
pub const KING_EDGE: Psqt = [
    -95,  -95,  -90,  -90,  -90,  -90,  -95,  -95,  
    -95,  -50,  -50,  -50,  -50,  -50,  -50,  -95,  
    -90,  -50,  -20,  -20,  -20,  -20,  -50,  -90,  
    -90,  -50,  -20,    0,    0,  -20,  -50,  -90,  
    -90,  -50,  -20,    0,    0,  -20,  -50,  -90,  
    -90,  -50,  -20,  -20,  -20,  -20,  -50,  -90,  
    -95,  -50,  -50,  -50,  -50,  -50,  -50,  -95,  
    -95,  -95,  -90,  -90,  -90,  -90,  -95,  -95,
];
```

As you can see all the values except the four in the center are negative.
This table is used in the endgame. As you can understand, the king is not a
playable piece for most of the opening and middle game. It has to be kept
safe, or it runs the risk of being checkmated by the opponent's
long-distance pieces when lines are opened. The main King PSQT makes sure
the king stays in the corner and off the middle files by giving bonus
points for castling.

However, in the endgame, the role of the king changes. Depending on the
situation he can be a defensive or an offensive piece, and because there
are much fewer pieces on the board the risk of suddenly being checkmated is
much less. In fact, in the endgame, being checkmated is much more of a risk
if the king stays near the edges, because its mobility is much lower. Also,
after enough pieces are traded, the king MUST come out of the corner to
help with checkmating the opponent. That is where the KING_EDGE table is
for. It is used to add a massive penalty to the king's position if it is
near the edges of the board in the endgame.

>__Sidenote:__ For the eagle-eyed among you: this is already a hint towards
>tapering the PSQT's between opening and endgame values. This KING_EDGE
>table basically is a poor man's version of tapering the King's positional
>values. This table will obviously be dropped when a tapered evaluation is
>implemented.

The other special table mentioned above is the FLIP table:
```rust,ignore
pub const FLIP: [usize; 64] = [
    56, 57, 58, 59, 60, 61, 62, 63,
    48, 49, 50, 51, 52, 53, 54, 55,
    40, 41, 42, 43, 44, 45, 46, 47,
    32, 33, 34, 35, 36, 37, 38, 39,
    24, 25, 26, 27, 28, 29, 30, 31,
    16, 17, 18, 19, 20, 21, 22, 23,
     8,  9, 10, 11, 12, 13, 14, 15,
     0,  1,  2,  3,  4,  5,  6,  7,
];
```

This table is used to flip the point of view of the PSQT's. To make the
PSQT's easier to relate to and easier to edit, they have been laid out as a
normal chess board with A1 at the lower left corner, as such:

```rust, ignore
let psqt_white = [
    A8, B8, C8, D8, E8, F8, G8, H8,
    A7, B7, C7, D8, E8, F8, G7, H7,
    A6, B6, C6, D6, E6, F6, G6, H6,
    A5, B5, C5, D5, E5, F5, G5, H5,
    A4, B4, C4, D4, E4, F4, G4, H4,
    A3, B3, C3, D3, E3, F3, G3, H3,
    A2, B2, C2, D2, E2, F2, G2, H2,
    A1, B1, C1, D1, E1, F1, G1, H1,
];
```

Black sees this same table from the other side. Imagine as if you are
flipping the chess board over, by lifting it at the A1-H1 row and then
tipping it backwards. The A1-H1 row is now on top, and the A8-H8 row is on
the bottom. It gives this table for black's viewpoint:

```rust, ignore
let psqt_black = [
    A1, B1, C1, D1, E1, G1, F1, H1,
    A2, B2, C2, D2, E2, G2, F2, H2,
    A3, B3, C3, D3, E3, G3, F3, H3,
    A4, B4, C4, D4, E4, G4, F4, H4,
    A5, B5, C5, D5, E5, G5, F5, H5,
    A6, B6, C6, D6, E6, F6, G6, H6,
    A7, B7, C7, D7, E7, G7, F7, H7,
    A8, B8, C8, D8, E8, F8, G8, H8,
]
```

Now we have a problem, because we want to store each PSQT only once for
both players, not twice. 

When a chessboard is stored within an array, the convention is for the
first element (the one at position [0] in the array) to be A1. The second
element would be B1, the 3rd C1, and so on, which leads to this setup:

```rust,ignore
let array = [
    A1, B1, C1, D1, E1, G1, F1, H1,
    A2, B2, C2, D2, E2, G2, F2, H2,
    A3, B3, C3, D3, E3, G3, F3, H3,
    A4, B4, C4, D4, E4, G4, F4, H4,
    A5, B5, C5, D5, E5, G5, F5, H5,
    A6, B6, C6, D6, E6, F6, G6, H6,
    A7, B7, C7, D7, E7, G7, F7, H7,
    A8, B8, C8, D8, E8, F8, G8, H8,
]
```

Now let's compare the white viewpoint array with the actual layout. In the
layout, we have replaced the square names with the array indexes. The
storage array is on the left, the white viewpoint is on the right:

```rust, ignore
| let array = [                     | let psqt_white = [                  
|    0,  1,  2,  3,  4,  5,  6,  7, |    A8, B8, C8, D8, E8, F8, G8, H8, 
|    8,  9, 10, 11, 12, 13, 14, 15, |    A7, B7, C7, D8, E8, F8, G7, H7, 
|   16, 17, 18, 19, 20, 21, 22, 23, |    A6, B6, C6, D6, E6, F6, G6, H6, 
|   24, 25, 26, 27, 28, 29, 30, 31, |    A5, B5, C5, D5, E5, F5, G5, H5, 
|   32, 33, 34, 35, 36, 37, 38, 39, |    A4, B4, C4, D4, E4, F4, G4, H4, 
|   40, 41, 42, 43, 44, 45, 46, 47, |    A3, B3, C3, D3, E3, F3, G3, H3, 
|   48, 49, 50, 51, 52, 53, 54, 55, |    A2, B2, C2, D2, E2, F2, G2, H2, 
|   56, 57, 58, 59, 60, 61, 62, 63, |    A1, B1, C1, D1, E1, F1, G1, H1, 
| ];                                | ];
```

We said above that, when storing the array, the A1 square would be at
element 0, B1 would be at element 1, and so on. If you super-impose the
PSQT-array (left) on the storage array (right), it can be seen that the
following is true:

Square 0 (A1)  => PSQT element 56.
Square 7 (H1)  => PSQT element 63.
Square 56 (A8) => PSQT element 0
Square 63 (H8) => PSQT element 7

So when playing white, and we want to get the PSQT value for A1, which is
stored as Square 0, we actually need to look in the PSQT at element 56. If
you keep up this mapping, the FLIP table mentioned earlier comes out:

```rust,ignore
pub const FLIP: [usize; 64] = [
    56, 57, 58, 59, 60, 61, 62, 63,
    48, 49, 50, 51, 52, 53, 54, 55,
    40, 41, 42, 43, 44, 45, 46, 47,
    32, 33, 34, 35, 36, 37, 38, 39,
    24, 25, 26, 27, 28, 29, 30, 31,
    16, 17, 18, 19, 20, 21, 22, 23,
     8,  9, 10, 11, 12, 13, 14, 15,
     0,  1,  2,  3,  4,  5,  6,  7,
];
```

This is how it is used:

```rust,ignore
let square = 0 // (A1)
let psqt_element = FLIP[square];
white_psqt_value = Psqt[piece_type][psqt_element];

// Or the shorter version:
let piece_type = 1; // queen
let square = 0;     // gets a value from a function, for example.
white_pst_value = PSQT_SET[piece_type][FLIP[square]]
```

So what about the black pieces? Let's compare the storage array with the
PSQT from black's point of view:

```rust, ignore
| let array = [                     | let psqt_black = [
|    0,  1,  2,  3,  4,  5,  6,  7, |     A1, B1, C1, D1, E1, G1, F1, H1,
|    8,  9, 10, 11, 12, 13, 14, 15, |     A2, B2, C2, D2, E2, G2, F2, H2,
|   16, 17, 18, 19, 20, 21, 22, 23, |     A3, B3, C3, D3, E3, G3, F3, H3,
|   24, 25, 26, 27, 28, 29, 30, 31, |     A4, B4, C4, D4, E4, G4, F4, H4,
|   32, 33, 34, 35, 36, 37, 38, 39, |     A5, B5, C5, D5, E5, G5, F5, H5,
|   40, 41, 42, 43, 44, 45, 46, 47, |     A6, B6, C6, D6, E6, F6, G6, H6,
|   48, 49, 50, 51, 52, 53, 54, 55, |     A7, B7, C7, D7, E7, G7, F7, H7,
|   56, 57, 58, 59, 60, 61, 62, 63, |     A8, B8, C8, D8, E8, F8, G8, H8,
| ];                                | ];
```
As you can see, the storage array and PSQT from black's point of view are
the same. Square 0 = A1, Square 7 is H1 and so on, and Square 63 is H8.
Therefore when we're getting a PSQT value for black, we don't have to flip
anything and we can index directly into the array. Thus the code looks like
this:

```rust, ignore
// Or the shorter version:
let piece_type = 1; // queen
let square = 0;     // gets a value from a function, for example.
black_pst_value = PSQT_SET[piece_type][square];
```

### Functions

To make use of the PSQT values, we need to read them for the current
position. We have the same choice as we did with the material count: we can
read the entire PSQT set over and over again for each move, or we can
update it incrementally. Rustic has always done this incrementally because
it saves a lot of time. To do this, we 'apply' the PSQT to the position
when the engine initializes the board. After that, we just add and subtract values
according to the pieces that move or are captured.

This is the init() function of the board:

```rust, ignore
fn init(&mut self) {
    // other init code omitted for brevity
    // ...

    let psqt = psqt::apply(self);
    self.game_state.psqt[Sides::WHITE] = psqt.0;
    self.game_state.psqt[Sides::BLACK] = psqt.1;
}
```

That's it. We execute the _apply()_ function in the PSQT module. It adds
the PSQT values for white, then adds the ones for black, and returns both
in a nameless tuple. We then store these values in the game state struct.
This is the _apply()_ function, which basically is a more extensive version
of the one that counts material:

```rust, ignore
pub fn apply(board: &Board) -> (i16, i16) {
    let mut w_psqt: i16 = 0;
    let mut b_psqt: i16 = 0;
    let bb_white = board.bb_pieces[Sides::WHITE]; // Array of white piece bitboards
    let bb_black = board.bb_pieces[Sides::BLACK]; // Array of black piece bitboards

    // Iterate through the white and black bitboards (at the same time.)
    for (piece_type, (w, b)) in bb_white.iter().zip(bb_black.iter()).enumerate() {
        let mut white_pieces = *w; // White pieces of type "piece_type"
        let mut black_pieces = *b; // Black pieces of type "piece_type"

        // Iterate over pieces of the current piece_type for white.
        while white_pieces > 0 {
            let square = bits::next(&mut white_pieces);
            w_psqt += PSQT_MG[piece_type][FLIP[square]] as i16;
        }

        // Iterate over pieces of the current piece_type for black.
        while black_pieces > 0 {
            let square = bits::next(&mut black_pieces);
            b_psqt += PSQT_MG[piece_type][square] as i16;
        }
    }

    (w_psqt, b_psqt)
}
```

Analogous to the material count incremental update, the PSQT's are updated
using the board's functions that call piece movement:

```rust,ignore
    pub fn remove_piece(&mut self, side: Side, piece: Piece, square: Square) {
        // Other piece movement code omitted
        //...
        let flip = side == Sides::WHITE;
        let s = if flip { FLIP[square] } else { square };
        self.game_state.psqt[side] -= PSQT_SET[piece][s] as i16;
    }

    pub fn put_piece(&mut self, side: Side, piece: Piece, square: Square) {
        // Other piece movement code omitted
        //...
        let flip = side == Sides::WHITE;
        let s = if flip { FLIP[square] } else { square };
        self.game_state.psqt[side] += PSQT_SET[piece][s] as i16;
    }
```

So now that you understand the purpose and workings of the PSQT's, we can
move on to the next step: implementing the evaluation function. After what
you've done up to this point, the first version of the evaluation will turn
out to be rather easy.
