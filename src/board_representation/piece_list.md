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
this question: Can a White knight on c3 go to e4? The engine would do the
following to determine this:

- From the move generator, get all the squares the White knight from c3
  could go to on an empty board, and remove the ones that contain pieces of
  White itself. (Black pieces don't have to be removed because they can be
  captured.)
- Make sure the Knight is not pinned.

What is left is all the legal moves of the Knight on c3, and if e4 is in
the list, then the answer is "YES": the Knight can go do e4.

(This page is not finished yet...)