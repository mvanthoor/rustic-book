
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Moving Pieces](#moving-pieces)
  - [Explanation](#explanation)
  - [Incremental updates](#incremental-updates)
  - [List of movement functions](#list-of-movement-functions)
    - [Remove piece from square](#remove-piece-from-square)
    - [Put piece onto square](#put-piece-onto-square)
    - [Move piece from one square to another](#move-piece-from-one-square-to-another)
    - [Set a square to be the en-passant destination](#set-a-square-to-be-the-en-passant-destination)
    - [Clear the en-passant square.](#clear-the-en-passant-square)
    - [Swap active side from one to the other](#swap-active-side-from-one-to-the-other)
    - [Update castling permissions](#update-castling-permissions)

<!-- /code_chunk_output -->



# Moving Pieces

## Explanation

First we'll need to explain the difference between "moving pieces" and
"making a move."

In this section we're looking at moving the pieces around on the board
Moving the pieces on the board is distinctly different from making and
unmaking moves, though these functionalities are obviously closely related.
"Moving a piece" means that you pick it up from one square and put it down
on another. If there's a piece on that other square, you remove it from the
board. The functions which move pieces on the board should always do
exactly this, without checking the legality of the action.

Making and unmaking moves is discussed in the next section and describes
how moves are made and retracted. This functionality uses the piece
movement functions, but make() assures that the executed move is legal for
the position. Unmake() retracts the last move and restores the previous
board state.

## Incremental updates

In the [initialization section](../board_representation/init.md) we've seen multiple mentions of
"This value will be updated incrementally." In this section we're going to see how this works. To
recap: if a piece is moved, _most_ of the board does not change. This means that _most_ of the
values associated with the position do not change, so it would be a waste of time to calculate every
value from scratch again and again as the pieces move around. Because the search function makes and
unmakes moves constantly, recalculating everything all the time would massively impact the engine's
search speed and thus its playing strength. The solution is an incremental update. First, we
calculate a value when the position is set up, and then we adjust it for the new situation if a
piece moves. Often this means:

- Remove the piece from the starting square. Take the piece's information out of the value.
- Put the piece on the destination square. Add the new information back into the value.
- If a piece was captured on the destination square, remove that piece's information from the value.

We then do this for every incrementally updated value.

## List of movement functions

### Remove piece from square

This function removes a piece from a square. As you can see, the pieces it taken out of the piece
type bitboards, pieces per side bitboard, the piece list, and the zobrist key is updated to not
include this piece. Then the piece is removed from the phase_value in the game state and removed
from the PSQT value.

```rust,ignore
// Remove a piece from the board, for the given side, piece, and square.
pub fn remove_piece(&mut self, side: Side, piece: Piece, square: Square) {
    self.bb_pieces[side][piece] ^= BB_SQUARES[square];
    self.bb_side[side] ^= BB_SQUARES[square];
    self.piece_list[square] = Pieces::NONE;
    self.game_state.zobrist_key ^= self.zobrist_randoms.piece(side, piece, square);

    // Incremental updates
    // =============================================================
    self.game_state.phase_value -= EvalParams::PHASE_VALUES[piece];
    let square = Board::flip(side, square);
    self.game_state.psqt_value[side].sub(EvalParams::PSQT_SET[piece][square]);
}
```

### Put piece onto square

Almost self-explanatory, this is the counterpart of remove_piece(), but it adds the piece into all
the bitboards, lists, and values when it lands onto a square.

```rust,ignore
pub fn put_piece(&mut self, side: Side, piece: Piece, square: Square) {
    self.bb_pieces[side][piece] |= BB_SQUARES[square];
    self.bb_side[side] |= BB_SQUARES[square];
    self.piece_list[square] = piece;
    self.game_state.zobrist_key ^= self.zobrist_randoms.piece(side, piece, square);

    // Incremental updates
    // =============================================================
    self.game_state.phase_value += EvalParams::PHASE_VALUES[piece];
    let square = Board::flip(side, square);
    self.game_state.psqt_value[side].add(EvalParams::PSQT_SET[piece][square]);
}
```

### Move piece from one square to another

Because a piece doesn't suddenly disappear from the board (except when captured), it is very
convenient to have a "move piece" function that combines the removing and putting parts into one
function.

```rust,ignore
pub fn move_piece(&mut self, side: Side, piece: Piece, from: Square, to: Square) {
    self.remove_piece(side, piece, from);
    self.put_piece(side, piece, to);
}
```

### Set a square to be the en-passant destination

When a pawn moves two steps forward, the square behind the pawn becomes the en-passant square. This
function first removes the current en-passant square from the game state (if any). It then sets the
new en-passant square and adds this back into the Zobrist key.

```rust,ignore
pub fn set_ep_square(&mut self, square: Square) {
    self.game_state.zobrist_key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);
    self.game_state.en_passant = Some(square as u8);
    self.game_state.zobrist_key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);
}
```

### Clear the en-passant square.

This function clears the en-passant square. It first removes the current ep-square from the Zobrist
Key (if any), and then sets the square to None. It adds this back into the key.

```rust,ignore
pub fn clear_ep_square(&mut self) {
    self.game_state.zobrist_key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);
    self.game_state.en_passant = None;
    self.game_state.zobrist_key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);
}
```

### Swap active side from one to the other

It's almost becoming boring. I think you can reiterate what's been said before. All together now:
"This function removes the current active side from the Zobrist Key. Then it swaps the sides and
adds it back into the key." Now sing this for 10 minutes to the rhythm of the Picard Song. If you
don't know it, go and find it on Youtube. It's hilarious. Especially the phrase "I'm not entitled to
ramble on about something everyone knows" is fitting here.

```rust,ignore
pub fn swap_side(&mut self) {
    self.game_state.zobrist_key ^= self
        .zobrist_randoms
        .side(self.game_state.active_color as usize);
    self.game_state.active_color ^= 1;
    self.game_state.zobrist_key ^= self
        .zobrist_randoms
        .side(self.game_state.active_color as usize);
}
```

### Update castling permissions

Ah, what the heck. I'll not even explain it now. If you don't understand, go read the comment on the
previous function and sing the Picard Song for another 30 minutes.

```rust,ignore
pub fn update_castling_permissions(&mut self, new_permissions: u8) {
    self.game_state.zobrist_key ^= self.zobrist_randoms.castling(self.game_state.castling);
    self.game_state.castling = new_permissions;
    self.game_state.zobrist_key ^= self.zobrist_randoms.castling(self.game_state.castling);
}
```