# Game History

The Game History struct is the last part we need for the board
representation, before we can wrap all the parts into the Board struct. As
stated in the previous chapter, this struct is even simpler than the Game
State one. Here it is:

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

