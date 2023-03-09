# Bitboards

So, what is a bitboard? In the chess engine world, this is a 64-bit integer
which is a representation of something on the board. "Something" can be a
piece on a square, if a square is occupied or not, if a piece can move to
that square, and many more things.

In Rustic, the bitboards least significant bit (LSB), is bit 0, and in the
case of squares, it represents square A1. When printing a bitboard to the
screen however, the square A1 is on the bottom left, so effectively, the
bitboard s printed backwards in rows of 8, and upwards. Let's take a look
at some concrete examples.

Let's put a knight on e4:

```
bitboard, LSB on the far right:

00000000 00000000 00000000 00000000 00010000 00000000 00000000 00000000
                                       ^e4 ^a4      ^a3      ^a2      ^a1

printed backwards and upwards:
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 0 <- h3
0 0 0 0 0 0 0 0 
0 0 0 0 0 0 0 0 <- h1
^a1
```

This bitboard would have the value "268_435_456" internally, but we are
(almost) never going to use these values directly. If you ever need to see
what a bitboard holds, you will need to print it to the screen, because the
debugger will just tell you the value. For example, what does this mean?
