# Detecting draws by FIDE rules

To be able to play chess properly, the engine has to be able to detect
draws. In addition, to fully support the XBoard protocol [(see the chapter
about communication)](../communication/communication.html), it has to be
able to claim those draws. This chapter describes how to use the board
representation to detect these draws.

## Draw by insufficient material rule

There is a rule in chess which is often misunderstood: draw by insufficient
material. It seems simple: if none of the players have enough material on
the board to checkmate the king, a draw can be claimed. However, this is
not exactly what the "FIDE Laws of Chess" handbook
([download](https://www.fide.com/FIDE/handbook/LawsOfChess.pdf)) says. The
relevant rule is as follows:

> The game is drawn when a position has arisen in which neither player can
> checkmate the opponent’s king with any series of legal moves.

This is slightly different from the commonly held belief of "not having
enough material to checkmate the enemy king. If you have a King+Bishop vs.
King, then the position is a draw, because no player can checkmate the
other using any series of legal moves. What about King+Bishop vs.
King+Bishop, or King+Knight vs. King+Knight? These positions should be
drawn, because no player can achieve a mate, right? Wrong...

K+B vs. K+B:<br />
<img src="../../positions/kbkb_mate.png" />

K+N vs. K+N:<br />
<img src="../../positions/knkn_mate.png" />

*Foul play!* I hear you cry. *Black must have assisted white in achieving
these mates, because they cannot be forced!* True enough: these mates
cannot be forced. Black must assist white by blocking his own king with his
last move, so white can deliver mate. However, these positions are
achievable through a series of legal moves. This means that a player can
not _claim_ a draw on the basis of insufficient material. Of course, both
players can agree on a draw because they both know that the other will not
assist in getting mated. Even if one of the players refuses to agree on a
draw, the game will eventually run into the 50-move rule and be drawn
anyway.

Because of the above, the function which implements this rule
(*draw_by_insufficient_material_rule()*) is different from the function
used during searching (*sufficient_material_to_force_checkmate()*). This
function determines if it is legal, according to the rules of chess, to
claim a draw and end the game. The function used during searching
determines if one of the players is still able to force a checkmate, and if
not, to score the position as a draw.

> Note that no mate can ever be achieved in a King+Bishop vs. King+Bishop
> position, if both bishops are on squares of the same color. Any position
> with that material is therefore claimable as a draw.

The function implementing this rule is fairly simple. It just lists all the
positions in which a draw by insufficient material can be claimed:

```csharp
pub fn draw_by_insufficient_material_rule(&self) -> bool {
    // Get the piece bitboards for white and black.
    let w = self.get_bitboards(Sides::WHITE);
    let b = self.get_bitboards(Sides::BLACK);

    // Determine if at least one side has either a Queen, a Rook or a
    // pawn (qrp). If is the case, a draw by rule is not possible.
    let qrp = w[Pieces::QUEEN] != 0
        || w[Pieces::ROOK] != 0
        || w[Pieces::PAWN] != 0
        || b[Pieces::QUEEN] != 0
        || b[Pieces::ROOK] != 0
        || b[Pieces::PAWN] != 0;

    // No queens, rooks or pawns. We may have a draw.
    if !qrp {
        // King vs. King
        let kk = w[Pieces::BISHOP] == 0
            && w[Pieces::KNIGHT] == 0
            && b[Pieces::BISHOP] == 0
            && b[Pieces::KNIGHT] == 0;
        // King/Bishop vs. King
        let kbk = w[Pieces::BISHOP].count_ones() == 1
            && w[Pieces::KNIGHT] == 0
            && b[Pieces::BISHOP] == 0
            && b[Pieces::KNIGHT] == 0;
        // King/Knight vs. King
        let knk = w[Pieces::BISHOP] == 0
            && w[Pieces::KNIGHT].count_ones() == 1
            && b[Pieces::BISHOP] == 0
            && b[Pieces::KNIGHT] == 0;
        // King vs. King/Bishop
        let kkb = w[Pieces::BISHOP] == 0
            && w[Pieces::KNIGHT] == 0
            && b[Pieces::BISHOP].count_ones() == 1
            && b[Pieces::KNIGHT] == 0;
        // King vs. King/Knight
        let kkn = w[Pieces::BISHOP] == 0
            && w[Pieces::KNIGHT] == 0
            && b[Pieces::BISHOP] == 0
            && b[Pieces::KNIGHT].count_ones() == 1;
        // King/Bishop vs. King/Bishop
        let kbkb = w[Pieces::BISHOP].count_ones() == 1
            && w[Pieces::KNIGHT] == 0
            && b[Pieces::BISHOP].count_ones() == 1
            && b[Pieces::KNIGHT] == 0;

        // Both bishops have to be on the same colored square for a
        // draw to be claimable.
        let same_color_sq = if kbkb {
            let wb_sq = w[Pieces::BISHOP].trailing_zeros() as usize;
            let bb_sq = b[Pieces::BISHOP].trailing_zeros() as usize;

            Board::is_white_square(wb_sq) == Board::is_white_square(bb_sq)
        } else {
            false
        };

        if kk || kbk || knk || kkb || kkn || (kbkb && same_color_sq) {
            return true;
        }
    }

    // All other cases cannot be claimed as a draw.
    false
}
```