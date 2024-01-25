
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Tapering the evaluation](#tapering-the-evaluation)
  - [Understanding the concept](#understanding-the-concept)
  - [Implementation](#implementation)

<!-- /code_chunk_output -->

# Tapering the evaluation

## Understanding the concept

In the previous chapters where we talked about the PSQT's it was already
mentioned a few times that we would be 'tapering the evaluation.' This
chapter discusses this concept using the PSQT's, but it can be applied to
any other evaluation term in the future. Let's say, we have a bonus for
having the bishop pair:

```rust, ignore
const BISHOP_PAIR: i16 = 10;
```

In this case, having both bishops will increase the evaluation for that
side by +10 centipawns. However, the more the board opens up, the more
advantageous the bishop pair becomes. In the endgame, the advantage may be
(for example) as high as 40 centipawns. Thus the importance of this
evaluation term increases the more we are going into the endgame. If we
want to encode this, we could do it as follows, so we don't have to make an
extra variable:

```rust,ignore
pub struct W(pub i16, pub i16);
const BISHOP_PAIR: W = W(10, 40);
```

Here we defined a type called W.

>__Sidenote:__ The type declaration comes from "weight", which is the chess
engine/machine learning name of how much impact a certain feature has.
We're using "W" instead of "Weight" because we need it later, in the PSQT,
and the table would become extremely wide. You'll see in a moment.

The type can hold two i16 values, so we can now put TWO values in the
BISHOP_PAIR constant. The first value can be read by using
_"BISHOP_PAIR.0"_, and the second one is _"BISHOP_PAIR.1"_; The first value
is the one for the opening, the second one is the one for the endgame.

When the game is in the opening, we use 100% of the first value and 0% of
the second value. This means that in the opening the BISHOP_PAIR value is
calculated like this:

```rust, ignore
let value = BISHOP_PAIR.0 * 1 + BISHOP_PAIR.1 * 0
```

This boils down to 10 + 0 = 10.

However, when we are half-way through the game, so exactly in the middle
game, we would use 50% of the first value and 50% of the second, which
would give us this:

```rust, ignore
let value = BISHOP_PAIR.0 * 0.5 + BISHOP_PAIR.1 * 0.5
```

This will come down to (10 * 0.5) + (40 * 0.5), which is 5 + 20 = 25. As
you can see, the value of the bishop pair is increasing as we approach the
endgame. You can guess that the value will end up at 0% opening and 100%
endgame, which will turn out to be 40, at some point in the game.

This process of calculating in-between values is called _interpolation_.
The chess engine world also calls it _tapering_, because the opening value
gradually tapers into the endgame value as the game progresses.

This works exactly the same within a PSQT. The only difference is that the
PSQT encodes a piece-on-square relationship. Thus, the QUEEN PSQT has 64
elements (because of 64 squares), and each element has two values (opening
and endgame), and there will be 6 PSQT's (because there are 6 piece types).
The QUEEN PSQT may look like this:

```rust,ignore
pub struct W(pub i16, pub i16);
const QUEEN: Psqt = 
[
    W(865,918), W(902,937), W(922,943), W(911,945), W(964,934), W(948,926), W(933,924), W(928,942),
    W(886,907), W(865,945), W(903,946), W(921,951), W(888,982), W(951,933), W(923,928), W(940,912),
    W(902,896), W(901,921), W(907,926), W(919,967), W(936,963), W(978,937), W(965,924), W(966,915),
    W(881,926), W(885,944), W(897,939), W(894,962), W(898,983), W(929,957), W(906,981), W(915,950),
    W(907,893), W(884,949), W(899,942), W(896,970), W(904,952), W(906,956), W(912,953), W(911,936),
    W(895,911), W(916,892), W(900,933), W(902,928), W(904,934), W(912,942), W(924,934), W(917,924),
    W(874,907), W(899,898), W(918,883), W(908,903), W(915,903), W(924,893), W(911,886), W(906,888),
    W(906,886), W(899,887), W(906,890), W(918,872), W(898,916), W(890,890), W(878,906), W(858,879),
];
```

>__Sidenote:__ See what I meant above? If we had used "Weight" instead of
>"W", this table would have been massively wide.

Remember that the PSQT is shown from white's point of view, so the lower
left corner is A1. When the queen is in the corner on A1 in the opening,
it's worth is 906 cp (centipawns), but if its there in the endgame, its
worth drops to 886 cp. This means that your queen will lose value if you
leave it in the corner in the endgame. When you look at square D4 (4th from
the left, 4th from the bottom), you'll see that the queen's worth is 896 in
the opening, but 970 cp in the endgame. This makes sense: you DON'T want to
have your queen in the middle of the board in the middle game because it's
vulnerable there, but in the endgame, the queen would control the entire
board from that square.

So, by applying the PSQT to the position, we now have an opening value and
an endgame value for each piece. During the game we can calculate the value
of the piece, on a certain square, by _interpolating_ between these two
values. We do so for every piece on every square. Now let's look at how to
implement this.

## Implementation

I am assuming you are writing your evaluation in such a way that material
values are folded into the PSQT (like with the queen above), and that each
element in the PSQT holds two values. You could maintain separate material
values and use a QUEEN_OT and QUEEN_ET table (for OpeningTable and
EndgameTable), but all those extra variables just makes it more laborious
to work with. Putting the material and both values into one array is much
more convenient, and that is the reason why Rustic switched to this format
between Alpha 3 and version 4.