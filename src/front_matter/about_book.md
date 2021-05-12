# About this book

## What this book is

This book is the documentation for the chess engine Rustic. It describes
the design of the engine, all of the parts, how they work, and how they fit
together. The chess engine is written in Rust, so that is the programming
language that will be used in this book. Code and examples will be taken
directly from a working release of the engine. This means that this book
contains information about the following:

- Chess engine programming
- Code examples taken straight from the engine
- Chess knowledge, or how to use this in an engine
- The Rust programming language, where necessary
- Pointers on how to get the code to compile
- Software engineering and design

When designing and writing your own engine, either in Rust or a different
programming language, this book provides a way (not _the_ way, because
there are many ways to create software) on how to accomplish this. All of
the information has been gathered throughout the internet:

- Websites
- YouTube video's
- Forum posts
- Newsgroup posts of 25 years ago
- Computer science blogs

Listing all the sources would be quite impossible. This would become a
gargantuan list, not even taking into account that the internet is
volatile. A source found in 2019 may not be available anymore in 2021, or
the information may have changed. Some people who are mentioned in the
credits also provide lots of information on chess programming. You may want
to check out their work.

## What this book is not

If you are hoping for a step-by-step tutorial, you are probably going to be
disappointed. This book is not laid out in such a way that you can just
follow it from beginning to end and then have a perfectly working chess
engine. That would _maybe_ be possible if you are using Rust, but with
different programming languages, different design decisions may need to be
made and implementation details may need to be changed. This is because not
all languages have the same features, and similar features don't work the
same way.

In the end, the goal is to consolidate enough information in one place to
be able to design and write a chess engine from scratch in any modern
programming language. The book should provide you with a good foundation in
chess programming and engine design. You will be able to research and
implement techniques that are not (yet) described in these pages... or
even, try and implement your own ideas.