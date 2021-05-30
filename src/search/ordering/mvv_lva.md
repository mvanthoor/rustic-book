# MVV-LVA

MVV-LVA stands for Most Valuable Victim, Least Valuable Attacker. This is a
move ordering technique that does exactly what it describes: it orders the
capture moves, ordered from the strongest to the weakest. The more valuable
the captured piece is, and the less valuable the attacker is, the stronger
the capture will be, and thus it will be ordered higher in the move list.
As a consequence, the alpha-beta function will search the stronger captures
first, which causes it to find better moves faster, and thus it can
disregard large portions of the search tree.

In Rustic, MVV-LVA has been implemented since the first version, Alpha 1.
It is recommended that you implement this function directly after you get
iterative deepening and alpha-beta working. It is instrumental to getting
your engine to play decent chess.

The function is implemented as a two-dimensional array:

```rust
// MVV_VLA[victim][attacker]
pub const MVV_LVA: [[u8; NrOf::PIECE_TYPES + 1]; NrOf::PIECE_TYPES + 1] = [
    [0, 0, 0, 0, 0, 0, 0],       // victim K, attacker K, Q, R, B, N, P, None
    [50, 51, 52, 53, 54, 55, 0], // victim Q, attacker K, Q, R, B, N, P, None
    [40, 41, 42, 43, 44, 45, 0], // victim R, attacker K, Q, R, B, N, P, None
    [30, 31, 32, 33, 34, 35, 0], // victim B, attacker K, Q, R, B, N, P, None
    [20, 21, 22, 23, 24, 25, 0], // victim N, attacker K, Q, R, B, N, P, None
    [10, 11, 12, 13, 14, 15, 0], // victim P, attacker K, Q, R, B, N, P, None
    [0, 0, 0, 0, 0, 0, 0],       // victim None, attacker K, Q, R, B, N, P, None
];
```

As you can see, the two-dimensional array is addressed in the
"victim"-"attacker" order, just like MVV-LVA says. Each combination has a
value. For example:

```rust
let value1 = MVV_LVA[Pieces::BISHOP][Pieces::PAWN];
```

In this case, the victim is a Bishop, while the attacker is pawn. The
value of this capture will therefore be 35. Another capture could be:

```rust
let value2 = MVV_LVA[PIECES::QUEEN][Pieces::ROOK];
```

Now, the a Queen can be captured by a Rook. The value of this capture would
thus be 52. If the engine has a choice between both captures, the second
one will be ordered higher in the move list, and the engine will search it
first, beecause captureing the queen using a rook is probably better (5
points of material gain), than capturing the bishop using a pawn (2 points
of material gain.)

This is how the ordering is implemented, when MVV_LVA is the only sort
scoring functionality in the engine:

```rust
pub fn score_moves(ml: &mut MoveList) {
    for i in 0..ml.len() {
        let m = ml.get_mut_move(i);
        let value = MVV_LVA[m.captured()][m.piece()];
        m.add_score(value);
    }
}
```

I said this would be easy. We just run through the move list, grab the
combination of the victim and attacker from the MVV_LVA-array, and we add
it to the move's score. (Because this is the only place in the engine where
moves are scored for sorting, the member function _add_score()_ is renamed
to _set_sort_score()_ in later versions of Rustic, so you may see
_set_sort_score()_ in other move sorting chapters.)

So, what about the 0's in the table? The number of elements for the rows
and columns in the table are "NrOf::PIECE_TYPES + 1". There are 6 piece
types in chess: King, Queen, Rook, Bischop, Knight, Pawn. Rustic also has a
piece type called Pieces::NONE, which is used when there's no piece on a
square, or, in the move format, no piece is in the Promotion or Captured
fields. That is where the +1 comes from.

In MVV_LVA move sorting, Piece::NONE cannot be the victim and it also
cannot be the attacker. You can't capture TO an empty square. You also
can't capture FROM an empty square. Therefore all the cells that have
Piece::NONE as part of the combination are 0. (Actually, a move FROM an
empty square isn't even possible... supposing your engine has no bugs in
the move generator.) The top row is also 0, because the King cannot be
captured.

Setting up the MVV_LVA array this way makes the scoring function somewhat
simpler because you don't have to check if the FROM and TO square actually
contain pieces, and that you're not capturing a king.

