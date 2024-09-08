# Bitboards

>This page seems to deal with simple material at first, but it is probably
>the most important page in the book. It explains how bitboards work and
>how they are used to represent things within the chess engine. It is
>essential to understand this page, along with the topics [The binary
>system](../appendix/binary_system.md) and [Bitwise
>Operations](../appendix/bitwise_operations.md) from the appendices. This
>is the third time this is mentioned: if you are not yet _completely_
>confident that you understand these two topics, go back now and study
>them. If you continue with gaps in this knowledge, making progress with
>your engine will be hard. Debugging will be even harder.

So, what is a bitboard? In the chess engine world, this is a 64-bit integer
which is a representation of something on the board. "Something" can be a
piece on a square, if a square is occupied or not, if a piece can move to
that square, and many more things.

In Rustic, the bitboard's least significant bit (LSB) is bit 0, and in the
case of squares, it represents square A1. Therefore, if we have a bitboard
that denotes that square A1 is occupied, it looks like this:

```
    00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001
```

This is not very practical to work with while developing the  engine. A
bitboard denoting a piece on e4 will have an internal value of 268_435_456.
That is even harder to understand in a debugger. It is much easier to debug
when you can print the bitboard to the screen, with A1 being on the lower
left, just as with a normal chess board. Therefore, one of the first
functions you should write is one to print bitboards to the screen in the
layout of a chess board. This will be very useful in the initial stages of
the engine's development. After your board representation and move
generator are done, you can remove this function if you so wish.

If we'd just print the bitboard from left to right, starting at the most
significant bit, and starting a new row every 8 bits, square A1 would end
up on the bottom right. This is because the least significant bit is the
final bit to be printed. That's obviously not correct, because A1 on a
chess board is on the bottom left.

Let's put a piece on e4:

```
bitboard, LSB on the far right:

00000000 00000000 00000000 00000000 00010000 00000000 00000000 00000000
                                       ^e4 ^a4      ^a3      ^a2      ^a1

The integer value of this bitboard is: 268_435_456. This is not helpful.
When we print this bitboard, we want to see this:

0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 
0 0 0 0 0 0 0 0
```

So how do we achieve this? First lets analyze which squares are which bits:

```
v-a8 (bit 56)
0 0 0 0 0 0 0 0 <- h8 (bit 63)
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 0 <- h3 (bit 23)
0 0 0 0 0 0 0 0 
0 0 0 0 0 0 0 0 <- h1 (bit 7)
^-a1 (bit 0)
```

As you can see, the bitboard is printed from bit 56-63, followed by a
newline, then bits 48-55 are printed, followed by a newline, and so on,
until we end up with bit 0-7 on the last row. We need a function that
prints rows of 8 bits, starting at bit 56, then starting at 48, then
starting at 40, until we reach the last 8 bits. That function looks like
this:

```rust,ignore
fn print_bitboard(bitboard: u64) {
    const LAST_BIT: u64 = 63;
    for rank in 0..8 {
        for file in (0..8).rev() {
            let mask = 1u64 << (LAST_BIT - (rank * 8) - file);
            let char = if bitboard & mask != 0 { '1' } else { '0' };
            print!("{char} ");
        }
        println!();
    }
}
```

My suggestion would be to do a few iterations by hand. Note that the outer
loop for ranks runs forward from 0 up to and including 7, while the inner
loop for files runs backwards, from 7 down to and including 0. The _mask_
variable will therefore be calculated as such, while printing the top row
of bits:

```
mask = 63 - (0 * 8) - 7 = 56
mask = 63 - (0 * 8) - 6 = 57
mask = 63 - (0 * 8) - 5 = 58
... and so on...
```

And for the second row:

```
mask = 63 - (1 * 8) - 7 = 48
mask = 63 - (1 * 8) - 6 = 49
mask = 63 - (1 * 8) - 5 = 50
... and so on...
```

As I said, it would be best to do a few iterations of these loops by hand,
and also take a close look at how the mask is being created. First we
calculate the target bit and then shift a 1 into that spot in the mask, so
we can use it to determine if the incoming bitboard has a 1 set at that
particular spot.

Now that we know how bitboards work, we can take a look at how they are
used to represent an entire chess board. _It is here that the actual
implementation of the chess engine starts_.

