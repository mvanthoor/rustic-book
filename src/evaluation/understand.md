# Understanding evaluation

If we could solve chess perfectly, we would have to know only one of three
things: are we checkmated, is the opponent checkmated, or is the game a
draw? To be able to determine this, we would need to look through all
possible combinations of moves, to reach all possible ending positions, so
we can see if either we or the opponent are checkmated, or if the position
cannot be won by any player, we have thus drawn the game.

This is obviously not feasible. The number of chess positions are bigger
than all the atoms in the universe. It would take until the end of time
(and beyond) to calculate all the possible paths to all the possible ending
positions.

We do have opening books with some very long lines in them, and endgame
tablebases containing all endgames with up to 7 pieces (even though they
require a huge amount of storage space). So, the beginning and ending of
the game are coming closer together. The middle game, the part just after
the opening and before the beginning of the endgame database is still vast
enough that this part of the game cannot be bridged by any computer or
program (yet).

Because we cannot see the end of the game, we have to try and win by taking
small steps, move by move. We therefore have to determine if a move is
good. Human chess players use many criteria to determine if a move is good.

If I play this move:
- Will it check the opponent?
- Can I win material with it?
- Do I lose material because of it?
- Does it improve my position?
- Does it make my king (un)safe?

And so on.

We try to come up with an idea, such as: "I want to get my knight to _d5_",
or "I want to open the f-file for an attack." Then we run through the moves
in our head to reach the position where the knight is on _d5_, or the
f-file is open. In humans, this is called "thinking ahead"; in computer
chess, this is called "searching." (How to search will be explained in a
different chapter.)

After we reach the position, we have to determine if this position is good
for us. If you got the knight on _d5_ and won the d5-pawn in the process,
the position may be good for you. If you lost the queen in the process,
it's probably bad; and you'll have to look for a different way to get the
knight on _d5_, without losing the queen.

Determining if the position at the end of a search is good or not, is
called _evaluating the position_. A chess engine therefore has an
_evaluation function_ which has lots and lots of ways to examine a
position.

It asks questions, such as: "Are my pieces on good squares?", "Is my king
safe?", "What is the material balance?", "Do I have the bishop pair?" These
questions are called _evaluation terms_. The evaluation function runs
through all of the terms. Each of these terms carries a certain weight
which is expressed as a value. For example: "I have the bishop pair, so I
raise my evaluation by 25 points."

The function comes up with final value in which all the evaluation terms
have been considered. This value indicates if a position is good or bad.
This called the _evaluation score_. The evaluation score is always
determined from the side which is doing the evaluation: if white is
evaluating and white it a pawn up, the position is +100. If it was black to
move in the same position then he would be a pawn down, and the evaluation
is therefore -100.

This chapter discusses how the engine comes up with this evaluation score.