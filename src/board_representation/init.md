
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Initialization](#initialization)
  - [The init-function](#the-init-function)
  - [Initializing pieces per side](#initializing-pieces-per-side)
  - [Initializing the piece list](#initializing-the-piece-list)

<!-- /code_chunk_output -->

# Initialization

In the previous sections we discussed and wrote all the parts we need to
create the Board data structure. Initialization of a new board consists of
three steps:

- Create a new board or clone an existing one
- [Set up the position using the FEN-function](../board_functionality/handling_fen_strings.md)
- Initialize it according the position that was just set up. This is
  handled by the FEN-function directly after parsing the FEN-string.

The reason why we don't initialize the board directly after creating it is
because the values in many parts of the board are dependent on what
position is set up. We could pass an optional position into the new()
constructor, which will then call the FEN-fen function to set up and
initialize the board. Maybe I'll add this in future versions of Rustic
after it is transformed into a library. In version 4 and before, the steps
of getting a usable board are:

```rust,ignore
let mut board = board::new();
board.fen_setup(FEN_START_POSITION);
```

After setting up the pieces as defined in the FEN-string, the fen_setup()
function will call board.init(). This will make sure that all parts of the
board will always have the correct values and calling the init-function
can't be forgotten.

## The init-function

First let's recall the Board struct:

```rust,ignore
pub struct Board {
    pub bb_pieces: [[Bitboard; NrOf::PIECE_TYPES]; Sides::BOTH],
    pub bb_side: [Bitboard; Sides::BOTH],
    pub game_state: GameState,
    pub history: History,
    pub piece_list: [Piece; NrOf::SQUARES],
    zobrist_randoms: Arc<ZobristRandoms>,
}
```

- bb_pieces: one bitboard per piece type per side
- bb_side: one bitboard per side with all of that side's pieces
- game_state: current state of all the board properties
- history: a list of all the moves played in the game
- zobrist_randoms: The keys used for Zobrist hashing.

Note that bb_pieces is filled by [parsing an
FEN-string](../board_functionality/handling_fen_strings.md). The
FEN-function then calls the init() function to fill out the rest of the
data. The init-function is split into different parts, each handling a
piece of information:

```rust,ignore
pub fn init(&mut self) {
    // Gather all the pieces of a side into one bitboard; one bitboard
    // with all the white pieces, and one with all black pieces.
    let pieces_per_side_bitboards = self.init_pieces_per_side_bitboards();
    self.bb_side[Sides::WHITE] = pieces_per_side_bitboards.0;
    self.bb_side[Sides::BLACK] = pieces_per_side_bitboards.1;

    // Initialize incrementally updated values.
    self.piece_list = self.init_piece_list();
    self.game_state.zobrist_key = self.init_zobrist_key();
    self.game_state.phase_value = Evaluation::count_phase(self);
    self.game_state.next_move = Move::new(0);

    // Set initial PST values: also incrementally updated.
    let pst_values = Evaluation::psqt_apply(self, &EvalParams::PSQT_SET);
    self.game_state.psqt_value[Sides::WHITE] = pst_values.0;
    self.game_state.psqt_value[Sides::BLACK] = pst_values.1;
}
```

## Initializing pieces per side

This function creates one bitboard per side containing all the piece
locations of that side. It is coded as follows:

```rust,ignore
fn init_pieces_per_side_bitboards(&self) -> (Bitboard, Bitboard) {
    let mut bb_white: Bitboard = 0;
    let mut bb_black: Bitboard = 0;

    // Iterate over the bitboards of every piece type.
    for (bb_w, bb_b) in self.bb_pieces[Sides::WHITE]
        .iter()
        .zip(self.bb_pieces[Sides::BLACK].iter())
    {
        bb_white |= *bb_w;
        bb_black |= *bb_b;
    }

    // Return a bitboard with all white pieces, and a bitboard with all
    // black pieces.
    (bb_white, bb_black)
}
```

It runs through the bitboards of all piece types per side and then collects
all the white into one bitboard and the other pieces into another. Then the
result is returned.

The reason why we want this bitboard is because it allows us to instantly
answer the question: "Is there a white piece on d6?" Also, we can combine
the two bitboards like this:

```rust,ignore
// Pseudo-code
let bb_occupancy = bb_ll_white_pieces | bb_all_black_pieces;
```

The bb_occupancy bitboard will contain all pieces on the board, which means
that !bb_occupancy contains all empty squares. This is very useful
information while generating moves.

## Initializing the piece list

The piece list has already been discussed [in its own
section](./piece_list.md), including its initialization function. I'll
shortly recap what it for. Sometimes, you need to know of what type a piece
on a square is, if any is there at all. Without the piece list, you'd take
your square number and then take a look in every one of the 12 piece type
bitboards to see if you can find a piece on that square. This is very slow.
To speed this up, we keep an array of 64 squares, with each square holding
the piece type that is there, if any. Now we can just index the square
number into the array to find out if, and what piece is on that square.

[The piece list section can be found here.](./piece_list.md)


