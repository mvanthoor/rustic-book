
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Bitwise operations](#bitwise-operations)
  - [Left and right shift](#left-and-right-shift)
  - [Bitwise NOT (Invert bits)](#bitwise-not-invert-bits)
  - [Bitwise OR (Set bits)](#bitwise-or-set-bits)
  - [Bitwise AND (Get bits)](#bitwise-and-get-bits)
  - [Bitwise XOR (Toggle bit)](#bitwise-xor-toggle-bit)
  - [Unset bit](#unset-bit)
- [Conclusion](#conclusion)

<!-- /code_chunk_output -->

# Bitwise operations

Bitwise operations are the bread and butter of a bitboard-based chess
engine. This chapter explains how these operations work. A chess engine
uses 64-bit integers, but to demonstrate the workings we're going to use an
8-bit integer. This will save writing out long strings of zeroes. For the
demonstration we'll use the same number throughout:

```rust,ignore
decimal:  36
binary:   0010_0100
```

In a bit-wise representation as shown above the most significant bit is on
the left, just as the most important digit is on the left in the decimal
system. This means that the least important bit is on the right. These are
normally called the Most Significant Bit (MSB) and Least Significant Bit
(LSB).

> Note: When talking about bit sets, the Least Significant Bit (LSB) on the
> right can be called "First bit" or "bit 1", referring to the fact that it
> is the first bit in the set. It can also be called "0th bit" or "bit 0",
> because it is at location 0 in the set.

> This can be very confusing, so keep this in mind. I try to be consistent
> and talk about bit location instead of bit number, so in the case of this
> book, the LSB is called "the bit at location 0", or "bit 0". It has the
> advantage of not having to think about subtracting 1 from the bit number
> all the time, to know at which location a bit is.
>
> When reading articles about bitwise operations and manipulations, just
> make sure you know if the author is talking about bit locations (LSB =
> bit 0) or bit numbers (LSB = 1st bit / bit 1).

## Left and right shift

This operation is also often called LSHIFT and RSHIFT. What it does is
shift the bits to the left or the right by a given number of positions.
Note that this can cause bits to "drop out" of the integer. If you shift
the bits so many positions that the integer is too small to hold them (as
in, there are less bits in the integer than what would be required for the
shift to work), then they will be lost. Note that you can't get them back
again by shifting in the opposite direction.

> Note: there is also a "wrapping shift", where a bit that would normally
> drop off of the integer on the left doesn't get lost, but appears again
> on the right. It also works the other way around of course. This is not
> discussed here, because it is only used rarely in chess engine. I mention
> it so you know it exists when you encounter it.

```rust,ignore
fn main() {
    let mut number: u8 = 36;
    
    println!("Starting situation.");
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Shift left by two bits.");
    number = number << 2;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Shift right by one bit.");
    number = number >> 1;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    // Note the alternative, shorter notation of the operator.
    // If you want the result to end up in the same variable
    // as the one you are shifting, you can combine the operator
    // and the assignment. If you do not want the original number
    // to change and catch the result in a different variable, you
    // will have to use the longer notation shown above. 
    println!("Shift right by one bit. We're back at the starting number.");
    number >>= 1;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Shift left by 3 bits.\nThe left-most 1 will drop off the number.");
    number <<= 3;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Shift right by 5 bits.\nThe lost bit is not coming back!");
    number >>= 5;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Shift right by 1 bits.\nNow we lost all the bits.");
    number >>= 1;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
}
```

If you compile and run the above code, this is the result:

```ignore
Starting situation.
decimal: 36
binary: 00100100

Shift left by two bits.
decimal: 144
binary: 10010000

Shift right by one bit.
decimal: 72
binary: 01001000

Shift right by one bit. We're back at the starting number.
decimal: 36
binary: 00100100

Shift left by 3 bits.
The left-most 1 will drop off the number.
decimal: 32
binary: 00100000

Shift right by 5 bits.
The lost bit is not coming back!
decimal: 1
binary: 00000001

Shift right by 1 bits.
Now we lost all the bits.
decimal: 0
binary: 00000000
```

## Bitwise NOT (Invert bits)

This is one of the easier operators to understand. It inverts all the bits
in  set: a 1 becomes a 0 and vice versa. In Rust the bitwise NOT operator
is __!__, while in many other programming languages it is __~__. This is
how its used:

```rust,ignore
fn main() {
    let mut number: u8 = 36;

    println!("Starting situation.");
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");

    println!("Invert all the bits.");
    number = !number;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Invert them back again.");
    number = !number;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n"); 
}
```

And the result of this code is:

```ignore
Starting situation.
decimal: 36
binary: 00100100

Invert all the bits.
decimal: 219
binary: 11011011

Invert them back again.
decimal: 36
binary: 00100100
```

That's it for the bitwise NOT operator.

## Bitwise OR (Set bits)

The bitwise OR operator checks if either the bit within the bit-set or the
bit within the mask is enabled. This operator is represented in Rust by
__|__ (the single vertical pipe). It has the following truth table:

| bit | OR | mask | result |
|-----|----|------|--------|
| 1   | \| | 1    | 1      |
| 0   | \| | 1    | 1      |
| 1   | \| | 0    | 1      |
| 0   | \| | 0    | 0      |

As you can see, the operator returns 1 if one of the bits is set. It only
returns 0 if both bits are not set. This can be used to turn bits in a set
from 0 to one.

> Note: Bit-sets in a chess engine are mostly 64-bits. If we are setting
> or checking for example the bit at location 59 in a set, we would use
> this notation:
>
> number & (1u64 << 59);
>
> What we are effectively doing is creating a huge number by getting a
> 64-bit representation of the number 1. We then shift this 1, which is the
> least significant bit (and the only one in the mask) by 59 locations to
> left. With this notation it is very clear that we just created a mask to
> target the bit at location 59 in the set. Instead of the shift, we could
> also have AND-ed with the number "576460752303423488", but in that way it
> is almost impossible to see what we are doing. This would make your code
> completely unreadable.
>
> If we wanted to create a mask that targets positions 59 and 30 in a set,
> we would do this as follows:
>
> number & ((1u64 << 59) | (1u64 << 30));
>
> On the right side of the &, we first create the mask for the bit at
> location 59, and within this mask, we also set the bit for location 30
> using the bitwise OR operator discussed above.

Here is a series of examples on the workings of this operator:

```rust,ignore
fn main() {
    let mut number: u8 = 36;
    let mut mask: u8;

    println!("Starting situation.");
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Set the bit at location 0");
    mask = 1u8 << 0;
    number = number | mask;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    // Note the alternative notation of
    // the operator, which can be used if
    // you want the result to end up in
    // the same variable.
    println!("Set the bit at location 1");
    mask = 1u8 << 1;
    number |= mask;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Set number to 0 to clean up");
    number = 0;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Set bits at location 0 and 7 (first and last bit)");
    mask = (1u8 << 0) | (1u8 << 7);
    number |= mask;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
}
```

The output of this code is going to be:

```ignore
Starting situation.
decimal: 36
binary: 00100100

Set the bit at location 0
decimal: 37
binary: 00100101

Set the bit at location 1
decimal: 39
binary: 00100111

Set number to 0 to clean up.
decimal: 0
binary: 00000000

Set bits at location 0 and 7 (first and last bit)
decimal: 129
binary: 10000001
```

There is nothing more to it. If you have a bitset and then bitwise OR this
with a mask where one or more bits are enabled, then those bits will be
enabled in the bitset as well (or stay enabled if they where already 1).

## Bitwise AND (Get bits)

It often happens that you want to know if a certain bit is set or not. This
way you can determine if a piece is on a certain square or if a square is
empty or not. This table shows the results of combining a bit with a mask
using the AND-operator:

| bit | AND | mask | result |
|-----|-----|------|--------|
| 1   | &   | 1    | 1      |
| 0   | &   | 1    | 0      |
| 1   | &   | 0    | 0      |
| 0   | &   | 0    | 0      |

A set bit will return 1 if AND-ed with a 1; if it is not set, or AND-ed
with a 0, it will return 0.

Let's use our trusty number 36 again, and explore the workings of the
bitwise AND-operator some more.

```rust,ignore
fn main() {
    let number: u8 = 36;
    let mut mask: u8;
    let mut result: bool;

    println!("Starting situation.");
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");

    println!("Let's see if the bit at location 0 is set.");
    mask = 1u8 << 0;
    result = (number & mask) != 0;
    println!("Bit at location 0 is set: {result}\n");
    
    println!("Now check the bit at location 2 (third bit from the right).");
    mask = 1u8 << 2;
    result = (number & mask) != 0;
    println!("Bit at location 2 is set: {result}\n");
    
    println!("What about BOTH bits at location 2 and 5?");
    mask = (1u8 << 2) | (1u8 << 5);
    result = (number & mask) == mask;
    println!("Both bits at location 2 and 5 are set: {result}\n");
    
    println!("What about BOTH bits at location 2 and 6?");
    mask = (1u8 << 2) | (1u8 << 6);
    result = (number & mask) == mask;
    println!("Both bits at location 2 and 6 are set: {result}\n");
    
    println!("Is AT LEAST one of the bits at location 2 or 6 set?");
    mask = (1u8 << 2) | (1u8 << 6);
    result = (number & mask) != 0;
    println!("At least one of the bits at location 2 or 6 is set: {result}\n");
}
```

The output of this code will be:

```ignore
Starting situation.
decimal: 36
binary: 00100100

Let's see if the bit at location 0 is set.
Bit at location 0 is set: false

Now check the bit at location 2 (third bit from the right).
Bit at location 2 is set: true

What about BOTH bits at location 2 and 5?
Both bits at location 2 and 5 are set: true

What about BOTH bits at location 2 and 6?
Both bits at location 2 and 6 are set: false

Is AT LEAST one of the bits at location 2 or 6 set?
At least one of the bits at location 2 or 6 is set: true
```

## Bitwise XOR (Toggle bit)

The Exclusive OR operator, abbreviated as XOR, can be used to toggle bits
back and forth between disabled and enabled. This operator is represented
by __^__ and it has the following truth-table:

| bit | XOR | mask | result |
|-----|-----|------|--------|
| 1   | ^   | 1    | 0      |
| 0   | ^   | 1    | 1      |
| 1   | ^   | 0    | 1      |
| 0   | ^   | 0    | 0      |

As you can see this operator returns 1 if _either_ the bitset or the mask
have a 1. If _both_ have a 1 or _both_ have 0, the operator returns 0.
Therefore a bit can be toggled from 0 to 1 and from 1 to 0 by the same
mask, as demonstrated by the code below:

```rust,ignore
fn main() {
    let mut number: u8 = 36;
    let mut mask: u8;

    println!("Starting situation.");
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Toggle the bit in location 2 from 1 to 0");
    mask = 1u8 << 2;
    number = number ^ mask;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
    
    println!("Toggle the bit in location 2 from 0 to 1");
    mask = 1u8 << 2;
    number = number ^ mask;
    println!("decimal: {number}");
    println!("binary: {number:08b}\n");
}
```

The output of this code is as follows:

```rust,ignore
Starting situation.
decimal: 36
binary: 00100100

Toggle the bit in location 2 from 1 to 0
decimal: 32
binary: 00100000

Toggle the bit in location 2 from 0 to 1
decimal: 36
binary: 00100100
```

## Unset bit

Sometimes bits in a set need to be disabled. To accomplish this, two
operators must be combined: the bitwise NOT (__!__) and bitwise AND (__&__)
operator. This operation is somewhat more difficult to understand than the
others, so it requires a bit of explanation. Let's say we have a set of
four bits, with the LSB and MSB enabled:

```ignore
bitset = 1001
```

Now we want to disable the most significant bit (the left-most one), so we
create a mask that targets this bit:

```ignore
mask = 1000
```

However, there is no "unset bit" operator. Let's run through some of the
available operators. If necessary, refer to the truth tables discussed
earlier in this chapter.

We can't use AND, because the left-most bit is enabled in both the bitset
and the mask. In addition, the least significant bit will be disabled:

```ignore
1001
1000
----- &
1000
```

Oops. We disabled the wrong bit.

We also can't use NOT, because this will just invert all the bits:

```ignore
!(1001) = 0110
```

That is not what we want. Both the MSB and LSB are disabled, and the two
bits in the middle are now enabled.

Let's see about the OR operator:

```ignore
1001
1000
----- |
1001
```

Meh. Nothing happened, so this can't be right.

XOR then?

```ignore
1001
1000
----- ^
0001
```

Hey, this worked! I can hear you thinking: "So why would we need to do
weird stuff? XOR works fine. If a bit is set, it will disabled it." Yes,
true enough... but now consider that we want to unset a bit _regardless_ if
it was set or not. Let's take our bitset 1001, but now we want to unset the
bit in location 2, even if it is unset already. Our mask will be be 0100,
which targets the bit in location 2.

```ignore
1001
0100
---- ^
1101
```
Oops. The bit in location 2 was already disabled, and XOR flipped it to
enabled. XOR can _only_ be used to disable a bit if we _know_ it is
enabled. So how do we disable a bit, even it is disabled already? We
combine bitwise AND and bitwise NOT. First, we review what happens if we
are disabling a bit that is enabled at the start:

```ignore
bitset  = 1001
mask    = 1000 (disables MSB)

 1001
!1000
------ &
 ?

Flip the mask because of bitwise NOT:

1001
0111
----- &
?

And now we can solve the AND:
1001
0111
----- &
0001
```

As you can see, this works. The MSB is disabled, while the LSB was left
untouched. Now let's try to disable an already disabled bit:

```ignore
bitset  = 1001
mask    = 0100 (disables bit in location 2)

Note that the bit in location 2 is already disabled.

 1001
!0100
------ &
 ?

Flip the mask because of bitwise NOT:

1001
1011
----- &
?

And now we can solve the AND:
1001
1011
----- &
1001
```
The bitset has not changed, which is correct: we created a mask to disable
the bit in location 2. This bit was already disabled and after this
operation it was still disabled. The other bits were not touched, so the
bitset stayed unchanged. Thus the "unset bit" operation were a mask
disables a bit in a bitset can be summarized as:

```ignore
x = bitset & (!bitmask)

Where "x" will contain the new bitset.

or shorter, if you want to use the same variable for the result:

bitset &= !bitmask

This works because the bitwise NOT operator takes precedence
over bitwise AND, so it will be executed first.
```

# Conclusion

There you have it: an explanation of the bitwise operations that are used
within a bitboard-based chess engine. Make sure you understand this
material perfectly, because without it you will struggle to understand what
a bitboard-based chess engine is all about.

For the extremely curious and brave, you could look into the mathematical
field this topic belongs to: [Boolean
algebra](https://en.wikipedia.org/wiki/Boolean_algebra). You are warned
before embarking on that journey though: There be monsters, and insanity is
around every corner... (which is to say that this topic can become
extremely complex.)