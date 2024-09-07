# Game History

The chess engine needs to keep a history of all positions that occured in
the game, including the associated data. This is needed for functionality
such as move takebacks (which is done during search, for example) and
determining if a game is a draw due to three-fold repetition.

There are three ways to keep the game history:

1. Save the entire board, including all the associated staes, each time a
   move is made. When a move is taken back, you restore the previous
   version of the board with everything in it. This is a simple method to
   understand and relatively easy to implement, but the drawback is that it
   is very slow to save and restore the entire board.
2. It is possible to only store the moves that were played. When a move is
   taken back it is reversed on the board. This means it is not needed to
   save the entire board or the game state. Saving just the move is very
   fast. the problem with this method though is that it needs to not only
   undo the move, but also incrementally undo all the associated data, such
   as the Zobirst Key, Phase and PSQT values, castling, and the en-passant
   square. This will take extra work and calculation on top of reversing
   the move, so restoring a board can still be slow.
3. The last option is to save the game state, and include the move that was
   made from that board position. This stores a bit more data than option
   2, but not nearly as much as option 1. Now, when a move is taken back,
   the previous game state is retrieved from the history and assigned to
   the board. All the variables holding the state have the correct value
   immediately, without any calculation. However, the move that was made
   while the board was still in this state (which is the move we are now
   taking back) is still on the board, so we need to reverse this, just as
   in option 2.

Rustic went with option 3, because it is a happy medium between all of
them. It gets around having to save the entire board and everything in it
from option 1, and also avoids having to revert or recalculate all the game
state variables as they are all recovered instantly from the history.

> ***Sidenote** The one snag is that we can't use the board's own
> functionality to put and remove pieces on the squares. The unmake
> function has its own version that don't touch the incrementally updated
> states (because they are all instantly recovered when using option 3.)
> This will be explained in detail in the Unmake Moves chapter.

The Game History struct to implement this is the last part we need for the
board representation, before we can wrap all the parts into the Board
struct. As stated in the previous chapter, this struct is even simpler than
the Game State one. Here it is:

```rust,ignore
pub struct History {
    list: [GameState; MAX_GAME_MOVES],
    count: usize,
}
```

That's it. This is just an array holding game states: one for each move, up
to the maximum number of moves a game can last. (In Rustic, that is 2048
moves, but no game will ever reach this.) The "count" member keeps track of
the number of elements in use. Now we can write a few useful functions to
make the history work like a stack:

| Function | Goal                               |
|----------|------------------------------------|
| push     | Put a new GameState in the history |
| pop      | Take out the last pushed GameState |
| get_ref  | Get a reference to a GameState     |
| len      | Get the number of used elements    |
| clear    | Clear the history list             |

The entire implementation looks like this:

```rust,ignore
impl History {
    // Create a new history array containing game states.
    pub fn new() -> Self {
        Self {
            list: [GameState::new(); MAX_GAME_MOVES],
            count: 0,
        }
    }

    // Put a new game state into the array.
    pub fn push(&mut self, g: GameState) {
        self.list[self.count] = g;
        self.count += 1;
    }

    // Return the last game state and decrement the counter. The game state is
    // not deleted from the array. If necessary, another game state will just
    // overwrite it.
    pub fn pop(&mut self) -> Option<GameState> {
        if self.count > 0 {
            self.count -= 1;
            Some(self.list[self.count])
        } else {
            None
        }
    }

    pub fn get_ref(&self, index: usize) -> &GameState {
        &self.list[index]
    }

    pub fn len(&self) -> usize {
        self.count
    }

    pub fn clear(&mut self) {
        self.count = 0;
    }
}
```

That's it. This is the entire History implementation. Note that, for speed
reasons, there is no error checking here. If you try to get a reference to
an index that does not exist, the engine will crash. If you have no bugs in
your engine, this should never happen. Still, Rustic's chess logic will
probably get turned into a library (libRustic) in the future, so error
handling may get implemented at that point at the cost of some speed.

Now we have everything ready to create the Board struct, which will be the
final (and most extensive) chapter of the Board Representation section.