Rustic uses two array with 6 bitboards each. One array is for the white
pieces, the other is for the black ones. There are 6 bitboards per side,
because there are 6 piece types: King, Queen, Rook, Bischop, Knight and
pawn. These 6 piece types are represented in this struct:

```rust,ignore
pub struct Pieces;
impl Pieces {
    pub const KING: Piece = 0;
    pub const QUEEN: Piece = 1;
    pub const ROOK: Piece = 2;
    pub const BISHOP: Piece = 3;
    pub const KNIGHT: Piece = 4;
    pub const PAWN: Piece = 5;
    pub const NONE: Piece = 6;
}
```

This way we don't have to remember that the King has piece value 0, the
Queen is 1, and so on. We can just use Pieces::KING when we need to index
into a pieces array. Note that the struct is just there as a namespace for
the constants, so there is no type checking in this case. When indexing
into an array of piece types, we MUST make sure that the king pieces are on
index 0, the queen pieces are on index 1, because otherwise it doesn't
work.

>**Sidenote:** This is a bit a C-based way of doing it. It would have been
>better to make the piece types an enum, but when I started Rustic I
>couldn't find a way to automatically convert enums into integers and back
>again, and I didn't want to add a bunch of functions to do so. Maybe I
>will look into this in the future and change some of the struct/const
>setups to actual enums.

The arrays for holding the piece bitboards are contained in the Board
struct, which encompasses all the parts of the board representation. This
struct will be discussed the [last chapter of this
section](./board_struct.md).

The two arrays for the pieces are defined like this:

```rust,ignore
pub bb_pieces: [[Bitboard; NrOf::PIECE_TYPES]; Sides::BOTH],
pub bb_side: [Bitboard; Sides::BOTH],
```

bb_piece contains 6 arrays for white and 6 arrays for black. Each arary is
set up like this, containing only pieces for one color:

- index 0: all kings (always only one)
- index 1: all queens
- index 3: all rooks
- index 4: all bishops
- index 5: all knights
- index 6: all pawns

So if you want to get all the white rooks on the board, you do this:

```rust,ignore
let white_rooks = bb_pieces[Sides::WHITE][Pieces::ROOK];
```

Because this is something we need a lot, there's a function for this. Many
oneliners such as this are wrapped into a function:

```rust,ignore
pub fn get_pieces(&self, side: Side, piece: Piece) -> Bitboard {
    self.bb_pieces[side][piece]
}
```

These functions are described in the section [Board
Functionality](../board_functionality/board_functionality.md).

>**Sidenote:** In many places in the engine there structs with all public
>fields. The board struct is one of those. These structs basically act as
>namespaces for a few variables and the functions that work on them. In the
>future I might decide to encapsulate everything if there is a way without
>having to write thousands of accessor methods and without having to use
>external crates or macro's for this.

There is another array with only two elements:

```rust,ignore
pub bb_side: [Bitboard; Sides::BOTH],
```

This array holds two bitboards: one with all the pieces for the white side,
one with all the pieces for the black side. This is very useful, because we
often need to do things such as this:

```rust,ignore
if (destination_square & bb_side[OPPONENT]) {
    // we captured something
}
```

This is much faster than having to iterate over all the bitboards of the
opponent's color, or having to combine them into a single bitboard first.
When working with pieces, the bb_side bitboard will also be updated to
reflect where all the pieces of each color are. Using this we can do cool
tricks such as this:

```rust,ignore
let empty = !(bb_side[Sides::WHITE] | bb_side[Sides::BLACK])
```

This is the crux of the matter why bitboard-based engines are so blazingly
fast. Because they keep everything in bitboards that can be combined like
this, it is easy to "ask" the board for certain things. The line determines
which squares on the board are empty. It works as follows:

- Take the bb_side bitboard for all of White's pieces.
- Take thee bb_side bitboard for all of Black's pieces.
- Combine them to get all the pieces on the board (called "occupancy").
- Then, all squares that are NOT in this bitboard must be empty.

This is why I keep hammering on the fact that it is essential to understand
how the binary system and bitwise operations work; lines such this one are
going to be throughout the entire engine, everywhere and anywhere, all the
time, especially in the move generator and the evaluation function.

Now let's move on to something easier before your head explodes. The next
chapter deals with the piece list, which is related to the bitboards, but
much easier to understand.
