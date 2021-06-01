# How move ordering works

Move ordering is a very important part of a chess engine, as described in
the previous chapter. There are several techniques involved, but if you get
right down the basics, what happens is actually quite easy to understand.
Let's take the alpha-beta function, which performs the core part of the
search in a chess engine. To keep this simple, all parts not related to
move ordering have been removed.

```javascript
pub fn alpha_beta(
        mut depth: i8,
        mut alpha: i16,
        beta: i16,
        pv: &mut Vec<Move>,
        refs: &mut SearchRefs,
    ) -> i16 {
        // Early alpha/beta exit conditions are here
        ...

        // Generate all the moves for this position.
        refs.mg .generate_moves(refs.board, &mut move_list, MoveType::All);

        // Do move scoring, so the best move will be searched first.
        Search::score_moves(&mut move_list, tt_move, refs);

        // Loop through all the moves, where moves will be picked.
        for i in 0..move_list.len() {
            // Put the move with the higest sort socre at the loop's index.
            Search::pick_move(&mut move_list, i);

            // Get this move and examine it.
            let current_move = move_list.get_move(i);

            // Search logic here ...
            ...
            ...
        }

        // We found the value for our best move. Return this.
        alpha
    }
```

This is basically it. The alpha-beta function receives everything it needs
for its search. These will be explained in detail on the page about the
alpha/beta-function. Further down, the function does the following things:

1. Generate all the moves
2. Call score_moves()
    - Pass a reference to the move list that was just generated.
    - Pass the transposition table move.
    - Pass the collection with references that hold information about the
      search.
3. score_moves() iterates over the move list, giving each move its own sort
   score (using the SORTSCORE field in the move integer).
4. Then alpha/beta starts examining moves, by iterating over the move list
   itself. The loop starts counting at 0 (first move in the list), and
   obviously ends with the last legal move in the position.
5. pick_move() puts the move with the highest sort score at the index where
   the move loop currently is.
6. Then that move is put into _current_move_, so we don't have to
   move_list.get_move(i) over and over again in the loop.

So, only points 2 and 5 are actually involved in ordering, and then picking
the moves. We'll see score_move() in several different versions when we
discuss the move ordering techniques, but pick_move() is always the same:

```javascript
pub fn pick_move(ml: &mut MoveList, start_index: u8) {
    for i in (start_index + 1)..ml.len() {
        if ml.get_move(i).get_sort_score() > ml.get_move(start_index).get_sort_score() {
            ml.swap(start_index as usize, i as usize);
        }
    }
}
```

It receives a reference to the move list, and the index it should start at.
This is the index where alpha/beta's move loop currently is. Then
pick_move() runs through the list, and keeps swapping moves with higher
scores into the start_index location. In the end, the move having the
higest sort score will end there, so alpha/beta will examine this move
next. (This function could be optimized a tiny bit to do only ONE swap, but
for some reason, this made Rustic slower each time I've tried it. Maybe
this optimization will make it in later.)

> **Sidenote** Why do we assign sort scores to the moves, and then use
> pick_move() to swap one move to the current index of the move list while
> alpha/beta iterates over it? Couldn't we just actually _physically sort
> the list_ before the move loop starts, and be done with it?
>
> It's possible, but we don't do that because of how alpha/beta works. If
> alpha/beta encounters a move that is so good that searching further down
> the move list is no longer required, then it will exit and return the
> evaluation score of that move. (This is a so-called bèta-cutoff.) If you
> physically sorted all the moves before the move loop starts, you may have
> sorted lots of moves that may never be examined by alpha/beta. This costs
> time, and thus it makes the engine slower.
> 
> You can hear and read "move ordering" and "move sorting" interchangeably.
> The difference is that "move ordering" does the "score and pick"
> approach, and "move sorting" actually physically sorts the entire move
> list. The result is the same, but move ordering is the faster approach,
> as described above.