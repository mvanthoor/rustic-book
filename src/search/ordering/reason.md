# The reason

Why would you want to order the moves the engine is going to search? The
answer to that may not be immediately obvious. However, if you take a
moment to take into account how a human thinks about a position, everything
will become clear. Let's review an example that could occur in an actual
chess game. Consider the following position, with white to move:

<img src="../../positions/position_1.png" />

There are a lot of possible moves to be played. The rook and the queen have
22 moves together, and we're not even counting the king, the knight, and
the pawns. You may ask yourself: Which moves should I consider _first_?
That is a perfectly valid question, because moves are not of equal value.
For example:

- a2-a3? Nah. That move does nothing.
- Nf5xg7, winning a pawn? No. The black king will just recapture.
- Maybe Nf5-e7+, attacking the king and the rook at the same time?

The last move seems to be great: it attacks the black king AND the rook,
and after the king moves, the rook can be captured. Great. But wait...
there's bigger fish to fry.

__Rc2xc8!__

At first glance, this move only seems to trade rooks after _Qb8xc8_, but
you have to realize that it's now the black _queen_ on c8, and the move
Nf5-e7+ is still on the board: the knight now attacks the black king and
the _queen_, instead of the rook, for an even greater gain.

So, what if black doesn't take the white rook, moving the queen off the
back rank with Qb8-a7? White could still play Nf7-e7+, but now that would
be a mistake. The better move is:

__Rc8xe8#!__

Black is checkmated. The black queen is unable to move off the back rank
and keep the knight defended at the same time. (Note that _Qb8-e5_ does not
work, because white will just capture with _d4xe5_.) Qb8xh2+ is possible
because it checks the white king, but then white will just recapture, so
that's no good either. Worse, the Rc8xe8 checkmate threat is still on the
board.

Thus after Rc2xc8, the response _Qb8xc8_ is the only good move, even though
white will follow it up with _Nf5-e7+_, which wins the black queen and the
game. (You could therefore say that _Qb8xc8_ is the least bad move instead
of the best move.)

The takeaway from this example is that most moves on the board are not even
considered by humans. If you're in the middle of an attack such as in the
position discussed above, it's unlikely you're going to consider moves such
as _a2-3_. The order in which moves are mostly evaluated is the following:

- Captures are evaluated before quiet moves.
- Good captures are evaluated before bad captures.
- Checks come before completely quiet moves.
- Quiet moves will only be evaluated if there are no immediate threats to
  to make against the opponent's king, or no threats against the king or
  one's one pieces needs to be addressed.

Using techniques like these, humans narrow down the list of moves to
examine. A chess engine has to do the same thing, or it will be examining a
lot of useless moves. This slows it down in finding good moves and it will
be weaker because of that. If the engine has a way of examining good moves
first, it can skip searching huge chunks of the move tree, which will be
saving a lot of time in finding the promising variations. It can use this
saved time to search deeper. The engine will become stronger as a result.

In this chapter, we'll discuss various techniques on how to order moves
during the search. To make the gains of move ordering more tangible, an
example of a search _with_ and _without_ move ordering will be provided in
the MVV-LVA chapter.