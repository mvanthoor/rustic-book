
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Support Functions](#support-functions)
  - [Explanation](#explanation)
  - [List of support functions](#list-of-support-functions)
    - [new](#new)
    - [reset](#reset)
    - [get_pieces](#get_pieces)
    - [get_bitboards](#get_bitboards)
    - [occupancy](#occupancy)
    - [us](#us)
    - [opponent](#opponent)
    - [king_square](#king_square)
    - [has_bishop_pair](#has_bishop_pair)
  - [Example](#example)

<!-- /code_chunk_output -->

# Support Functions

## Explanation

These functions are easy to understand. They make using the board easier.
Often they only wrap one or a few lines of code, but because these lines
are used throughout the engine it makes the code much cleaner. One example
is this the us() function:

```rust,ignore
pub fn us(&self) -> usize {
    self.game_state.active_color as usize
}
```

This function just returns the active_color, which is white or black. If we
are doing something where piece color is important, like castling (white
castles on different squares than black), we can now write this:

```rust,ignore
if board.us() == WHITE { }
```

instead of this:

```rust,ignore
if (self.game_state.active_color as usize) == WHITE { }
```

It's the same thing, but the first version is much easier to understand
because it literally reads "if "us" is white, then ...", while the second
version needs a bit of thinking to understand what is going on.

>__Sidenote:__ This concept isn't new. Many engines use this. Rustic just
>uses us() for the side to move and opponent() for the other side. Other
>engines may use names such as we() or stm() (side to move) for the active
>color and them() or just !us() for the opponent.

Below is a list of all the support functions accompanied by a short
description.

## List of support functions

### new

Creates a brand new board.

```rust,ignore
pub fn new() -> Self {
    Self {
        bb_pieces: [[EMPTY; NrOf::PIECE_TYPES]; Sides::BOTH],
        bb_side: [EMPTY; Sides::BOTH],
        game_state: GameState::new(),
        history: History::new(),
        piece_list: [Pieces::NONE; NrOf::SQUARES],
        zobrist_randoms: Arc::new(ZobristRandoms::new()),
    }
}
```

### reset

Resets the board so a new game can be started.

```rust,ignore
pub fn reset(&mut self) {
    self.bb_pieces = [[EMPTY; NrOf::PIECE_TYPES]; Sides::BOTH];
    self.bb_side = [EMPTY; Sides::BOTH];
    self.game_state.clear();
    self.history.clear();
    self.piece_list = [Pieces::NONE; NrOf::SQUARES];
}
```

### get_pieces

Return a bitboard with all the locations of a certain piece type for a
particular side.

```rust,ignore
pub fn get_pieces(&self, side: Side, piece: Piece) -> Bitboard {
    self.bb_pieces[side][piece]
}
```

### get_bitboards

Get all the bitboards of all pieces for a particular side.

```rust,ignore
pub fn get_bitboards(&self, side: Side) -> &[u64; NrOf::PIECE_TYPES] {
    &self.bb_pieces[side]
}
```

### occupancy

Returns a bitboard containing all the pieces on the board. This is called
the "board occupancy": the bitboard contains a 1 for each square on which a
piece is located. This means that !occupancy() will yield all the empty
squares.(Maybe I'll add an empty_squares() function some day.)

```rust,ignore
pub fn occupancy(&self) -> Bitboard {
    self.bb_side[Sides::WHITE] | self.bb_side[Sides::BLACK]
}
```

### us

Returns the side to move.

```rust,ignore
pub fn us(&self) -> usize {
    self.game_state.active_color as usize
}
```

### opponent

Returns the side that is NOT to move.

```rust,ignore
pub fn opponent(&self) -> usize {
    (self.game_state.active_color ^ 1) as usize
}
```

### king_square

Returns the square the king of the given side is currently on.

```rust,ignore
pub fn king_square(&self, side: Side) -> Square {
    self.bb_pieces[side][Pieces::KING].trailing_zeros() as Square
}
```

### has_bishop_pair

Asks the board to determine if a particular side has the bishop pair or
not. The definition of a "bishop pair" is that one side needs to have at
least two bishops, one of which must be light-squared and the other
dark-squared. Because this is a somewhat more complicated function, it has
it's own section: [Detecting the Bishop Pair](./detecting_bishop_pair.md)

```rust,ignore
pub fn has_bishop_pair(&self, side: Side) -> bool {
    let mut bishops = self.get_pieces(side, Pieces::BISHOP);
    let mut white_square = 0;
    let mut black_square = 0;

    if bishops.count_ones() >= 2 {
        while bishops > 0 {
            let square = bits::next(&mut bishops);

            if Board::is_white_square(square) {
                white_square += 1;
            } else {
                black_square += 1;
            }
        }
    }

    white_square >= 1 && black_square >= 1
}
```

## Example

Often we need to know on which square a king is. This would be a hassle to
do by hand because we'll always need two lines (or one complicated-looking
consolidated line) to do it. Using the support functions we can just ask
the board what we want to know:

```rust,ignore
let our_king_square = board.king_square(board.us());
let their_king_square = board.king_square(board.opponent());
```

This is the essence of these functions. As soon as you find yourself
repeating a line over and over to get a certain piece of information it
will be advantageous to wrap it in a support function. As your engine
grows, more and more of these functions will emerge. Now we're finally
getting to a place where we can start using the board. To do so, we first
need to set it up. We have a function for this, which sets up the board
according to a FEN-string. [This will be explained in the next
section.](./handling_fen_strings.md)
