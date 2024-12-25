# Board struct

We're almost done with one of the chess engine's basic components. All the
parts for building the data structure for the board presentation are
finally finished, and we only have to neatly pack them up.

To recap, we have the following data structures that make up the board when
they're all being put together:

- Bitboards
    - 6 per side; one for each piece
    - 1 per side for all pieces of that side
- Game State
- Game History
- Piece List
- Zobrist Keys.

So, without further ado, here's the Board struct:

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

By instantiating the Board struct, we can create a new board for
the engine to work with. Sometimes it is useful for the engine to use a
copy of the main board to work with, so by putting all the needed data
structures in one Board struct, it is easy to either create copies or even
new boards when required.

Another advantage of packaging everything up in an overarching struct is
that we can now create and initialize the board using its own constructor
and init function. This will be discussed in [the next chapter.]()
