# Game State

The GameState struct is really simple. It is just a collection of variables
which represent the state of things you can't immediately see when looking
at a board. This is the struct:

```rust,ignore
pub struct GameState {
    pub active_color: u8,
    pub castling: u8,
    pub half_move_clock: u8,
    pub en_passant: Option<u8>,
    pub fullmove_number: u16,
    pub zobrist_key: u64,
    pub phase_value: i16,
    pub psqt_value: [W; Sides::BOTH],
    pub next_move: Move,
}
```

Here are the descriptions for each of the state variables.

| Variable           | Meaning                           |
|--------------------|-----------------------------------|
| 1. active_color    | Side to move                      |
| 2. castling        | Castling permissions              |
| 3. half_move_clock | Half moves played                 |
| 4. en_passant      | Active en-passant square, if any  |
| 5. fullmove_number | Total number of full moves played |
| 6. zobrist_key     | Zobrist Key                       |
| 7. phase_value     | Evaluation Phase Value            |
| 8. psqt_value      | Piece Square Table Value          |
| 9. next_move       | The move played in this position  |

The variable *active_color* just holds which side is to move in this
position. The castling permissions variable is an integer, representing
which side can castle where.

The variable *en_passant* marks the "active" en_passant square, if any.
Imagine there is a black pawn on _d4_, and white plays _c2-c4_. The c4-pawn
can now be captured by _d4xc3 ep_, so after the move _c2-c4_, the square
_c3_ becomes the active en-passant square. Note that this is only set
directly after the white _c2-c4_ two-square pawn move, and it will be unset
after whatever move black decides to play. This is because en-passant is
only a valid move directly after a two-square pawn push.

Sometimes the values for half_move_clock and fullmove_number can be
confusing, so we'll discuss them a bit further to make sure these are
understood correctly.

In day-to-day chess talk, chess we tend to call it a "move" when one side
moves a piece. Technically though, this is only a half-move. The full move
is only complete when both white and black have moved a piece. It is often
confusing when applying the 50 **move** rule for draws: it means that _both
white and black_ must have made 50 moves, so the total is 100 half-moves.

This is what the half_move_clock means: it is the number of half moves made
since the last pawn push or piece capture. As soon as a pawn is moved or a
piece is captured, the half_move_clock is reset to 0.

Consequently, the fullmove_number is the number of completed turns in the
game where both white and black have done their half-move part.

In the previous chapter we discussed how the Zobrist Key works; this is the
place where it is kept, and it is incrementally updated each move so it
does not have to be recalculated from scratch.  This saves a lot of time
that can be spent elsewhere; either for searching further, or in the
evaluation function.

The Phase Value and PSQT Value are calculated by the evaluation function to
try and determining who is better in the position. Because this is a
lenghty job, just like the Zobrist Key, these values are calculated at the
start of the game and put into these variables. As soon as a piece is
moved, the variables can be updated. This works similar to the Zobrist
Keys: when a piece moves, remove the value it had on the previous square,
then re-add the value it now has on the new square. For more information
about this, see the chapter [about the Evaluation
function](../evaluation/evaluation.md).

The variable next_move is not absolutely necessary, but it is very useful
when making and unmaking moves during the search and gameplay. It holds the
move that was played in this position. It ends up in the history table.
What happens is that the engine, before making a move, captures the current
state, adds the move to be played, and pushes this into the history table.
This way the engine doesn't need to store the entire board; it can just get
the previous game and undo the move that was made there, to restore the
previous position. This saves having to store the entire board in the
history.

That is it for the GameState struct. Now we only need a history struct
which keeps track of the entire game's history. Believe it or not, but this
is even simpler than the GameState struct. We'll discuss it in the next
chapter.