# Piece List

The previous chapter described how the bitboard board representation is
implemented in Rustic and also shows some of the nice things that can be
done with them, such as quickly determining which squares are occupied by
which pieces, or which squares on the board are empty.

However, there is one big problem with having a set of six bitboards to
represent a board. It is like having 12 chess boards in front of you while
playing a game.

- One board that only has the white king.
- One board that only has the black king.
- One board containing only two white rooks...
- And one board containing only two black rooks...
- And so on.

Very often this is not a problem for the engine because it is working with
one piece at a time and the occupancy of the board. For example, to answer
this question: Can a White Knight on c3 go to e4? The engine would do the
following to determine this:

- From the move generator, get all the squares the White Knight from c3
  could go to on an empty board, and remove the ones that contain pieces of
  White itself. (Black pieces don't have to be removed because they can be
  captured.)
- Make sure the Knight is not pinned, so the king is not in check when it
  moves.

What is left is all the legal moves of the Knight on c3, and if e4 is in
the resulting bitboard of squares where the Knight is allowed to move, then
the answer is "YES": the Knight can go do e4.

But what if you want to know if the Knight captured something on e4, and if
so, which piece was it? It is essential to know this information, because
when taking back the move (which the engine will do during the search), the
Knight has to move from e4 back to c3, but if there was a piece on e4, it
has to be restored also.

We can't get this information from the occupancy of the board, because that
only tells us if there are pieces on a square, but not which piece it is.
Because of this, the question "Which piece is on e4?" means we will have to
loop through the 6 Black bitboards (one for each piece type) and check if
the bit for e4 is set. This way we can determine what piece type is on e4,
if any. This process is very slow.

Enter the piece list to resolve this issue. The piece list is just an array
of 64 elements, one for each square. If a square is empty, it holds the
value 0; if a piece is on a square, it holds the number for that piece
type. The array looks like this:

```rust,ignore
piece_list: [Piece; NrOf::SQUARES]
```

When setting up the board, we first fill the 6 bitboards for each side.
Then we loop through the bitboards for both White and Black, and put each
piece we encounter into the correct square in the piece list. This happens
when we start the engine, or when we initialize it with a new position. The
piece list initialization looks like this:


```rust,ignore
  fn init_piece_list(&self) -> [Piece; NrOf::SQUARES] {
      let bb_w = self.bb_pieces[Sides::WHITE]; // White piece bitboards
      let bb_b = self.bb_pieces[Sides::BLACK]; // Black piece bitboards
      let mut piece_list: [Piece; NrOf::SQUARES] = [Pieces::NONE; NrOf::SQUARES];

      // piece_type is enumerated, from 0 to 6.
      // 0 = KING, 1 = QUEEN, and so on, as defined in board::defs.
      for (piece_type, (w, b)) in bb_w.iter().zip(bb_b.iter()).enumerate() {
          let mut white_pieces = *w; // White pieces of type "piece_type"
          let mut black_pieces = *b; // Black pieces of type "piece_type"

          // Put white pieces into the piece list.
          while white_pieces > 0 {
              let square = bits::next(&mut white_pieces);
              piece_list[square] = piece_type;
          }

          // Put black pieces into the piece list.
          while black_pieces > 0 {
              let square = bits::next(&mut black_pieces);
              piece_list[square] = piece_type;
          }
      }

      piece_list
  }
```

That it. The piece list is initialized. This is done when a new position is
set up, so the engine now knows which piece type is on which square. Note
that it does not have to know if the piece belongs to Black or White.
Because a captured piece is always the opposite of the color of the piece
that does the capturing (so we already know the captured color), we only
have to store the captured piece type in the piece list.

So, when the Knight captures a piece on e4, we can know what it was by
using this one-liner:

```rust,ignore
let captured_piece = board.piece_type[SQUARES::E4];
```

If the captured_piece is of type PIECES::None, we captured nothing.
Obviously captured_piece should never be PIECES::King, because in that case
the opponent would have made a move that does not get the King out of
check. If this happens, the engine plays illegal chess and a testing
program such as CuteChess will count the game as lost.

Now we've collected all the parts for the board representation:

- Bitboards; 12 (6 per color)
- Bitboards; 2 (ooccupancy per color)
- Game State
- Game History
- Zobrist Keys
- Piece List

Now we can go and combine al of this into a Board Struct, which will
complete the board representation. Then, we'll have to write some
functions to get the board to do something. On to the next and final
chapter of this section: [Board Struct](./board_struct.md)


