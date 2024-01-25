# The evaluation function

Here we are, finally: after implementing the board, move generator and
search, the evaluation function is the last step to make the engine play
actual, somewhat sane chess. In the previous chapters we've looked at
material counting, PSQT's, and how to implement them. In this chapter,
which will be very short, we'll look at implementing the evaluation itself.
I'm assuming you've chosen to put the material values directly into the
PSQT, and use the incremental update implementation as described earlier,
because it makes everything run faster. This also makes implementing the
first version of the PSQT-only evaluation easier. Here it is:

```rust,ignore
pub fn evaluate_position(board: &Board) -> i16 {
    const KING_ONLY: i16 = 300; // PSQT-points
    let side = board.game_state.active_color as usize;
    let w_psqt = board.game_state.psqt[Sides::WHITE];
    let b_psqt = board.game_state.psqt[Sides::BLACK];
    let mut value = w_psqt - b_psqt;

    // If one of the sides is down to an (almost) lone king, apply the KING_EDGE
    // PSQT to drive that king to the edge and mate it, or if it's your own king,
    // to run it out into the center to be more active.

    if w_psqt < KING_ONLY || b_psqt < KING_ONLY {
        let w_king_edge = KING_EDGE[board.king_square(Sides::WHITE)];
        let b_king_edge = KING_EDGE[board.king_square(Sides::BLACK)];
        value += w_king_edge - b_king_edge;
    }

    value = if side == Sides::BLACK { -value } else { value };

    value
}
```

First we create a const some shorthand variables. We get the side to move
and both the white and black PSQT values from the game_state struct. Then
we compute the base value by subtracting the black PSQT value from the
white one.

In the next step is where the KING_EDGE table comes into play. When either
the white or black PSQT value is lower than the KING_ONLY value, which is
set at 300 (one light piece), the KING_EDGE table is applied for both
sides. If a king is near the edge, the evaluation for that side will incur
a huge penalty. This forced the engine to bring the king out into the
middle. When it does that, is controlled by the KING_ONLY variable; the
higher that number is, the earlier the king will come out, so this can be
tweaked depending what playing style you want from the engine, or what is
strongest. You'd need to test different values.

>__Sidenote:__ When implementing a tapered evaluation, this way of doing
>things will obviously disappear, because the king will already have
>endgame values in its PSQT.

This function calculates the evaluation from white's point of view: a
positive value means "white is better", a negative value means "black is
better". Alpha/Beta requires the value returned from the viewpoint of the
side that is being evaluated. Therefore if it is black to move, the value
must first be flipped to black's viewpoint before it can be returned.

That it: the first version of a PSQT-only evaluation is now implemented. In
the next step we'll look into how the evaluation can be tapered, to contain
both opening and endgame values in one PSQT.