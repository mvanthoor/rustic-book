# Introduction

Now that the Board Representation is done, we have something with which we
can start to play chess, assuming you implemented everything without too
many bugs. This chapter, Board Functionality, describes functions that
perform actions on the current board, or can provide information about the
position. Think about:

- Set up the position from a FEN-string.
- Create a FEN-string from a position.
- Move a piece from one square to the other.
- Detect if a side has the bishop pair or not.
- Detect if a side can still force mate
- Provide the square our king is on
- ... and many others.

While reading this chapter you may draw the conclusion that some of these
functions could, or should be part of the engine itself instead of the
board. Some of these functions could be in the engine, but my philosophy
with regard to function location is that it should be as close to the
object it pertains to, without taking outside information into account.

This might actually not clarify things too well, if you're not yet
well-versed in the concept of "separation of responsibility", so here's are
some examples of what I mean.

Because I can can determine if a side has a bishop pair or not by just
looking at the board, without even using the engine, the function should
belong to the board.

That example should be clear enough. The one about moving pieces is a bit
more difficult, because... isn't it the _engine_ who moves pieces? Let's
see.

I can move a piece from one square to the other by just using the board. I
don't need the engine for this. Pick up the piece, put it down somewhere
else. Thus, the function should be in the board. "But, you could make an
illegal move", I hear you thinking. Yes, indeed. But this is the part of
"without taking outside information into account."

It would be possible to move a knight from c3 to c5, and if you do so using
the board's move_piece() function, it would work correctly; the knight will
disappear from c3 and land on c5, even if this captures a piece of the same
color, even if it leaves the king in check, or both. If you order the board
to remove a pawn from b7 and put a bishop on f8, it should do it.

Why is that?

If the move is legal or not in that position, is not the responsibility of
the board to determine. It is something the engine needs to decide, before
the move even happens. This is critical if you want to write an engine that
can play different variants of chess. Such an engine can have different
move generators and different rule sets, even though each variant uses the
same board and the same pieces.

>**Sidenote**: You will find the make() and unmake() functions in the
>playmove.rs file, which belongs to the board. I can imagine that this is
>confusing because I just said that the board is not responsible for moving
>pieces. Technically, make() and unmake() don´t move pieces: they enforce
>some of the rules of chess such as the king can not be left in check after
>a move. The board has different functions that _actually_ move the pieces,
>such as put_piece(), remove_piece() and move_piece(). A board could have
>multiple make() and unmake() functions, one set for each different chess
>variant. Each would use the underlying put_piece(), remove_piece() and
>move_piece() functions.
>
>The reason why make() and unmake() are in the board and not in the engine
>is because then the entire Engine struct would need to be available in the
>Search function so it can make and unmake moves. Because Search is a part
>of the Engine struct itself and running in a separate thread, this would
>make things either very messy, or it would maybe even be impossible.

Now that we have determined how the board should be working, we can dive
into it and look at the easiest parts: [Support functions.](./support_functions.md)