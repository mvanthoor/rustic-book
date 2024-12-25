# Initialization

In the previous sections we discussed and wrote all the parts we need to
create the Board data structure. Initialization of a new board consists of
two steps:

- Create a new board or clone an existing one
- [Set up the position using the FEN-function](../board_functionality/handling_fen_strings.md)
- Initializing it with default values

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
board will always have the correct values when setting up a new position,
so you don't have to call the init-function yourself (and hence, you can't
forget this).

## The init-function

The init function is split into different parts, each handling one part of
the board's data. All these parts are then called from board.init():

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


