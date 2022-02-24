# Detecting cannot force mate

In  the section [Detecting draws by FIDE rules](./detecting_fide_draws.md)
the "Draw by insufficient material rule." is explained and implemented.
This rule makes it possible to claim a draw due to having insufficient
material to win the game. However, that rule states that a draw can only be
claimed if there is no way to achieve checkmate by any order of legal
moves.

There are some material combinations such as King+Knight vs. King+Knight
where a mate *can* be achieved by legal moves, even though one of the
players has to assist in getting mated. In case of these material
combinations a draw cannot be claimed under the FIDE rules. When
implementing XBoard, the engine has to know about this.

The function to detect this situation is
*draw_by_insufficient_material_rule()*.

When searching however, the situation is a bit different. Even though the
game cannot be *claimed* as draw in the King+Knight vs. King+Knight
situation, it is, in practice, still a draw because checkmate cannot be
forced. We need a different function to detect such conditions, which is
then used during the engine's search routine. It can be used to score a
position where mate cannot be forced as a draw. The engine will actively
try to achieve such a position when it is losing or try to avoid it when it
is winning.

This function is *sufficient_material_to_force_checkmate()*.

Checkmate can still be forced if one of the following conditions is true:

- There is still a queen, rook, or pawn on the board. (The pawn can
  potentially promote to a rook or a queen in the future, which can then
  be used to force a checkmate.)
- At least one player has the bishop pair. This means that one player has
  to have two bishops, and they must be on squares of different color.
  ([See: Detecting the bishop pair](./detecting_bishop_pair.md)).
- At least one player has both a bishop and a knight.
- At least one player has at least three knights, which can be achieved by
  promoting a pawn. Granted, this is a super-rare situation, but with three
  knights against a lone king, checkmate can be forced.

The following function checks the above conditions. If any of them
applies it returns *true* to indicate that checkmate can still be forced.
It can be found in the file *draw.rs*:

```csharp
pub fn sufficient_material_to_force_checkmate(&self) -> bool {
    let w = self.get_bitboards(Sides::WHITE);
    let b = self.get_bitboards(Sides::BLACK);

    w[Pieces::QUEEN] > 0
    || b[Pieces::QUEEN] > 0
    || w[Pieces::ROOK] > 0
    || b[Pieces::ROOK] > 0
    || w[Pieces::PAWN] > 0
    || b[Pieces::PAWN] > 0
    || self.has_bishop_pair(Sides::WHITE)
    || self.has_bishop_pair(Sides::BLACK)
    || (w[Pieces::BISHOP] > 0 && w[Pieces::KNIGHT] > 0)
    || (b[Pieces::BISHOP] > 0 && b[Pieces::KNIGHT] > 0)
    || w[Pieces::KNIGHT].count_ones() >= 3
    || b[Pieces::KNIGHT].count_ones() >= 3
}
```
During the search the engine will call this function after every move. As
soon as it returns *false*, the engine scores the evaluation of the
position as a draw and it will be useless to search any deeper.
