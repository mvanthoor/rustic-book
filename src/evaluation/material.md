
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Material counting](#material-counting)
  - [Explanation](#explanation)
  - [Implementation](#implementation)
    - [Value per piece type](#value-per-piece-type)
    - [Counting vs. Updating](#counting-vs-updating)
      - [Counting from scratch](#counting-from-scratch)
      - [Incrementally updating](#incrementally-updating)

<!-- /code_chunk_output -->

# Material counting

## Explanation

Counting the material on the board is the most basic way of evaluating a
chess position. You can just compare material: "I have one bishop and 2
knights, while my opponent has two bishops and one knight. So light pieces
are equal. We both have a queen, and one rook, so that is equal as well. He
has four pawns and I have five... so in the end, I'm one pawn up." That is
basically it, with counting material.

Somewhat more complicated is the relationships between pieces, if material
is not equally balanced. How much pawns is one knight worth? How much
bishops + pawns is a rook worth? What do I need to get back for giving up
my queen (if it's not the queen of the opponent)? Humans often use this:

- Queen: 9 points
- Rook: 5 points
- Bishop: 3 points
- Knights: 3 points
- Pawn: 1 points

So a knight is worth 3 pawns. A Rook is worth one bishop and two pawns. Two
rooks are worth slightly more than a queen. Three light pieces are exactly
worth a queen too. A queen is also equivalent to a rook + knight + pawn,
and so on.

Some people prefer the values below. For example, Max Euwe (World Champion
1935-1937), who was a well-known chess educator in the middle part of the
20th century:

- Queen: 9.5 points
- Rook: 4.5 points
- Bishop: 3 points
- Knights: 3 points
- Pawn: 1 points

So a bishop and knight are still worth three pawns, but Euwe states that a
rook cannot be exactly compensated. A knight + pawn is just a tiny bit too
little, while a knight + two pawns is a tiny bit too much. Two rooks fall
_just_ short of a queen, but rook + pawn is a bit over; same with 3 light
pieces versus a queen.

Which is the best setup? It completely depends on your  playing style. I've
tested both setups with Rustic Alpha 1 and 2, and with my own hand-written
PST's (Piece-Square Tables), the first setup is the better. Versions of
Rustic with the first setup are about 30-40 Elo stronger than the ones with
the second setup. If I had written my PST's differently, it could have been
the other way around, so in your engine, try both.

For more information about PST's, see the chapter about Piece-Square
tables. Also take into account that later, the material values will be
worked directly into the Piece-Square tables, and these will then be
automatically tuned. This is going to be discussed in the "Tapering and
Tuning" chapter.

## Implementation

Implementation difficulty of counting the material balance is in the easy
to lower medium range. We will be discussing two methods of calculating the
material balance: counting from scratch, and keeping the value updated
incrementally. (Rustic uses the second approach.)

### Value per piece type

Rustic has a struct holding all the piece types in its Board definitions:

```rust,ignore
pub struct Pieces;
impl Pieces {
    pub const KING: Piece = 0;
    pub const QUEEN: Piece = 1;
    pub const ROOK: Piece = 2;
    pub const BISHOP: Piece = 3;
    pub const KNIGHT: Piece = 4;
    pub const PAWN: Piece = 5;
    pub const NONE: Piece = 6;
}
```

Using this, we can create an array which assigns a numerical value to each
piece. These are implementations of the values from above:

```rust,ignore
pub const PIECE_VALUES: [u16; 6] = [0, 900, 500, 320, 310, 100];
// --- or this one ---
pub const PIECE_VALUES: [u16; 6] = [0, 980, 475, 320, 310, 100];
```

So, this table can be indexed by:

```rust,ignore
let v = PIECE_VALUES[Pieces::Queen]; // v = 900 here.
let v = PIECE_VALUES[Pieces::Pawn]; // v = 100 here.
```

There are two differences with the values we saw earlier in the chapter.

First, the evaluation is MUCH faster if we only use integers. If at all
possible, we will never use floating point numbers while calculating
anything. (Except, maybe, in the very last step before outputting
something.) This is the reason why we have multiplied all the values by one
hundred. This gives the evaluation a way to think in terms of "parts of a
pawn", by using values such as 50 or 10, instead of 0.5 or 0.1.

Second, in _most_ chess engines, it turns out to be better to have the
bishop be worth slightly more than the knight, and that a light piece is a
bit better to have than three pawns. This causes the engine to not trade a
bishop for a knight (all else being equal), and it'll keep the light piece
instead of trading it for three pawns.

Again, don't fret too much about these values because in later stages of
engine development, they will be merged with the Piece-Square Tables and
automatically tuned.

### Counting vs. Updating

#### Counting from scratch

There are two ways you could know the material balance when evaluating the
position: you can count the material at that point, or you update
the value incrementally.

Counting the material balance at the point where you need to know it is
very straightforward. Below is a piece of pseudo-code. This doesn't come
from Rustic (and it isn't even Rust), because the engine doesn't do it this
way. This implementation is given for completeness' sake, because the
difference of "counting vs. updating" will crop up again later, when
implementing the PST's. We can then skip this example.

Here's the pseudo-code to count the material balance for use in the
evaluation function:

```rust,ignore
function count_material(position) {
    let value_white = 0;
    let value_black = 0;

    // For example, piece-type == knight
    for each piece_type in all_piece_types {
        // For all knights on the board
        for each piece in piece_type {
            if piece is a white piece {
                value_white += PIECE_VALUES[piece];
            } else if piece is a black piece {
                value_black += PIECE_VALUES[piece];
            }
        }
    }

    let balance =  value_white - value_black;
    return balance;
}
```

That is basically it. Run through all the pieces, and add the piece's value
to either the white or black counter. Then subtract black from white, and
you have the balance: if positive, white is ahead, if negative, black is
ahead. If material balance is exactly equal, then "balance" will obviously
be zero.

#### Incrementally updating

Counting from scratch works perfectly well, but it is slow: each time the
engine evaluates a position, it has to calculate the material balance all
over again. At most it only has to add 2x 16 values and then subtract the
results, but think about the fact that this has to be done millions and
millions of times a second during a search run.

When playing a game of chess yourself, you're not going to re-count the
pieces from scratch each time the position changes. At the beginning of the
game you know the material balance is equal. When you move a piece and
there is an exchange, you can then adjust the material balance to the new
situation.

This is exactly what Rustic Alpha 1 and 2 do, and it's called an
incremental update. To achieve this, Rustic keeps the material in a game
state struct, along with some other things it needs to know throughout the
game:

```rust,ignore
pub struct GameState {
    pub active_color: u8,
    pub castling: u8,
    pub halfmove_clock: u8,
    pub en_passant: Option<u8>,
    pub fullmove_number: u16,
    pub zobrist_key: u64,
    pub material: [u16; Sides::BOTH],
    pub psqt: [i16; Sides::BOTH],
    pub next_move: Move,
}
```

The "material" member keeps the total material value for each side
throughout the game. It is initialized in the Board class, when the engine
starts:

```rust,ignore
fn init(&mut self) {
    // ...

    let material = material::count(&self);
    self.game_state.material[Sides::WHITE] = material.0;
    self.game_state.material[Sides::BLACK] = material.1;

    // ...
}
```

The function counting the material is an implementation of the
pseudo-code given before, in Rust:

```rust,ignore
pub fn count(board: &Board) -> (u16, u16) {
    // Material values for white and black.
    let mut white_material: u16 = 0;
    let mut black_material: u16 = 0;

    // Make a shorthand for the white and black piece bitboards
    let bb_w = board.bb_pieces[Sides::WHITE];
    let bb_b = board.bb_pieces[Sides::BLACK];

    // Iterate over white and black bitboards by piece_type
    for (piece_type, (w, b)) in bb_w.iter().zip(bb_b.iter()).enumerate() {
        // Get white and black pieces of the type "piece"
        let mut white_pieces = *w;
        let mut black_pieces = *b;

        // Add material values for white (for each piece of this piece type)
        while white_pieces > 0 {
            white_material += PIECE_VALUES[piece_type];
            bits::next(&mut white_pieces);
        }

        // Add material values for black (for each piece of this piece type)
        while black_pieces > 0 {
            black_material += PIECE_VALUES[piece_type];
            bits::next(&mut black_pieces);
        }
    }

    // Return values
    (white_material, black_material)
}
```

After starting, the engine will know the total material value for both
white and black. As soon as a piece is removed from a square (by capture,
or by moving it) or put down on a square the material value is updated:

```rust,ignore
pub fn remove_piece(&mut self, side: Side, piece: Piece, square: Square) {
    // ...
    self.game_state.material[side] -= PIECE_VALUES[piece];
    // ...
}

pub fn put_piece(&mut self, side: Side, piece: Piece, square: Square) {
    // ...
    self.game_state.material[side] += PIECE_VALUES[piece];
    // ...
}
```

Note that this will even take promotions into account: remove a pawn from
the 7th rank and thus subtract _the knight's_ value from the material
count, and then put a queen down, adding _the queen's_ value back to the
material count.

Obviously, the remove_piece() and put_piece() functions also remove and put
the piece, and they keep other incremental scores next to the material,
such as as the incrementally updated PST's. That code has been omitted.

With this implementation, the engine knows the white and black piece value
count throughout the entire game, and the evaluation function only has to
do one thing to use that as a basis:

```rust,ignore
pub fn evaluate_position(board: &Board) -> i16 {
    // ...

    let w_material = board.game_state.material[Sides::WHITE] as i16;
    let b_material = board.game_state.material[Sides::BLACK] as i16;

    // Base evaluation, by counting material.
    let mut value = w_material - b_material;

    // ...

    // Return evaluation value.
    value
}
```

As you can see, the re-counting of the piece value from scratch has been
reduced to just getting the white and black values from the board and then
subtracting them.

This exact same technique will also be used to implement the Piece-Square
tables incrementally, which will be the next chapter. It is exactly the
same, only there are 6 arrays instead of 2 (one for each piece type instead
of each side), and 64 integers per array instead of 6 (one for each square
instead of for each piece type).

Now that we know how we can keep the material count, we can move on to the
next part of the evaluation: Piece-Square Tables. These will add a little
bit of positional knowledge to the chess engine on top of the material
count, so it will have an idea of what to do with a piece.