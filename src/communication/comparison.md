
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [XBoard vs. UCI](#xboard-vs-uci)
  - [Comparison](#comparison)
  - [Philosophy for XBoard](#philosophy-for-xboard)
  - [Philosophy for UCI](#philosophy-for-uci)
- [Summary](#summary)

<!-- /code_chunk_output -->
# XBoard vs. UCI
## Comparison

Below is a table comparing some of the most important differences between
the UCI and XBoard protocols.

|                   | XBoard     | UCI        |
|-------------------|------------|------------|
| Commands          | Short      | Long       |
| Commands in       | 18         | 7          |
| Commands out      | 13         | 9          |
| Supports features | yes        | yes        |
| Supports openings | yes        | yes        |
| Debug mode        | yes        | yes        |
| Messages          | yes        | yes        |
| Time management   | yes (game) | yes (move) |
| Play in terminal  | yes        | bothersome |
| Protocol type     | Stateful   | Stateless  |
| Who is the boss?  | Engine     | GUI        |
| Can resign        | yes        | no         |
| Can offer draw    | yes        | no         |
| Can accept draw   | yes        | no         |
| Supports variants | yes        | dialects   |

As you can see int he table above, there are many similarities between the
two protocols, but also many differences. The first three rows already
indicate a large difference. Rustic implements 7 incoming and 9 outgoing
commands for UCI; it implements 18 incoming and 13 outgoing commands for
XBoard.

The short commands used by XBoard were an advantage in the past, when
parsing commands was done on a character by character basis. Parsing long
commands was very tedious and more importantly, very error prone. It was
even harder and more tedious to write good error checking routines.
However, having more commands also means more stuff to write.

Nowadays many newer programming languages don't require you to parse
commands character by character, building tokens as you go. They have
splitting functions, iterators, built-in parsing functions (from string to
int, for example) _including_ error checks, and so on. Therefore the long
commands sent and received by UCI are much less of a hassle with current
programming languages.

The other big difference is the fact that XBoard is stateful, where UCI is
stateless. An XBoard engine needs to be aware of the state of the game to
be able to play. It also needs to be aware of the state the engine is in
(observing, waiting, thinking, analyzing). This is not required for UCI, so
creating this functionality again requires more code.

In short, UCI is faster to implement by writing less code, as it sends much
more information back and forth per command. It also doesn't have to deal
with engine and game state. This was one of UCI's design goals. This is the
main reason why UCI is now the default protocol for most newer engines.

The difference between "stateless" (UCI) and "stateful" (XBoard) is the one
where the philosophical debate is at. I'll try to explain.

## Philosophy for XBoard

The XBoard protocol was designed in a time where user interfaces on
computers were not as far along developed as they are today. They were very
basic: displaying a the board, a clock, the move list, and (maybe) the
variation the engine was thinking about. There were two ways of having
chess engines play against one another:

- use the user interface as a match/tournament organizer.
- run each engine on its own computer, communicating through a serial
  interface connecting the two computers.

Because resources and user interface functionality was limited (and
sometimes, there even _wasn't_ a user interface involved in an engine vs.
engine match), XBoard was designed to make the engine smart.

Smart engine, dumb user interface.

What this means is that the engine functions a lot like a human would.

- At the start of the game, you give it a time control.
- The engine has its own opening book.
- It knows if it is playing black or white.
- It knows it is on move after the opponent has moved.
- It knows all the rules of chess.
- When lost, it can resign.
- It can claim draws by the rules such as three-fold repetition.
- When it thinks the position is equal, it offers a draw.
- When a draw offer comes in, it can be accepted or rejected.

You give both engines the information they need at the start of the game,
and then send _"go"_ to the engine playing white. The engine will make a
move and send it to the opponent; which can be a human or another engine.
If it is another engine, that other engine executes the incoming move. It
knows it is expected to move now, so it starts thinking. Depending on the
time control (depth, time per move, etc...) it terminates its search at an
appropriate moment and returns the best move it found to the first engine.

And that's it: the engines are now playing a game of chess all by
themselves until either one is mated or resigns, a draw is claimed by one
of them on the basis of the rules, or they agree on a draw.

When using the XBoard protocol, the engine is a chess playing entity.

## Philosophy for UCI

Playing chess is not an easy task. There's lots of rules you need to know
and many situations can arise. So, you have to keep the state of the game
within the engine to know if you exceeded the fifty-move rule, have a
three-fold repetition, if you are even on move, if the engine is waiting or
thinking, and so on... let alone write routines for loading and searching
an opening book, determine if you should resign or not, accept or offer a
draw or not, and so on. All of this requires extra code and extra testing.

By the year 2000, computers had progressed enough so that running an
advanced user interface that could run matches and tournaments between
engines on the same computer (giving the CPU first to one engine, then the
other) was completely possible. Also: why should each engine implement code
to read an opening book (or endgame tablebases), or verify draws or
determine if it should resign? Those could be options in the user
interface, where they could be set by the user, instead of leaving them up
to the engine authors. (Even though those authors could obviously make
those options configurable; but that was not a requirement.)

So the idea was: let's just rip everything that is _not_ move searching
from the engine and have this handled by the user interface.

Dumb engine, smart user interface.

UCI was created in 2000.

Because UCI does not require the engine to keep its state, the user
interface sends long commands to the engine that make it set up its board
from scratch each time a move is made. For example, the GUI sends
_"position startpos moves e2e4 e7e5 g1f3 b8c6"_ to the engine playing
white, after black played Nb8-c6. The white engine sets up its starting
position and plays all the moves after _"moves"_... and then it waits.
Next, the GUI sends a _"go"_ command with the time controls for the next
move. Thus, UCI re-sends the game state and time control on each move, over
and over again, so the engine does not have to maintain the game state.

It also doesn't have to maintain engine state (idle, thinking, analyzing,
etc...) because UCI requires that the user interface keeps track of this
and does not send inappropriate commands for the state the engine is in.

As you can imagine, this setup requires a lot less code to write by the
chess engine programmer. The disadvantage incurred by this ease of use is
that the engine cannot resign anymore, nor can't it offer, accept, or claim
draws. These tasks are now done by the user interface.

When using UCI, the engine is a "find the best move in this particular
position" program.

# Summary

After the UCI-protocol was first introduced there have been heated debates
on which of the two is "better": XBoard, which makes the engine behave more
like you would expect from a human, or UCI, which is cleaner, faster and
somewhat easier to implement. For a few years it was a toss-up between the
established XBoard protocol and the UCI newcomer.

Chessbase, the largest supplier of chess software in the world, supported
UCI almost immediately with Fritz 7, released in 2001. Before that date
Fritz supported alternative engines but they used a proprietary Chessbase
protocol. To get a chess engine to run under the Chessbase / Fritz GUI, you
had to submit it to Chessbase and hope they would adapt it and offer it for
download. As of November 2021, this download page is still available
through the WayBack Machine (late 2006 version). The downloads still work,
so if you want the old native Chessbase engines, grab them as soon as
possible. (I can recommend Turing, which can be a nice match for an average
club player.)

[Click this link to visit the old download page](https://web.archive.org/web/20061111015327/http://www.chessbase.com/download/index.asp?cat=Engines)

This page was taken offline in 2007. In newer versions of Fritz, Chessbase
completely switched over to the UCI protocol.

At this time, the most important competitor to Fritz was the ChessMaster
GUI. It supported the XBoard protocol for the use of alternative engines,
which was a great plus over Fritz's proprietary protocol. Many free
XBoard/WinBoard chess engines could be found across the internet that could
be used with Chessmaster, but not with Fritz. (Except if you would use an
XBoard-to-UCI adapter program, but this is not a user-friendly solution for
the user groups Fritz and Chessmaster were targeting.) In the years after
2001 though, this GUI waned in popularity and the last version running The
King 3.50 by Johan de Koning, was released in 2007. At the time of writing,
this was almost 15 years ago.

UCI gained traction as Fritz 7 and newer got into user's hands while
ChessMaster's popularity diminished. Nowadays UCI is the de-facto standard,
while most new engines do not implement the XBoard protocol anymore.

Rustic still implements both UCI and XBoard for several reasons:

- I'm a completionist. Most GUI's support XBoard. So why not?
- I like to have and provide options. Maybe some people would like to use
  Rustic with a GUI which only supports XBoard and don't want to use a
  UCI-XBoard adapter.
- I wanted to have a go at implementing two different protocols in Rustic
  without making the core of the engine aware of the fact that it is
  running in one or the other. (Apart from forwarding incoming commands to
  the correct handler.)
- I like the thought of the engine being a "chess playing entity" instead
  of a "find the best move" program.

The two protocols are on par in Rustic, except with regard to reporting
some statistics. UCI provides a few more stats. Even though XBoard can also
provide those statistics, the command to provide them is not supported by
most user interfaces I've tested.

Now that we have a general idea of both protocols and their differences, we
can get into designing a communication module for Rustic.