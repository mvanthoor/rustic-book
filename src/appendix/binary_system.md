# The binary system

So what is a bitboard? Basically, a bitboard is the bit-wise representation
of a number. To be able to understand this, we'll have to look into how
the binary system works. Here's a quick recap.

First, consider the decimal system. It consists of the digits 0-9, so we
can count 0, 1, 2, 3... up to and including 9. Most people will just keep
counting without thinking about it: 11, 12, 13 ... 19, 20. This is what we
are doing every day and we have become so used to it that most of us don't
really think about what is really happening. Let's take a closer look.

First we count up to and including nine:
```rust,ignore
0, 1, 2, 3, 4, 5, 6, 7, 8, 9
```

So what comes after 9? Most of you will say "ten", which is written as
"10"... but there is no digit "10" in the decimal system. So what has
happened? In the decimal system, the digits above (0-9) are units. It's
called the decimal system because we have 10 digits in it. ("deci" means
"ten".) Should we need more than 9 units, we can create new numbers by
adding another position to the left of the number:

```rust,ignore
0 units
1 units
2 units
...
9 units
10 (one set of "ten", zero units)
11 (one set of "ten", one units)
12 (one set of "ten", two units)
...
19 (one set of "ten", nine units)
20 (two sets of "ten", zero units)
98
99
100 (one set of "ten * ten, or hundred", 0 sets of "ten", zero units)
101 (one set of "ten * ten, or hundred", 0 sets of "ten", one unit)
```

So, if we run out of numbers after 9, what we do is to start on 1 again in
the next position and everything after that becomes 0. The binary system,
which means "system of two", works in exactly the same way, but now we only
use the digits 0 and 1:

```rust,ignore
0: 0 (zero units)
1: 1 (one unit)

// We ran out of numbers...
2: 10 (one set of "two", zero units)
3: 11 (one set of "two", one unit)

We are out of numbers again...
4: 100 (one set of "two * two", zero sets of "two", zero units)
5: 101 (one set of "two * two", zero sets of "two", one units)
6: 110 (one set of "two * two", one set of "two", zero units)
7: 111 (one set of "two * two", one set of "two", one units)
8: 1000 (one set of "two * two * two" ...)
```

If you look closely, you can see the pattern of of 2, 2\*2, 2\*2\*2... and you
can image that, if we need to add another digit, the pattern just repeats.
If we take a 4-bit number "1111", the values of each bit are therefore as
follows:

```rust,ignore
value   =   8   4   2   1
bits    =   1   1   1   1

number  =   8 + 4 + 2 + 1 = 15
```

Now we can count in the binary system:

```rust,ignore
decimal     bit-wise    calculation
            8 4 2 1
--------------------------------------------
0           0 0 0 0     0 
1           0 0 0 1     1
2           0 0 1 0     2
3           0 0 1 1     2 + 1 = 3
4           0 1 0 0     4
5           0 1 0 1     4 + 1 = 5
6           0 1 1 0     4 + 2 = 6
7           0 1 1 1     4 + 2 + 1 = 7
8           1 0 0 0     8
9           1 0 0 1     8 + 1 = 9
10          1 0 1 0     8 + 2 = 10
11          1 0 1 1     8 + 2 + 1 = 11
12          1 1 0 0     8 + 4 = 12
13          1 1 0 1     8 + 4 + 1 = 13
14          1 1 1 0     8 + 4 + 2 = 14
15          1 1 1 1     8 + 4 + 2 + 1 = 15
// and so on
```

I'm sure you can now figure out what the decimal number "16" should become
in binary representation. If not, you should go back and read this page
again; if necessary, find some more information on the internet and
practice a bit with this subject matter. Understanding this perfectly is
*absolutely essential* to be able to write a bitboard based chess engine.
Counting in the binary system and combining numbers (or more correctly,
sets of bits) with one another, comes up over and over again.
