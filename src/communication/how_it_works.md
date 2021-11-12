# How it works

When you have reached this point in the writing of your engine, you're very
close to making it play its first game under a user interface. Assuming you
have extensively tested the engine while writing it, making sure there are
no bugs in the board representation, the move generator and move handling,
then the only thing left is to set up the communication between the engine
and user interface.

We just discussed the two main protocols used in the chess engine world:
UCI and XBoard. First, we design a way to get commands into the engine and
receive replies from it; then we decide if we want to use the UCI or XBoard
protocol, or both. That is the most tedious part of the entire engine,
because it just comes down to: "If you receive command X, you do action Y
and send reply Z."

I'm sure you're eager to get your ~~monstrosity~~ ... _awesome creation_
... playing against either yourself or other engines, so let's get going.
First we remember the diagram from the concept page:

![](../diagrams/concept.svg)

Both the engine and the user interface are a separate piece of software,
running completely independently, so we have to get commands from the user
interface into the engine, and replies from the engine into the user
interface. This is done through "standard input/output", often called _stdin_
and _stdout_. You have been working with stdin and stdout for a long time
already during the writing of the engine. When you type a command on the
command line, you're putting text into stdin; if the command line prints
anything, that text comes out through _stdout_. (Or "standard error",
_stderr_; which could also be something else, such as a printer, but
nowadays, stdout and stderr are normally just the computer's display.)

At the very beginning of this journey, the only code you had was probably
something like this:

```js
const NAME: &str = "Engine";
const VERSION: &str = "0.1";

fn main() {
    println!("{} {}", NAME, VERSION);
}
```

The output from "println" is sent to stdout, which normally is the screen.
If you also have functions to read text typed into the engine (for testing,
for example), then this text comes in through stdin, which normally is the
keyboard.

As the user interface starts the engine, it uses a construct called a
_pipe_: that is the red two-way arrow in the diagram above. The GUI
connects its own stdout on one end of the pipe and stdin from the engine is
connected to the other end. Thus, anything the GUI prints to its own stdout
goes through the pipe, right into the stdin of the engine, where the engine
can then read it. Obviously, the engine's stdout is connected to the GUI's
stdin through the same pipe, so any text the engine prints is received by
the GUI.

If you have ever used a command line, especially on Linux, you may have
actually seen a pipe and may not have known the name. For example, let's
see what files are in a directory:

```bash
$ ls
```

A list of files is printed on screen. Now we want to know if a file called
"stuff.txt" is among those files. Instead of reading the entire list, we
can do this, by searching for it with _grep_:

```bash
$ ls | grep -i "stuff.txt"
```

Note the _pipe symbol_ | in between the two commands. This means: connect
the stdout from "ls" to the stdin from "grep". This makes grep receive the
printed list of files, so it can search through it. (stdout from "grep" is
not connected to the stdin from "ls"; it is still connected to the display,
or you would never be able to see the result.)

Now that we know how the communication works, we can design a way to set
this up. This will be the next chapter.