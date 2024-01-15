
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Handling FEN-strings](#handling-fen-strings)
  - [FEN-string definitions](#fen-string-definitions)
  - [The FEN-parser](#the-fen-parser)
    - [FEN definitions](#fen-definitions)
    - [FEN setup](#fen-setup)
    - [Split the FEN-string](#split-the-fen-string)
    - [Create the FEN part parsers](#create-the-fen-part-parsers)
    - [Part 1: Piece setup](#part-1-piece-setup)
    - [Part 2: Side to move](#part-2-side-to-move)
    - [Part 3: Castling rights](#part-3-castling-rights)
    - [Part 4: En Passant](#part-4-en-passant)
    - [Part 5: Half-Move clock](#part-5-half-move-clock)
    - [Part 6: Full-Move number](#part-6-full-move-number)

<!-- /code_chunk_output -->

# Handling FEN-strings

## FEN-string definitions

Now that we know all the parts of the board struct, it is time we get some
information into it.That means, we're finally ready to set up a position.
The way to do this is by using a so-called FEN-string. "FEN" stands for
__Forsyth–Edwards Notation__, which is a standard to describe chess
positions. The starting position has the following FEN-string:

```
rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1
```

The FEN-string has the following characteristics:

- It has 6 parts. (An FEN-string can have 4 parts however. In this case, 0
  and 1 are assumed for parts 5 and 6.)
- Part 1: describes the pieces and squares.
- Part 2: states which side is to move.
- Part 3: states the castling permissions for both sides.
- Part 4: states the En Passant square, if any.
- Part 5: half-move clock. It marks the number of moves since the last pawn
  push or piece capture.
- Part 6: full-move counter. It marks the number of full moves. It starts
  at 1 and is incremented after black's move.
  
>Note about part 6: This is the number of _upcoming_ full moves. After
>white's first move (for example: _1. e4_), the nmber stays at one, but
>after black's move (for example: _1. ... e5_), it becomes 2. This means
>that the position after _1. e4, e5_ is at full move nr. 2. This is
>correct: white's next move is the first half of the next full move.

The definitions of each part are as follows:
1. Black pieces are denoted with small letters. White pieces are denoted
    with capital letters. The letters are K, Q, R, B, N, and P, which
    stand for King, Queen, Rook, Bischop, Knight and Pawn respectively. A
    number means the number of empty squares after the last piece. The
    forward slash means that the row is finished and we start describing
    the next row. The position setup starts at the top left (black's side,
    seen from white's perspective).
2. Side to move is either "w" for white, or "b" for black.
3. Castling permissions are "K" and "Q" for white's king-side and
   queen-side castling privilege, and "k" and "q" for black's king-side and
   queen-side privileges. If a privilege does not apply, the letter is
   either omitted (or, sometimes) replaced with a dash "-".
4. After a pawn moves a double step, the square directly behind that pawn
   is the En Passant square. This is notated in algebraic notation, for
   example _e3_. (Note: this does not mean the pawn can actually be
   captured en passant. This part is set for EVERY double step pawn move.)
   If the last move wasn't a double step pawn move, this part is a dash
   "-".
5. Half-move clock. This number increases each time a side makes a move. As
   soon as a pawn is moved or a piece is captured, the half-move clock is
   reset to 0. This is used for implementing the 50-move rule.
6. Full-move number. For a chess engine, this number is not really
   relevant. It denotes the full move number that is now going to be made
   from this position. So if this number is _53_, this means the game was
   52 full moves (one white, one black) long, and full move 53 is going to
   be the next one.

## The FEN-parser

To convert the FEN-string into a position, we will have to parse it.
Fortunately this is not difficult in modern programming languages. What
we're going to do are the following steps:

1. Split the FEN-string into its 6 parts.
2. Create a parser function for each part.
3. Create a temporary board.
4. Put each part through its respective parser.
5. As soon as one of the parser functions fails, the FEN-parsing fails.
6. If all the parts are parsed without errors, the board is set up.
6. We initialize everything else that isn't handled by the FEN-string.
7. Put the temporary board into the engine's board.

### FEN definitions

These are the definitions and constants used by the functions in this
module, so we can name things and don't have to repeat them.

```rust, ignore
const FEN_NR_OF_PARTS: usize = 6;
const LIST_OF_PIECES: &str = "kqrbnpKQRBNP";
const EP_SQUARES_WHITE: RangeInclusive<Square> = Squares::A3..=Squares::H3;
const EP_SQUARES_BLACK: RangeInclusive<Square> = Squares::A6..=Squares::H6;
const WHITE_OR_BLACK: &str = "wb";
const SPLITTER: char = '/';
const DASH: char = '-';
const EM_DASH: char = '–';
const SPACE: char = ' ';

#[derive(Debug)]
pub enum FenError {
    IncorrectLength,
    Part1,
    Part2,
    Part3,
    Part4,
    Part5,
    Part6,
}

impl Display for FenError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let error = match self {
            Self::IncorrectLength => "Error in FEN string: Must be 6 parts",
            Self::Part1 => "Error in FEN Part 1: Pieces or squares",
            Self::Part2 => "Error in FEN Part 2: Colors",
            Self::Part3 => "Error in FEN Part 3: Castling rights",
            Self::Part4 => "Error in FEN Part 4: En passant field",
            Self::Part5 => "Error in FEN Part 5: Half-move clock",
            Self::Part6 => "Error in FEN Part 6: Full-move number",
        };
        write!(f, "{error}")
    }
}

pub type FenResult = Result<(), FenError>;
pub type SplitResult = Result<Vec<String>, FenError>;
type FenPartParser = fn(board: &mut Board, part: &str) -> FenResult;
```

### FEN setup

This is the function used to parse an FEN-string (from _Rustic 4_). It
exactly mirrors the steps as laid out above.

```rust,ignore
    // This function reads a provided FEN-string or uses the default
    // position. It sets up the position on the engine's board if parsing
    // succeeds. If parsing fails, the board is not changed.
    pub fn fen_setup(&mut self, fen_string: Option<&str>) -> FenResult {
        // Try to split the FEN-string into 6 parts. Return with an error
        // if this fails.
        let fen_parts = split_fen_string(fen_string)?;

        // Create the 6 FEN-parsers.
        let fen_parsers = create_part_parsers();

        // Create a temporary board so we don't destroy the original if the
        // fen-string happens to be incorrect.
        let mut temp_board = self.clone();
        temp_board.reset();

        // Parse all the parts and check if each one succeeds. If not,
        // immediately return with the error of the offending part.
        for (parser, part) in fen_parsers.iter().zip(fen_parts.iter()) {
            parser(&mut temp_board, part)?;
        }

        // Put the temporary board in the original one's place if setting
        // up the position is succesful.
        temp_board.init();
        *self = temp_board;

        Ok(())
    }
```

That's it. The comments describe what each line does. Below we will take a
look at the functions called by fen_setup(), including the 6 parser
functions.

### Split the FEN-string

This function splits the incoming FEN-string into its component parts. It
does a bit of error handling, such as replacing the sometimes (mistakenly)
used "em-dash" with the normal dash. Also, if the FEN-string turns out to
be 4 parts long, the values 0 and 1 are assumed for the last two parts.

```rust,ignore
fn split_fen_string(fen_string: Option<&str>) -> SplitResult {
    const SHORT_FEN_LENGTH: usize = 4;

    let mut fen_string: Vec<String> = match fen_string {
        Some(fen) => fen,
        None => FEN_START_POSITION,
    }
    .replace(EM_DASH, DASH.encode_utf8(&mut [0; 4]))
    .split(SPACE)
    .map(String::from)
    .collect();

    if fen_string.len() == SHORT_FEN_LENGTH {
        fen_string.append(&mut vec![String::from("0"), String::from("1")]);
    }

    if fen_string.len() != FEN_NR_OF_PARTS {
        return Err(FenError::IncorrectLength);
    }

    Ok(fen_string)
}
```

### Create the FEN part parsers

This function just returns an array of functions that are used for parsing.
Each of these functions has the same type (_FenPartParser_), so the array
can be looped and each function can be called in turn with its
corresponding part. See the fen_setup() function above on how this works in
Rust.
```rust,ignore
fn create_part_parsers() -> [FenPartParser; FEN_NR_OF_PARTS] {
    [
        pieces,
        color,
        castling,
        en_passant,
        half_move_clock,
        full_move_number,
    ]
}
```

### Part 1: Piece setup

This function runs through the first part. If it finds a piece, it puts a 1
into the correct square of the bitboard for that piece. If it finds a
number, it skips that amount of files. When encountering the splitter, it
moves one rank down and resets the file to 0.

If anything is wrong, such as encountering the splitter an any other
position than after 8 files, or if a character is found that doesn't belong
in the FEN-string, the function immediately exits with an error. Parsing
has then failed.

```rust,ignore
fn pieces(board: &mut Board, part: &str) -> FenResult {
    let mut rank = Ranks::R8 as u8;
    let mut file = Files::A as u8;

    // Parse each character; it should be a piece, square count, or splitter.
    for c in part.chars() {
        let square = ((rank * 8) + file) as usize;
        match c {
            'k' => board.bb_pieces[Sides::BLACK][Pieces::KING] |= BB_SQUARES[square],
            'q' => board.bb_pieces[Sides::BLACK][Pieces::QUEEN] |= BB_SQUARES[square],
            'r' => board.bb_pieces[Sides::BLACK][Pieces::ROOK] |= BB_SQUARES[square],
            'b' => board.bb_pieces[Sides::BLACK][Pieces::BISHOP] |= BB_SQUARES[square],
            'n' => board.bb_pieces[Sides::BLACK][Pieces::KNIGHT] |= BB_SQUARES[square],
            'p' => board.bb_pieces[Sides::BLACK][Pieces::PAWN] |= BB_SQUARES[square],
            'K' => board.bb_pieces[Sides::WHITE][Pieces::KING] |= BB_SQUARES[square],
            'Q' => board.bb_pieces[Sides::WHITE][Pieces::QUEEN] |= BB_SQUARES[square],
            'R' => board.bb_pieces[Sides::WHITE][Pieces::ROOK] |= BB_SQUARES[square],
            'B' => board.bb_pieces[Sides::WHITE][Pieces::BISHOP] |= BB_SQUARES[square],
            'N' => board.bb_pieces[Sides::WHITE][Pieces::KNIGHT] |= BB_SQUARES[square],
            'P' => board.bb_pieces[Sides::WHITE][Pieces::PAWN] |= BB_SQUARES[square],
            '1'..='8' => {
                if let Some(x) = c.to_digit(10) {
                    file += x as u8;
                }
            }
            SPLITTER => {
                if file != 8 {
                    return Err(FenError::Part1);
                }
                rank -= 1;
                file = 0;
            }
            _ => return Err(FenError::Part1),
        }

        // If a piece found, advance to the next file. (So we don't need to
        // do this in each piece's match arm above.)
        if LIST_OF_PIECES.contains(c) {
            file += 1;
        }
    }

    Ok(())
}
```

### Part 2: Side to move

The harderst part is already done. The next few parsers are much simpler
and are probably self-explanatory. Parsing succeeds if all the requirements
for the part are met, and the information from the part is put into the
board. If any of the requirements fail, these functions return an error and
parsing fails.

```rust,ignore
fn color(board: &mut Board, part: &str) -> FenResult {
    if_chain! {
        if part.len() == 1;
        if let Some(c) = part.chars().next();
        if WHITE_OR_BLACK.contains(c);
        then {
            match c {
                'w' => board.game_state.active_color = Sides::WHITE as u8,
                'b' => board.game_state.active_color = Sides::BLACK as u8,
                _ => (),
            }
            return Ok(());
        }
    }

    Err(FenError::Part2)
}
```

### Part 3: Castling rights

```rust,ignore
fn castling(board: &mut Board, part: &str) -> FenResult {
    // There should be 1 to 4 castling rights. If no player has castling
    // rights, the character is '-'.
    if (1..=4).contains(&part.len()) {
        // Accepts "-" for no castling rights in addition to leaving out letters.
        for c in part.chars() {
            match c {
                'K' => board.game_state.castling |= Castling::WK,
                'Q' => board.game_state.castling |= Castling::WQ,
                'k' => board.game_state.castling |= Castling::BK,
                'q' => board.game_state.castling |= Castling::BQ,
                '-' => (),
                _ => return Err(FenError::Part3),
            }
        }
        return Ok(());
    }

    Err(FenError::Part3)
}
```

### Part 4: En Passant

```rust,ignore
fn en_passant(board: &mut Board, part: &str) -> FenResult {
    // No en-passant square if length is 1. The character should be a DASH.
    if_chain! {
        if part.len() == 1;
        if let Some(x) = part.chars().next();
        if x == DASH;
        then {
            return Ok(());
        }
    }

    // If length is 2, try to parse the part to a square number.
    if part.len() == 2 {
        let square = parse::algebraic_square_to_number(part);

        match square {
            Some(sq) if EP_SQUARES_WHITE.contains(&sq) || EP_SQUARES_BLACK.contains(&sq) => {
                board.game_state.en_passant = Some(sq as u8);
                return Ok(());
            }
            _ => return Err(FenError::Part4),
        };
    }

    Err(FenError::Part4)
}
```

### Part 5: Half-Move clock

```rust,ignore
fn half_move_clock(board: &mut Board, part: &str) -> FenResult {
    if_chain! {
        if (1..=3).contains(&part.len());
        if let Ok(x) = part.parse::<u8>();
        if x <= MAX_MOVE_RULE;
        then {
            board.game_state.halfmove_clock = x;
            return Ok(());
        }
    }

    Err(FenError::Part5)
}
```

### Part 6: Full-Move number
```rust,ignore
// Part 6: Parse full move number.
fn full_move_number(board: &mut Board, part: &str) -> FenResult {
    if_chain! {
        if !part.is_empty() && part.len() <= 4;
        if let Ok(x) = part.parse::<u16>();
        if x <= (MAX_GAME_MOVES as u16);
        then {
            board.game_state.fullmove_number = x;
            return Ok(());
        }
    }

    Err(FenError::Part6)
}
```

There you go. That's the entire FEN-parsing routine. If everything succeeds,
the board will be set up correctly for the engine to use, either for
playing or analyzing. If any one of the functions fails, the engine will
provide an error string and the internal board will not be changed.

