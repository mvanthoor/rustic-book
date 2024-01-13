# March 15, 2023 - A new computer

In 2023 my current computer will be 7 years old. It was built in the second
half of 2016 and it has four cores and 32 GB of RAM. To be honest, for all
of my day-to-day use this system would still be sufficient. I do not play
many games: The Witcher 3 is still the most demanding game I own (if you
don't count the next-gen update), and it runs fine on this system. If it
hadn't been for chess programming, I wouldn't have considered an update for
another 1-3 years.

However, I did get into chess programming, and the amount of required
testing and tuning is slowly increasing. I have to test each version of
Rustic in self-play, but also run the new version in a gauntlet against
other engines. Because I wish to run at least 1000 games for each engine
Rustic plays against, this gauntlet takes around 8 hours on this computer.
It can run 4 games at once (one per core, as I prefer not to use
hyperthreading with such a low core count), and it clearly isn't enough. I
don't want to wait for 8 hours plus the self-play test to see the result of
a change. Also, I don't (yet) want to be dependent on something like
OpenBench.

Therefore I decided to build a new workstation. At first, the plan was to
build this computer shortly after the release of Debian 12 (I switched to
Debian Linux permanently in 2021), which would be somewhere in July or
August 2023. The computer hardware world seems to have turned crazy, at
least in the Netherlands. From one day to the next, prices on motherboards
can fluctuate by as much as €150 up or down, even though the GPU's are a
bit more stable since the last two years. Parts can go from available
everywhere on Tuesday, to completely unavailable on Wednesday. I've decided
to build the computer earlier and run Debian 12 Bookworm Testing on it
right now and have it roll into Stable when it releases. So I've been
buying parts the last few weeks, when they were available and the prices
were good.

For the first time in 20 years I've decided to do an all-AMD build instead
of the usual Intel / nVidia I go for. The reasons are:

- As of the time of writing, AMD graphics cards have better open source
  support than nVidia cards (driver is in the Linux kernel, and they
  support the Wayland protocol better).
- Intel only has CPU's with 8 Performance cores. The rest are Efficiency
  cores, which are not useful when testing chess engines. In contrast, AMD
  has CPU's that have 6-16 normal (Performance) cores. (They also have
  CPU's with more cores, as has Intel, but those are outside my budget.)
- AMD CPU's can be limited with regard to power draw. Reviews and tests
  indicate that limiting the power draw of an AMD CPU will cost less than
  2% performance. Effectively it can reduce power consumption by 40-50% ro
  2% performance loss. I'm willing to make that trade.
- AMD has a history of supporting their sockets longer than Intel. If
  they'd release an AM5 CPU in 5 years that is a real improvement over what
  I'm buying now, I might just upgrade the CPU only.

After reading many reviews, I decided on getting an AMD 7950X, because it
will decrease both my power consumption and testing time compared to my
current system. The stats come down to:

- Testing time is down from 8 hours to 2 hours
- Testing time saved is 75%
- 8h x 95W - 2h x 140W = 760 - 240 = 520W saved
- Power draw for a test run is down by 70%

The computer I'm building should easily last me 7-10 years again, maybe
with a CPU-upgrade in between. If I'm still developing Rustic 10 years from
now (which I doubt, because I'll probably have declared it 'finished'
before that) I'll see what needs to be done.
