
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Zobrist Hashing](#zobrist-hashing)
  - [What is this?](#what-is-this)
  - [Why do we need this?](#why-do-we-need-this)
  - [Eplanation](#eplanation)
  - [Extending the method to chess](#extending-the-method-to-chess)
  - [Implementation](#implementation)
  - [Initialization](#initialization)
  - [Access to the keys](#access-to-the-keys)

<!-- /code_chunk_output -->

# Zobrist Hashing

## What is this?

Zobrist hashing is a method of representing unique positions in a board
game using a single large number. It was developed by Albert Lindsey
Zobrist (1942) in 1970 and it is this that Dr. Zobrist is most fanmous for.
When first studying this method it can be a bit confusing, but after you
get your head around it, you will surely appreciate its elegance.

## Why do we need this?

In a chess engine there are numerous reasons to quickly identify a
position. One would be to detect repetitions: if a position occurs on the
board the third time, with the same player to move, the game is a draw by
rule. Also, when implementing a transposition table which holds positions
that have been encountered and evaluated in the past (the transposition
table can be seen as the engine's memory), the Zobrist Keys are a way to
quickly find positions the engine has encountered before while searching
and then get their evaluation from the table instead of recalculating it
again. This saves an enormous amount of calculation time; so much so, that
adding a transposition table to an engine will typically gain 130-150 Elo
points in playing strength.

## Eplanation

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

Now let's extend this method to chess. The previous simple game had only 15
different positions (binary 0000 to 1111), so we could effectively give
each square/piece combination its own number/key by hand and combinging
those into a board key. With chess this is not possible, because for all
intents and purposes, the number of positions is unlimited. Compared to the
previous game, chess has:

- Two players instead of one, of which one has the turn and the other doesn't
- 64 squares instead of 4
- 6 piece types instead of one, which can be all on the board, or not
- A bunch of special cases such as castling and en-passant

All of this needs to be encoded in the board Zobrist key, so we first need
to know how many keys we need.

- We need 64 squares x 6 types x 2 players = 768 square/piece/player keys
- We need 16 keys for the castling positions. Castling is coded the same
  way as the game above: 0000 is no castling rights, 0001 is white can
  castle kingside, 0010 is white can castle queenside, and so on. This
  gives a binary number from 0000 to 1111, which are 16 values.
- We need 2 keys for the side to move: white/black.
- 17 en-passant keys (8 squares for white, 8 squares for black, and 1 for
  no en-passant square).

The total number of keys we have is 768 square/piece/player keys + 16
castling keys + 2 side-to-move keys + 17 en-passant keys = 803 keys.

> **Sidenote 1**: Rustic, at least up until version 3.0.0, used 65 en-passant
> keys; one for each square, even though only 16 are valid en-passant
> squares, and one for empty/no en-passant square. This makes no difference
> with regard to the Zobrist method or the performance. It just means that
> the engine generates 48 Zobrist keys it will never use. This may have
> been changed in later versions of Rustic to use only 17 en-passant keys.

> **Sidenode 2**: It is also possible to use only 5 castling keys: one each
> for white kingside, white queenside, black kingside, black queenside and
> no castling permissions. In that case however, if a player moves his
> king, you'd need to do two Zobrist transformations instead of one. I
> found it easier to just create 16 castling keys and do all the castling
> changes with one key change. This is also faster.
>
> It is even possible to cut down more on the keys in use. For example, you
> can have only 1 side-to-move key. If you include it in the board key,
> it's white to move; if it's not in there, it's black to move. Same for
> the castling and en-passant keys. You don't NEED to have the "empty" keys
> for no castling and no en-passant; you can just opt to not include any of
> the keys castling or en-passant keys to mark that these possibilities
> don't exist in the position. Personally I find this confusing, so I use a
> key for each possible state: 16 to capture all possible combinations of
> castling permissions including empty, and 16+1 for en-passant. It is more
> consistent, because you _always_ have a key included.

## Implementation

We need 803 keys to represent every possible position that could occur in a
chess game. Because there are so many, we want the number to be as big as
possible; i.e., we are not just going to use the numbers from 1 to 803,
because these keys wouldn't contain enough bits. When combining such small
keys, there would be many different positions that resolve to the same key,
which is something we definitely don't want. The solution to this is to
create 64-bit keys, which we will then fill by randomly generating a 64
integer for each.

Implementing this is easy: you could just use an array of 803 elements,
loop through it and generate a key for each. After that you index into the
array. The first 768 keys are for square/piece/player combinations, the
next 16 are for castling, and so on.

Personally I opted to split this up to make it easier to reason about, so I
defined a ZobristRandoms struct.

```rust,ignore
type PieceRandoms = [[[u64; NrOf::SQUARES]; NrOf::PIECE_TYPES]; Sides::BOTH];
type CastlingRandoms = [u64; NrOf::CASTLING_PERMISSIONS];
type SideRandoms = [u64; Sides::BOTH];
type EpRandoms = [u64; NrOf::SQUARES + 1];

pub struct ZobristRandoms {
    rnd_pieces: PieceRandoms,
    rnd_castling: CastlingRandoms,
    rnd_sides: SideRandoms,
    rnd_en_passant: EpRandoms,
}
```
>**Sidenote** This version still uses the 65 en-passant square keys, but
>for the overall implementation this makes no difference.

Then we fill up each of the arrays with random numbers. For example, for the
square/piece/side part, Rustic uses this code:

```rust,ignore
zobrist_randoms.rnd_pieces.iter_mut().for_each(|side| {
    side.iter_mut().for_each(|piece| {
        piece
            .iter_mut()
            .for_each(|square| *square = random.gen::<u64>())
    })
});
```

It is a short piece of functional code and it literally says what it is
doing: for each side, for each piece, on each square, generate a random
number. The code for the other keys (castling, side to move, and
en-passant) is very similar.

>**Sidenote** Obviously it would be possible to generate the Zobrist
>randoms once and then just copy/paste them into the code. You could even
>use an online number generator to create them, because their values don't
>matter, as long as they use the full 64 bits. I decided to use the rnd
>crate with the ChaCha algorithm, because it is guaranteed (at the time of
>writing this) to generate the same list of random numbers on any platform
>Rust runs on, as long as you seed the number generator with the same seed.
>However, even if it wouldn't generate the same numbers each time, it would
>still work. That could make debugging harder though, if the Zobrist Keys
>don't work as intended. It is therefore best that the Zobrist Keys are the
>same on every run of the engine, so it would be advisable to either use
>static keys, or a seeded random number generator.

## Initialization

We first have to initialize the Zobrist Key for the position we are
starting out with. I can hear you think: if we would use static numbers
instead of generating them, couldn't we just calculate the key once and
just assign it? It would work, but only for the standard starting position.
It would be impossible to support Fischer Random Chess (FRC, Chess960) or
start a game from a different position such as when giving pawn odds.
Therefore we just write a function to set up the initial Zobrist Key.

```rust,ignore
  pub fn init_zobrist_key(&self) -> ZobristKey {
      let mut key: u64 = 0;

      // "bb_w" is shorthand for "self.bb_pieces[Sides::WHITE]".
      let bb_w = self.bb_pieces[Sides::WHITE];
      let bb_b = self.bb_pieces[Sides::BLACK];

      // Iterate through all piece types, for both white and black.
      // "piece_type" is enumerated, and it'll start at 0 (KING), then 1
      // (QUEEN), and so on.
      for (piece_type, (w, b)) in bb_w.iter().zip(bb_b.iter()).enumerate() {
          // Assume the first iteration; piece_type will be 0 (KING). The
          // following two statements will thus get all the pieces of
          // type "KING" for white and black. (This will obviously only
          // be one king, but with rooks, there will be two in the
          // starting position.)
          let mut white_pieces = *w;
          let mut black_pieces = *b;

          // Iterate through all the piece locations of the current piece
          // type. Get the square the piece is on, and then hash that
          // square/piece combination into the zobrist key.
          while white_pieces > 0 {
              let square = bits::next(&mut white_pieces);
              key ^= self.zobrist_randoms.piece(Sides::WHITE, piece_type, square);
          }

          // Same for black.
          while black_pieces > 0 {
              let square = bits::next(&mut black_pieces);
              key ^= self.zobrist_randoms.piece(Sides::BLACK, piece_type, square);
          }
      }

      // Hash the castling, side to move, and en-passant state.
      key ^= self.zobrist_randoms.castling(self.game_state.castling);
      key ^= self
          .zobrist_randoms
          .side(self.game_state.active_color as usize);
      key ^= self.zobrist_randoms.en_passant(self.game_state.en_passant);

      key
  }
```

We need a place to put the board's current Zobrist Key. The GameState
struct, which collects all manner of data about the current position is an
ideal location for this, so we'll discuss this in the next chapter.

## Access to the keys

Because the random numbers are hidden inside this struct, we also need some
functions to get to them:

```rust,ignore
  pub fn piece(&self, side: Side, piece: Piece, square: Square) -> ZobristKey {
      self.rnd_pieces[side][piece][square]
  }

  pub fn castling(&self, castling_permissions: u8) -> ZobristKey {
      self.rnd_castling[castling_permissions as usize]
  }

  pub fn side(&self, side: Side) -> u64 {
      self.rnd_sides[side]
  }

  pub fn en_passant(&self, en_passant: Option<u8>) -> ZobristKey {
      match en_passant {
          Some(ep) => self.rnd_en_passant[ep as usize],
          None => self.rnd_en_passant[NrOf::SQUARES],
      }
  }
```

That's it for the Zobrist Keys. The next chapter deals with the GameState struct.