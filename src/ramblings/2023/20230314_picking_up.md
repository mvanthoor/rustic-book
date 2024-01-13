# March 14, 2023 - Picking up again

You will probably have noticed that there hasn't been a new major version
of Rustic since June 2021 and may be wondering if the engine and this book
are still in active development. [I can assure you that the engine is still
being developed](https://github.com/mvanthoor/rustic/tree/4.0-beta). What
is more: I've been maintaining all current Rustic versions to compile
without errors or warnings using the latest version of Rust, and the latest
versions of the libraries it depends on. Older versions will not receive
new functionality, but they will receive bugfixes or changes to the code if
a library or language update requires this.

I think that this is important to maintain the older versions, because
while there are lots of open source engines, many of them are either buggy,
unstable, or unable to compile with newer compilers and libraries. This
makes testing very hard for newcomers to the chess programming community.
It isn't fun having to fix 3, 5, 10 or 15 year old code first before it can
be compiled correctly. By maintaining older Rustic versions and preventing
them from code-rotting, I hope that there will at least be some engines in
the lower rating regions to test against that can be compiled without issue
if necessary.

So why do work on maintaining old versions instead of releasing version 4?

The reason is simple: life got in the way. Working on, and testing a new
version takes much more work and energy than maintaining the existing
versions, and refactoring the upcoming version to improve the code.

At work we had a massive rewrite of a part of our software due to
country-wide rule and procedure changes made by the Dutch government. This
had to be completely implemented in the second part of 2021 to obtain the
minimum viable product. Most of 2022 was used to debug and fix what was
written, and implement all the needed additional functionality. That took
lots of work and testing and some overwork as well. It left less time for
recreational activities. I chose to spend my free time away from the
computer after programming for work for 8-10+ hours a day.

In the second half of 2022 me and my girlfriend moved from our appartment
to a house in a different part of the city. Obviously that is also not
something you'll do between lunch and dinner... for a big chunk of this
time I didn't have my computer available and only used my backup laptop.

In addition to maintaining the older versions I also slowly added the still
missing functionality to finish support for the XBoard protocol. After this
is done, only the tuner needs to be written to complete version 4.

So, is it the end of Rustic? [Au contraire, mon Capitan. Heee's
baaack!](https://www.youtube.com/watch?v=F2c2NL58Uwc)

