# MVV-LVA

(Implemented since Rustic Alpha 1.)

MVV-LVA stands for Most Valuable Victim, Least Valuable Attacker. This is a
move ordering technique that does exactly what it describes: it orders the
capture moves, ordered from the strongest to the weakest. The more valuable
the captured piece is, and the less valuable the attacker is, the stronger
the capture will be, and thus it will be ordered higher in the move list.
As a consequence, the alpha-beta function will search the stronger captures
first, which causes it to find better moves faster, and thus it can
disregard large portions of the search tree.

In Rustic, MVV-LVA has been implemented since the first version, Alpha 1.
In version Alpha 2, it still is the only move sorting technique. It is
recommended that you implement this function directly after you get
iterative deepening and alpha-beta working, because it is instrumental to
getting your engine to play decent chess.

The function is implemented as a two-dimensional array:

```rust,ignore
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

The two-dimensional array is addressed in the victim-attacker order,
just as MVV-LVA says. Each combination has a value. For example:

```rust,ignore
let value1 = MVV_LVA[Pieces::BISHOP][Pieces::PAWN];
```

In this case, the victim is a Bishop, while the attacker is a pawn. The
value of this capture as provided by the MVV-LVA array, will therefore be
35. Another capture could be:

```rust,ignore
let value2 = MVV_LVA[PIECES::QUEEN][Pieces::ROOK];
```

Now, the a Queen can be captured by a Rook. The value of this capture would
thus be 52. If the engine has a choice between both captures, the second
one will be ordered higher in the move list. The engine will search it
first, because capturing the queen using a rook is probably better (5
points of material gain), than capturing the bishop using a pawn (2 points
of material gain.)

> **Sidenote** You may be wondering about the values in the table: why did
> I choose 10, 11, 12.... and then 20, 21, 22... for the next row? The
> simple answer is that these values are arbitrary. The only thing that
> counts is to make sure that the values for the better captures are
> higher. Instead of 10, 11, 12..., I could have chosen 137, 180, 231,...
> and then for 20, 21, 22,... the values could have been 400, 473, 474. It
> doesn't matter. Personally, when choosing values like these, I like to
> keep them as small as possible. Then they are easier to understand,
> easier to edit if necessary, and they fit into smaller integers.

This is how the ordering is implemented, when MVV_LVA is the only sort
scoring functionality in the engine:

```rust,ignore
pub fn score_moves(ml: &mut MoveList) {
    for i in 0..ml.len() {
        let m = ml.get_mut_move(i);
        let value = MVV_LVA[m.captured()][m.piece()];
        m.add_score(value);
    }
}
```

I already said that implementing this optimization would be easy. We just
run through the move list, grab the combination of the victim and attacker
from the MVV_LVA-array, and we add it to the move's score. (Because this is
the only place in the engine where moves are scored for sorting, the member
function _add_score()_ is renamed to _set_sort_score()_ in later versions
of Rustic, so you may see _set_sort_score()_ in other move sorting
chapters.)

An example of the effectiveness of this implementation is given below. In
the first run, Rustic Alpha 1 is searching a position without any move
ordering in place. In the second run, MVV-LVA has been enabled as the one
and only move ordering technique. (For brevity's sake, part of the output
string has been truncated, as the extra information is not relevant for
this example.)

The FEN of the position is:

```
r3k2r/p1ppqpb1/bn2pnp1/3PN3/1p2P3/2N2Q1p/PPPBBPPP/R3K2R w KQkq - 0 1
```

Run _without_ MVV-LVA move sorting:

```
info score cp 25 depth 1 seldepth 5 time 878 nodes 5161544
info score cp 25 depth 2 seldepth 5 time 6429 nodes 38369245
info score cp 20 depth 3 seldepth 5 time 121942 nodes 722743000
```

Run _with_ MVV-LVA move sorting:

```
info score cp 25 depth 1 seldepth 3 time 0 nodes 1598
info score cp 25 depth 2 seldepth 3 time 1 nodes 3196
info score cp 20 depth 3 seldepth 5 time 2 nodes 7315
info score cp 20 depth 4 seldepth 5 time 6 nodes 20260
info score cp 5 depth 5 seldepth 9 time 25 nodes 76603
info score cp 20 depth 6 seldepth 7 time 109 nodes 293985
info score cp 5 depth 7 seldepth 9 time 543 nodes 1333835
info score cp 5 depth 8 seldepth 9 time 2900 nodes 7288058
info score cp -40 depth 9 seldepth 11 time 16933 nodes 39339223
```

The first run without MVV-LVA took over 2 minutes to reach depth 3 in the
given position. The second run, with MVV-LVA enabled, took only 16.9
seconds to reach depth 9. The cause of this is the number of nodes that is
being saved in the search. In the first run, the engine searched
722.743.000 (722.7 million) moves to reach depth 3. In the second run, the
engine only searched 7.315 nodes to reach the same depth. Even at depth 9,
the number of nodes is 'just' 39.339.223 (39.3 million), which is a
fraction of the 722 million from the first run.

Granted, MVV-LVA is one of the techniques that brings the greatest
improvements. That is the reason why it is recommended to implement it
first. On top of this technique, further move ordering enhancements can be
implemented which will be discussed in the next chapters.


> **Sidenote** So, what about the 0's in the table? The number of elements
> for the rows and columns in the table is "NrOf::PIECE_TYPES + 1". There
> are 6 piece types in chess: King, Queen, Rook, Bishop, Knight, Pawn.
> Rustic also has a piece type called Pieces::NONE, which is used when
> there's no piece on a square, or, in the move integer, there is no piece
> in the Promotion or Captured fields. That is where the +1 comes from.
> 
> In MVV_LVA move ordering, Piece::NONE cannot be the victim and it also
> cannot be the attacker. You can't capture TO an empty square. You also
> can't capture FROM an empty square. Therefore all the cells that have
> Piece::NONE as part of the combination are 0. (A move FROM an empty
> square isn't even possible... supposing your engine has no bugs in the
> move generator.) The top row is also 0, because the King cannot be
> captured.
> 
> Setting up the MVV_LVA array this way makes the scoring function somewhat
> simpler because you don't have to check if the FROM and TO square contain
> pieces and that you're not capturing a king.

