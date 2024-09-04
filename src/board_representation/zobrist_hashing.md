
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Zobrist Hashing](#zobrist-hashing)
  - [Explanation](#explanation)
  - [Extending the method to chess](#extending-the-method-to-chess)

<!-- /code_chunk_output -->

# Zobrist Hashing

## Explanation

Zobristh hashing is a method of representing unique positions in a board
game. It was developed by Albert Lindsey Zobrist (1942) in 1970 and it is
this that Dr. Zobrist is most fanmous for. When first studying this method
it can be a bit confusing, but after you get your head around it, you will
surely appreciate its elegance.

To make it simpler to explain, let us first boil it down to its most simple
version. To do so, we'll define a rather simple board game:

- There is 1 player.
- The board has 4 squares. 
- There are 4 pieces, all of the same type.
- The player is allowed to take back moves and remove a piece.
- As soon as the player puts the fourth piece onto the board, he wins.

This game is obviously trivial. The player can win any game in 4 moves, or
he can get stuck in a loop by adding and removing pieces one after the
other. Still, it is enough to explain how the Zobrist method works.

We now want to define the Zobrist hashing method:
- We must have a number that is unique for the board position it was
  calculated for.
- When the board position changes, we must be able to update this number.
  We do not want to recalculate the number from scratch on every change,
  because this would be too slow.

In this game, achieving this is simple. We have a board with 4 squares, and
all pieces are of the same type. So, we can simply do this:

- 0000 (0), empty board.
- 0001 (1), piece on square 1
- 0010 (2), piece on square 2
- 0100 (4), piece on square 3
- 1000 (8), piece on square 4

There are only 5 numbers, because there are 4 squares x 1 piece type, and
the empty board. We have to define a number for each piece/square
combination. (Later, when extending this to chess, you'll see we also have
to define keys for special situations such as castling.)

We start with the empty board, zo the Zobrist Key (this is what the number
is most often called) for the board is 0000. The player adds a piece on
square 4. The key for a piece on this square is 1000. We now want to change
the board's Zobrist key, without looping through the entire board and
recalculating it from scratch. The way to do this, is with the XOR operand:

```
0000    (current board key)
1000    (piece/square key, added or removed)
----    xor
1000    (new key)
```

So the board's key is 1000. The player now adds a piece to square 3:

```
1000    (current board key)
0100    (piece/square key, added or removed)
----    xor
1100    (new key)
```

So the board's key is 1100. The player now adds a piece to square 1:

```
1100    (current board key)
0001    (piece added or removed)
----    xor
1101    (new key)
```

The board key is now 1101. The player takes back the move on square 3:

```
1101    (current board key)
0100    (piece added or removed)
----    xor
1001    (new key)
```

The key becomes 1001. The player adds the piece to square 3 again:

```
1001    (current board key)
0100    (piece added or removed)
----    xor
1101    (new key)
```

As you can see, the key returns to 1101 again. This is the crux of the
matter: By taking the current Zobrist board key and the XOR-ing it with the
key of the change you are making, you can either add this change to the
board key, or take it out again. By XOR-ing the same key repeatedly as we have
done in the last two steps, the Zobrist key flips back and forth between
two values.

## Extending the method to chess

Now let's extend this method to chess.