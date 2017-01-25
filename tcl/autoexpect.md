
[`expect`] is an immensely handy tool for scripting interactions with
programs that require a PTY.  Of course, there are clones in other
languages, but I like the original Tcl version.

TIL that `expect` ships with `autoexpect`, a tool that records your
interaction with a program and writes out an (excessively specific)
expect script to automate that interaction.  This is a handy way to
quickly capture a long interaction or just see how exactly to quote
some sequence of input or output.

Recording tools for this kind of thing are common -- for example
[Selenium] for recording browser interaction, and
[Pulover's macro creator], built on top of [AutoHotKey], for recording
Windows GUI interaction -- but the output is brittle and usually
requires some careful rewriting by hand.

[`expect`]: https://en.wikipedia.org/wiki/Expect
[Pulover's macro creator]: http://www.macrocreator.com/
[Selenium]: http://www.seleniumhq.org/
