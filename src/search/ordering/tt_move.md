## TT-move ordering

(Implemented since Rustic Alpha 2.)

TT-move ordering is a technique where a move returned from the
transposition table is ordered first in the list, in front of all other
moves. The hard part is implementing the TT itself; ordering on the TT-move
is very easy.

You should have already implemented the TT to be able to add the TT-move
ordering. If you haven't, it's recommended to take a look at the
Transposition table chapter. It explains what the TT is exactly, how it
works, and how it can be implemented. Then add create the TT first. In that
chapter it's also explained that the TT can store moves which  cause a beta
cutoff, or are part of the principal variation.

It should be very obvious by now why you want to try either of those moves
ASAP. If a move causes a beta cutoff, you want to try it as soon as
possible, because it saves you having to search large parts of the tree. If
a move is a PV-move, you also want to try it as soon as possible, as it
makes you find your best move faster. Again, the result is that you have to
search less.

This is the alpha/beta function with all the parts removed that have
nothing to do with TT-move ordering:

```csharp
impl Search {
    pub fn alpha_beta( ... ) -> i16 {

        // ...

        // Variables to hold TT value and move if any.
        let mut tt_value: Option<i16> = None;
        let mut tt_move: ShortMove = ShortMove::new(0);

        // Probe the TT for information.
        if refs.tt_enabled {
            if let Some(data) = refs
                .tt
                .lock()
                .expect(ErrFatal::LOCK)
                .probe(refs.board.game_state.zobrist_key)
            {
                let tt_result = data.get(depth, refs.search_info.ply, alpha, beta);
                tt_value = tt_result.0;
                tt_move = tt_result.1;
            }
        }

        // ...

        // Generate the moves in this position
        let mut move_list = MoveList::new();
        refs.mg
            .generate_moves(refs.board, &mut move_list, MoveType::All);

        // Do move scoring, so the best move will be searched first.
        Search::score_moves(&mut move_list, tt_move, refs);

        // ...

        // Iterate over the moves.
        for i in 0..move_list.len() {
            Search::pick_move(&mut move_list, i);
            let current_move = move_list.get_move(i);

            // ...
        }

        // ...
    }
}
```

 We have two variables, _tt\_value_ and _tt\_move_ which contain the value
and the move coming from the TT, if there are any. Assume we have a TT-move
in the _tt\_move_ variable. We generate all moves as we normally do, and
then call score_moves(). We pass the tt_move variable into score_moves().
We have seen this function before, but now it has been extended to take the
TT-move into account:

```csharp
const TTMOVE_SORT_VALUE: u32 = 60;

pub fn score_moves(ml: &mut MoveList, tt_move: ShortMove, refs: &SearchRefs) {
    for i in 0..ml.len() {
        let m = ml.get_mut_move(i);
        let mut value: u32 = 0;

        // Sort order priority is: TT Move first, then captures, then
        // quiet moves that are in the list of killer moves.
        if m.get_move() == tt_move.get_move() {
            value = MVV_LVA_OFFSET + TTMOVE_SORT_VALUE;
        } else if m.captured() != Pieces::NONE {
            // Order captures higher than MVV_LVA_OFFSET
            value = MVV_LVA_OFFSET + MVV_LVA[m.captured()][m.piece()] as u32;
        } else {
            let ply = refs.search_info.ply as usize;
            let mut n = 0;
            while n < MAX_KILLER_MOVES && value == 0 {
                let killer = refs.search_info.killer_moves[ply][n];
                if m.get_move() == killer.get_move() {
                    // Order killers below MVV_LVA_OFFSET
                    value = MVV_LVA_OFFSET - ((i as u32 + 1) * KILLER_VALUE);
                }
                n += 1;
            }
        }

        m.set_sort_score(value);
    }
}
```

The constant _TTMOVE\_SORT\_VALUE_ was added, which is higher than the
highest value from the MVV-LVA table. (See the chapter about MVV-LVA move
ordering, should you have forgotten this.) We then iterate through all the
moves, but now we first take a look if the move we are at in the list, is
the TT-Move which was just passed into the function. If so, we assign
MVV_LVA_OFFSET + TTMOVE_SORT_VALUE to _"value"_, and the TT-move will
therefore be ordered higher than any move ordered by MVV-LVA. As a result,
this move will be put first by the _pick\_move()_ function.

That's all, folks. Told you this was an easy one 😊 (After you implement
the TT itself that is, which isn't that easy at all 😱)

> **Sidenote: what about ordering on the PV-move?** There is a technique
> called PV-move ordering, which orders the best move from the previous
> iteration in the first spot. Ordering the move works the same was is with
> the TT-move; the only difference is that you pass the PV-move to the
> score_move() function instead of the TT-move. This is easier to
> implement, because you don't need a TT for it. As the TT stores the
> PV-moves (and the cut-moves), PV-move ordering is inherently built into
> TT-move ordering. If you have a TT, PV-move ordering becomes superfluous.
> 
> There is a chance your the TT-entry holding the PV-move for the position
> you are in was overwritten with a different entry so you have no PV-move
> to order on. As far as I know, many engines take this risk for granted
> and don't implement PV-move ordering is a fallback. It's probably not
> worth it with regard to Elo gain. Rustic does not implement PV-move
> ordering.
