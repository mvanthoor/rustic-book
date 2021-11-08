## SPRT testing

One of the best ways to test one engine against another is by using SPRT,
which stands for "Sequential Probability Ratio Test." A program which can
run such matches is
[_cutechess-cli_](https://github.com/cutechess/cutechess), by using the
SPRT parameter.

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
result is correct. (You can lower the 5% margin, but the test will run for
a _very_ long time. I deem 5% to still be practical; it is also the value
used by many other engine authors.)

Let's say we have a new engine, called version NEW. We also have an old
version, we call version OLD. We want to know if NEW is stronger than OLD,
and by how much Elo.

Now we state:

- H0: Engine NEW is at least 0 Elo stronger than OLD.
- H1: Engine NEW is stronger than OLD outside an error margin of 10 Elo.
- We set confidence level at 95%, so there's a 5% chance of getting the
  wrong result from the test.

When running cutechess_cli, we give it the SPRT parameter:

```
-sprt elo0=0 elo1=10 alpha=0.05 beta=0.05
```

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

Now we start the match between our NEW and OLD engine version,
and cutechess_cli will start to play games.

As long as the difference between NEW and OLD is between 0 and 10 Elo, the
SPRT-test keeps running, because it cannot be determined which hypothesis
is true. If NEW is 7 Elo stronger, then NEW is at least 0 Elo stronger than
OLD, but it isn't outside the 10 Elo error margin yet. The test could still
go either way. If this stays the case up to and including the maximum
number of games to play, then cutechess_cli will abort without accepting
either hypothesis, and the test is inconclusive.

On the other hand, if NEW is +100 Elo stronger than OLD after only 400
games, that is _clearly_ outside the 10 Elo margin required for H1 to be
accepted. As soon as cutechess_cli is 95% certain that the result will stay
outside the error margin, the test will be aborted, and H1 accepted.

Likewise, if NEW is -50 Elo, then this is clearly weaker than OLD, and the
error margin is outside the stated 10 Elo. As soon as cutechess_cli is
95% certain that the test will stay outside the error margin, the test is
aborted and H0 is accepted.

If you increase elo1, the error margin becomes larger, and thus cutechess
will be faster to terminate the test. Your self-play Elo will become much
less accurate tough. Let's say you set this:

```
-sprt elo0=0 elo1=50 alpha=0.05 beta=0.05
```

If NEW is 60 Elo stronger, and the error margin is 50 Elo, cutechess_cli
may accept that the engine is stronger, because 60 is outside the 50 Elo
margin. Even though NEW is stronger, the strength range is 10-110 Elo: NEW
can be anywhere between 10 and 110 Elo stronger, which is a very inaccurate
determination. If you had set elo1 to 10, the range would have been 50-70
Elo. If you set elo1 to 2 Elo, the range would be 58-62 Elo, but that will
take a very long time to test. I've found 10 Elo to be a fair setting, at
least on my (current) i7-6700K running 4 games concurrently at 10s+0.1s.