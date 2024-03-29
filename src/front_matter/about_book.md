
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [What this book is](#what-this-book-is)
- [What this book is not](#what-this-book-is-not)
- [How to use this book](#how-to-use-this-book)

<!-- /code_chunk_output -->
# What this book is

This book is the documentation for the chess engine Rustic. It describes
the design, all of the parts, how they work and how they fit together. The
chess engine is written in Rust, so this is the programming language which
will be used in this book. Code and examples will be taken directly from a
working release of the engine. This means the book contains information
about the following:

- How to design a chess engine
- Software engineering principles
- Chess engine programming and algorithms
- Code examples taken straight from the engine
- Chess knowledge and how to use this in an engine
- The Rust programming language, where necessary
- Pointers on how to get the code to compile
- Testing methodology

When designing and writing your own engine, either in Rust or a different
programming language, this book provides _a_ way (not _the_ way, because
there are many ways to create software) on how to accomplish this. All of
the information has been gathered throughout the internet:

- Websites
- YouTube video's
- Forum posts
- Newsgroup posts written 25 years ago
- Computer science blogs and articles
- And my own knowledge as a software engineer

Listing _all_ of the sources I used would be quite impossible. This would
become a gargantuan list, not even taking into account the fact that
internet is volatile. A source found in 2019 may not be available in 2021,
or the information may have changed. Some people who are mentioned in the
credits of Rustic and on the "Further study" page also provide lots of
information on chess programming. You may want to check out their work.

# What this book is not

If you are hoping for a step-by-step tutorial, you are probably going to be
disappointed. This book is not laid out in such a way that you can just
follow it from beginning to end and end up with a perfectly working chess
engine. This would _maybe_ be possible if you are using Rust, but when
using other programming languages different design decisions may need to be
made and implementation details may need to be changed. This is because not
all languages support the same features and similar features may not work
the same way.

The goal of the book is to consolidate enough information in one place to
be able to design and write a chess engine from scratch in any modern
programming language. The book should provide you with a good foundation in
chess engine design and programming. You will be able to research and
implement ideas that are not (yet) described in these pages... or even,
try and implement your own ideas.

Because I'm not the most evil guy ever, I'll help you along in case you
_were_ looking for a step-by-step tutorial. Please note that the links
below were available in November 2021; this may have changed since then.

Step-by-step tutorials:

- [Programming a chess engine in C (Richard Allbert / Bluefever
  Software)](https://www.youtube.com/watch?v=bGAfaepBco4&list=PLZ1QII7yudbc-Ky058TEaOstZHVbT-2hg)
  <br />
- [Bitboard chess engine in C (Maksim
  Korzh)](https://www.youtube.com/watch?v=QUNP-UjujBM&list=PLmN0neTso3Jxh8ZIylk74JpwfiWNI76Cs)
  <br />
- [Chess engine in Java (Software Architecture &
  Design)](https://www.youtube.com/watch?v=h8fSdSUKttk&list=PLOJzCFLZdG4zk5d-1_ah2B4kqZSeIlWtt)
  <br />
- [Java chess engine tutorial (Logic
  Crazy)](https://www.youtube.com/watch?v=a-2uSg4Kvb0&list=PLQV5mozTHmaffB0rBsD6m9VN1azgo5wXl)
  
# How to use this book

The best way to use this book is to read it once from beginning to end, to
get a general overview about the topics involved in creating a chess
engine. It does not matter if you don't immediately understand everything.
After you read the book, you can start designing and writing your own
engine. This can be as simple as setting up the _main()_ function, defining
two constants for your engine's name and version number and printing those
to the screen. At that point your engine has reached the "Hello, world!"
stage and you go from there. You can refer back to this book time and again
when you reach each new topic.

There is also a "Further study" page in the book, referring to some good
material to review. Parts of the tutorials above are no exception; even if
you _don't_ want to create a chess engine by following a tutorial, those
videos can still be of help. Just find the ones about the topic you are
researching in the playlist.

# Code consistency: book vs. engine

Part of this book was written during the development of Rustic Alpha 2 and
3, the rest during the development of Rustic 4 and later. This means that
there will be some differences between the code in the book and the code
you may see in any one version of the engine. The engine is constantly
changing, so it's almost impossible to keep this book updated with the
latest version. The plan at this point is to target the book at
Rustic 4 and higher. This means that some things as described in this book
were implemented in Alpha 3 and before, but they have been removed from the
engine since version 4 or later. One example is [material
counting](../evaluation/material.md) and [single-value
PSQT's](../evaluation/psqt.md), which were part of Rustic Alpha 3, but they
have been replaced by a [Tapered Evaluation](../evaluation/tapering.md) in
version 4. However, it should still be possible to understand the basics of
these concepts so you can implement them. Similar things will happen with
other concepts throughout the book.