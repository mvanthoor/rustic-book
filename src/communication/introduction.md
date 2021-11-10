
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Introduction](#introduction)
  - [User-Computer communication through the years](#user-computer-communication-through-the-years)
  - [The problem of the early software](#the-problem-of-the-early-software)
  - [The solution](#the-solution)

<!-- /code_chunk_output -->

# Introduction

## User-Computer communication through the years

A chess engine is a very simple entity: you give it a position and have it
think for some time. When this time is up, the engine returns the best move
it has found. We are now faced with the problem of how we are going to get
positions and moves _into_ the engine, and how the engine can communicate
its reply to us.

The first early commercial chess computers where released in the late
1970's. These computers looked more like a large calculator than a chess
board. Indeed, you often had to put in moves in algebraic notation and then
wait for the computer to print its response on a display. You had to have a
separate normal chess set available to move the pieces, so you could
actually see the position.

In later iterations of this idea the calculator-like interface became
smaller, until it could be integrated into a small chess board. This was
already starting to look like the chess computers that had their high point
in the mid-eighties to the late nineties.

The next step to improve the communication between the player and the chess
computer were the so-called sensory boards. You didn't have to type in your
moves, but could use the piece you wanted to move to press down on the
starting and ending squares. This would let the computer know which move
you made. The computer would then indicate the reply, either on a small
display, or using LED's to indicate the from and to squares of the piece it
wanted to move. You would then go ahead and press those squares for the
computer.

The last step would come in the form of some high-end chess computers
that were coveted back in the day, and are collector's items now. These
computers had reed contacts: you could just move a piece from one square to
he other without having to press down on it. In addition, the computer was
often hidden inside the board in a pull-out drawer. After you set up the
system, you could slide the drawer into the board. The only indication that
his was a chess computer would be the LED's on the board which the computer
would use to indicate its move. Some of these boards were made out of
expensive woods, and the chess computer was built like a module that could
be replaced by a newer version, or one with a different program.

In the beginning of the 80's, we also started to see chess programs for
personal computers. They started off similar to the tabletop chess
computers: in the earliest programs you had to type in moves. In later
programs you could click and drag the pieces using a mouse. This is still
the standard today.

In the late 1990's, the Dutch company DGT released a large tournament-size
chessboard that connects to a computer. You can play moves in the same
style as you do on the reed contact tabletop computers: just move a piece,
and the move is made. These are the boards you see at top-level
tournaments, where the ongoing games are broadcasted over the internet,
without someone having to transfer the move from the player's board to an
internet-connected computer.

## The problem of the early software

In the beginning, resources were limited. Early chess computers came with 2
MHz processors and 1 or 2 kB of useable RAM. In our age, where laptops and
desktop computers have speeds of 2000-4500 MHz, and the normal amount of
memory is at least 8 GB (8.388.608 kB), the resources available on a
computer back in the day seem hilarious. Still, people managed to write
chess programs for them, but they had to be very economical about it. Not a
single byte could be wasted, lest your program had a few bytes less memory
to use, or a few less cpu-cycles available. Tht could make it weaker than
competing programs.

The result of this was that the chess engine, and the communication
capabilities were wrapped up into a single program. The user interface of
the program and the part that calculated the next move, were often
intertwined. In a dedicated chess computer that would mean that you'd need
to rewrite the user interface part if you wanted to use the chess program
in a different computer. On a PC, it would mean that each chess program
came with its own user interface. If you liked the user interface of one
chess program, but would like to use the playing style of another, you'd be
out of luck.

## The solution

The obvious solution to this problem is to split the chess program into two
parts: the engine, and the user interface / GUI and establish a
communication protocol between them. This effort started in the beginning
of the 1990's, when XBoard-author Tim Mann devised a way to use XBoard as a
GUI for the GNU Chess program. The communication between XBoard and GNU
Chess evolved into what we now know as CECP, or the "Chess Engine
Communication Protocol". Many people just call it "XBoard" or "WinBoard"
(which is the Windows-version of XBoard). In this book and in Rustic, this
protocol is also called XBoard.

The XBoard protocol was not designed: it grew as new features were needed.
Nowadays, XBoard even supports chess-like games as "variants". Because of
this, fully and correctly supporting XBoard can be an involved task.

This led to the creation of a communication protocol that was less work to
write. Stefan Meyer-Kahlen, the author of the Shredder chess engine,
designed UCI (Universal Chess Interface) in 2000, which was specifically
written to allow the chess engine developer to spend as little time as
possible on GUI-engine communication. Granted, UCI does not support some
things XBoard can, especially in the case of chess variants. (To support
those, there are several 'dialects' of UCI, for Korean Chess for example.)
Its fast implementation and short specifications did make it the de-facto
standard for current-day chess engines.

Even so, both protocols are equally competent regarding standard chess, and
choosing one or the other (or both) comes down to philosophical reasons.
How you want your chess engine to work is a greater factor in determining
the protocol you are going to use than the technical differences. It is
fully possible to support _both_ protocols in a chess engine, which is
exactly what Rustic does. You obviously have to choose which protocol to
use; you can't run the engine with both protocols active at the same time.

Let's first take a look at a high-level description of both protocols, and
a comparison between how they expect a chess engine to work.