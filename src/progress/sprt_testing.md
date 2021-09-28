## SPRT testing

One of the best ways to test one engine against another is by using SPRT,
which stands for "Sequential Probability Ratio Test." A program which can
run such matches is _cutechess-cli_, by using the SPRT parameter.

It is beyond the scope of this book to explain all the mathematics behind
these formulas, even _if_ I'd understand everything in detail. I'm a
programmer, not a Statistician.

It is sufficient to understand the basics.

You run a match between two engines. This match does not have a set length,
but to make sure it doesn't run 'forever', we normally set a very large
number of games, such as 20.000. To make SPRT work, we need to set two
hypotheses: one we hope that comes true (H1), and one we hope that isn't
true (H0, the NULL hypothesis). We allow for a chance of 5% that the
SPRT-test gives us the wrong result. So, we are 95% confident that the
result is correct. (You can lower the 5% margin, but the test will run _a
lot longer_.)

Let's say we have a new engine, called version NEW. We also have an old
version, we call version OLD.

Now we state:

- H1: Engine NEW is at lesat 1 Elo stronger than engine OLD.
- H0: Engine NEW is NOT more than 5 Elo stronger than Engine OLD.
- Error margin: 5%.

When running cutechess_cli, we give it the SPRT parameter:

```
-sprt elo0=1 elo1=5 alpha=0.05 beta=0.05
```

(Alpha an Beta have nothing to do with Alpha/Beta searching. In this case,
Alpha is the chance we accept H1 while we shouldn't, and Beta is the chance
we accept H0 while we shouldn't; i.e., a chance of 5% we get the wrong
result from the test.)

A cutechess_cli command could look like this:

```bash
cutechess-cli \
-engine conf="Rustic Alpha 3.15.100" \
-engine conf="Rustic Alpha 3.1.112" \
-each \
    tc=inf/10+0.1 \
    book="/home/marcel/Chess/OpeningBooks/gm1950.bin" \
    bookdepth=4 \
-games 2 -rounds 2500 -repeat 2 -maxmoves 200 \
-sprt elo0=0 elo1=10 alpha=0.05 beta=0.05 \
-concurrency 4 \
-ratinginterval 10 \
-pgnout "/home/marcel/Chess/sprt.pgn"
```

With this command, we're testing 3.15.100 (NEW) against 3.1.112 (OLD),
where we run 2500 rounds with 2 games each, so each engine plays the same
opening with white and black. Time control is 10 seconds + 0.1 increment.

Now we start the match between our NEW and OLD (previous) engine version,
and cutechess_cli will start to play games.

Let's say that, after 400 games, Alpha 2 is 100 Elo stronger than Alpha 1.
This could still change, if you play enough games... but that is the point
of SPRT testing. As soon as cutechess_cli is 95% sure that the difference
between the two engines is not going to change anymore, it will abort the
match, which saves _a lot of time_.

At that point, cutechess_cli compares the result in Elo against the stated
hypotheses. With a result of +100 Elo for engine NEW, we can see:

- H1: NEW is at least 1 Elo stronger than OLD. True, because 100 > 1.
- H0: NEW is _NOT_ more than 5 Elo stronger than OLD. False, because it IS
  more than 5 Elo stronger (100 Elo is more than 5 Elo).

So H1 is accepted, and we have determined that NEW is a stronger engine
than OLD, and by how much (self-play) Elo.

If NEW scored -20 Elo, then we would have had this result:

- H1: NEW is at least 1 Elo stronger than OLD. False, because -10 < 1.
- H0: NEW is NOT more than 5 Elo stronger than Old. True, because -10 is
  indeed less than 5 Elo.

So H0 is accepted, which means that our NEW engine is not stronger than our
OLD engine; it is actually weaker, and we should not include this feature.
(At least, not yet: a feature which causes a strength loss now, could cause
a strength gain when added on top of other features, so it's worth it to
try again later.)

As long as the difference between NEW and OLD is between 1 and 5 Elo, the
SPRT-test keeps running, because both hypotheses are still true: 3 Elo is
"at least 1 Elo stronger", but it is also "not more than 5 Elo stronger."
If this result doesn't change, cutechess_cli would run forever; that is the
reason why we set a match limit of something like 20.000 games.