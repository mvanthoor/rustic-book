
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Initialization](#initialization)
  - [The init-function](#the-init-function)
  - [Initializing pieces per side](#initializing-pieces-per-side)
  - [Initializing the piece list](#initializing-the-piece-list)
  - [Initializing the game state](#initializing-the-game-state)
  - [Initializing the PSQT values](#initializing-the-psqt-values)
  - [Incremental updates](#incremental-updates)
  - [Next...](#next)

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

## Initializing the game state

The game state has multiple parts to it:

- It needs to calculate the current Zobrist key. (See the chapter about
  [Zobrist Hasing](./zobrist_hashing.md) for more information on how to set
  up the zobrist hash values.) The key is calculated like this:

```rust,ignore
pub fn init_zobrist_key(&self) -> ZobristKey {
    // Keep the key here.
    let mut key: u64 = 0;

    // Same here: "bb_w" is shorthand for
    // "self.bb_pieces[Sides::WHITE]".
    let bb_w = self.bb_pieces[Sides::WHITE];
    let bb_b = self.bb_pieces[Sides::BLACK];

    // Iterate through all piece types, for both white and black.
    // "piece_type" is enumerated, and it'll start at 0 (KING), then 1
    // (QUEEN), and so on.
    for (piece_type, (w, b)) in bb_w.iter().zip(bb_b.iter()).enumerate() {
        // Assume the first iteration; piece_type will be 0 (KING). The
        // following two statements will thus get all the pieces of
        // type "KING" for white and black. (This will obviously only
        // be one king, but with rooks, there will be two in the
        // starting position.)
        let mut white_pieces = *w;
        let mut black_pieces = *b;

        // Iterate through all the piece locations of the current piece
        // type. Get the square the piece is on, and then hash that
        // square/piece combination into the zobrist key.
        while white_pieces > 0 {
            let square = bits::next(&mut white_pieces);
            key ^= self.zobrist_randoms.piece(Sides::WHITE, piece_type, square);
        }

        // Same for black.
        while black_pieces > 0 {
            let square = bits::next(&mut black_pieces);
            key ^= self.zobrist_randoms.piece(Sides::BLACK, piece_type, square);
        }
    }

    // Hash the castling, active color, and en-passant state.
    key ^= self.zobrist_randoms.castling(self.game_state.castling);
    key ^= self
        .zobrist_randoms
        .side(self.game_state.active_color as usize);
    key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);

    // Done; return the key.
    key
}
```

The function seems a bit complicated at first sight because of the nested for and while
loops, but with the help of the comments it should still be fairly obvious.
What it does is basically:

- Loop through all the white and black pieces
- Determine the piece type per color (e.g.: White Knight)
- See on which square the piece is
- Hash the Zobrist Key for that color/piece-type/square combination into
  the "key" variable.
- Hash the Zobrist Keys for castling, active color, and en-passant state
  into the key.
- When done, return the key. This is the Zobrist Key for the position which
  is on the board.

(_The next two parts tie into the engine's evaluation. Note that the
sections on Evaluation may not be complete yet._)

Next, _if the engine uses a Tapered Evaluation, the game phase has te be
calculated. We haven't discussed this yet, but you can find more
information about the game phase in the [Evaluation
Chapter](../evaluation/evaluation.md). What it does is tell the engine
which phase the game is in: opening, middle game, or endgame. The function
look fairly similar to the Zobrist Key initialization:

```rust,ignore
pub fn count_phase(board: &Board) -> i16 {
    let mut phase_w: i16 = 0;
    let mut phase_b: i16 = 0;
    let bb_w = board.bb_pieces[Sides::WHITE];
    let bb_b = board.bb_pieces[Sides::BLACK];

    for (piece_type, (w, b)) in bb_w.iter().zip(bb_b.iter()).enumerate() {
        let mut white_pieces = *w;
        let mut black_pieces = *b;

        while white_pieces > 0 {
            phase_w += EvalParams::PHASE_VALUES[piece_type];
            bits::next(&mut white_pieces);
        }

        while black_pieces > 0 {
            phase_b += EvalParams::PHASE_VALUES[piece_type];
            bits::next(&mut black_pieces);
        }
    }

    phase_w + phase_b
}
```

It runs through all the white and black pieces per piece type. It adds all
the phase values for each piece type, for white and black. At the end it
adds both phases together and returns the result. The more pieces that are
on the board, the higher the end result will be; so a higher phase value
will mean that the game is in an earlier stage. An endgame has less pieces
on the board, so the phase value will be lower.

Lastly, the next move is set to 0, as there is no history and thus no next
move, because we've just initialized the board.

## Initializing the PSQT values

This part also ties into the engines evaluation. It sets up the initial
PSQT value for each side. See the sections on [Piece Square
Tables](../evaluation/psqt.md) and [Tapered
PSQT's](../evaluation/tapering.md) for more information on what these
tables do in the engine. (As a short recap: they tell the engine what
squares are good for which piece type, as a rule of thumb, which is the
starting point of the evaluation function.) Rustic Alpha 3 and before had a
simple evaluation with one value per piece/square combination, but Rustic 4
has a tapered evaluation with two values per piece/square combination. When
setting up the position, we need to know the PSQT-value for white and for
black. Again, the function is similar to the Zobrist and phase
initialization function:

```rust,ignore
pub fn psqt_apply(board: &Board, psqt_set: &PsqtSet) -> (W, W) {
    let mut psqt_w_mg: i16 = 0; // White middle-game value
    let mut psqt_w_eg: i16 = 0; // White end-game value
    let mut psqt_b_mg: i16 = 0; // Black middle-game value
    let mut psqt_b_eg: i16 = 0; // Black end-game value
    let bb_white = board.bb_pieces[Sides::WHITE]; // Array of white piece bitboards
    let bb_black = board.bb_pieces[Sides::BLACK]; // Array of black piece bitboards

    // Iterate through the white and black bitboards (at the same time.)
    for (piece_type, (w, b)) in bb_white.iter().zip(bb_black.iter()).enumerate() {
        let mut white_pieces = *w; // White pieces of type "piece_type"
        let mut black_pieces = *b; // Black pieces of type "piece_type"

        // Iterate over pieces of the current piece_type for white.
        while white_pieces > 0 {
            let square = bits::next(&mut white_pieces);
            psqt_w_mg += psqt_set[piece_type][FLIP[square]].mg();
            psqt_w_eg += psqt_set[piece_type][FLIP[square]].eg();
        }

        // Iterate over pieces of the current piece_type for black.
        while black_pieces > 0 {
            let square = bits::next(&mut black_pieces);
            psqt_b_mg += psqt_set[piece_type][square].mg();
            psqt_b_eg += psqt_set[piece_type][square].eg()
        }
    }

    (W(psqt_w_mg, psqt_w_eg), W(psqt_b_mg, psqt_b_eg))
}
```

So this function iterates over the white and black pieces per piece type
and then calculates the total "PSQT White Middle Game" and "PSQT White End
Game" value; and obviously the same for black. It ends up with 4 values,
which are then returned. (The "W" is just a type wrapper which means
"Weight", so two values per side can be returned at once.)

## Incremental updates

After setting all these values, they are never recalculated from scratch
like this during a game. This would take too much time, because the values
change with every move. It would be a massive waste: why, for example,
would we need to run over the pawns, bishops, kings, queens, etc... if it
was a knight that moved? It would be much faster to take the knight out of
all the values according to its starting square and put it back in
according to the destination square. If it captures something on the
destination square, we take that piece out of the values and we're done. We
will see how this works exactly in the section about [Moving
Pieces](../board_functionality/moving_pieces.md).

## Next...

Now that board creation and initialization are done, we can finally start
looking at the [Board
Functionality](../board_functionality/board_functionality.md), where we
define and write functions to make the board usable by giving us
information about the position, or moving pieces.
