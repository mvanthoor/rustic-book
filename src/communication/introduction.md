# Protocols

While we're not deeply going into the history of these protocols, it can be
said that nowadays, in 2021, the UCI-protocol is considered to be the
default. One of the reasons is that Chessbase, which is the de-facto
standard in the world of chess software, just like Photoshop is in the
graphics world, only supports UCI. (Assuming you don't want to use a UCI to
XBoard adapter to run an XBoard-engine under a Chessbase GUI.)

There is no reason why a user interface or an engine cannot have both
protocols implemented at the same time. Many actually do so. I'm not going
to get into a debate if Chessbase actually makes good software or not, or
if one protocol is better than the other. As the world basically settled on
UCI as the default protocol, my recommendation would be to choose this, if
you are going to implement only one of the two.

Rustic implements bot UCI and XBoard, but UCI is the default.

> **Stuff**
>
> The proper name for the XBoard-protocol is "CECP", which stands for
> "Chess Engine Communication Protocol." Most people do not get to know the
> protocol under that name however. CECP was first designed as part of
> XBoard, on Linux/Unix. XBoard was later ported to Windows under the name
> Winboard. Because UCI didn't even exist at that time, the protocol became
> known as the XBoard / Winboard protocol, next to CECP. So if you're also
> reading other material, you need to know that the protocols CECP, XBoard,
> and Winboard are the same thing.