
It's fairly well-known that `stty -a` will tell you the control
characters that you can use when interacting with your terminal.  This
is a nice thing to do if you've never done it before, or are on a new
operating system.  In particular, I've noticed that while everyone
knows about `^C`, `^D`, and `^Z`, not as many know about the useful
`^\` (send SIGQUIT), `^U` (delete the entire line), `^V` (insert next
character verbatim) control characters, or the mechanism by which
these are set.

(I remember, at one of my first jobs, scrawling echoed `^H`s all over
the screen while the lead programmer stood over my shoulder.  Having
someone watch you type seems to cause a special kind of motor
sickness; even more profoundly when a broken telnet session is
involved.  Frustrated, he muttered "just use `^U`" and it remains one
of my most used chords to this day.)

I went in to check what `status` was bound to and noticed `dsusp`,
which prompted me to look at [what POSIX specifies for terminal
special characters].  I am surprised that so few are actually
specified by POSIX.  Several interesting ones aren't POSIX, and one
needs to check the `termios` or `termio` manpage, whose section varies
across systems.

I think of `status` (`^T`) as pretty well-known, being a nice BSDism
that one occasionally misses on Linux.

For some reason I don't remember ever seeing `dsusp` (`^Y`) but it
exists not only on the modern BSDs but also at least AIX, Solaris,
QNX, yet not Linux.  It seems useful -- apparently it causes the
process to suspend when it reads the character, rather than
immediately -- but I couldn't get it to work in trivial tests.

Discard (`^O`) also didn't work for me in trivial tests, but I have
vague memories of using it on SunOS in a similar way to `^U`.
Apparently it's supposed to cause input to be dropped until the next
`^O` is sent.

Reprint (`^R`) is occasionally useful, and available on all the
systems I checked.  If the terminal gets messed up and `^L` does
nothing, it's worth trying `^R` next.

It's not clear any system still has the obscure-sounding `swtch`,
which seems to be the predecessor to `susp`.  Linux `termios(3)` says:

> Used in System V to switch shells in shell layers, a predecessor to shell job control.

and Solaris's `stty(1)` says:

> Solaris does not support any of the actions implied by swtch, which was used by the sxt driver on System V release 4. Solaris allows the swtch value to be set, and prints it out if set, but it does not perform the swtch action.
>
> The job switch functionality on Solaris is actually handled by job control. susp is the correct setting for this.

It's unfortunate that the [Unix history repository] doesn't seem to
have any System V sources; it might have been interesting to see how
this differed.  (And the OpenSolaris sources only go back to 2005.)

Interestingly, it looks like OpenSolaris got `status`/SIGINFO in 2013,
as we see in `uts/common/io/ldterm.c`:

```c
					/*
					 * Consumers do not expect the ^T to be
					 * echoed out when we generate a
					 * VSTATUS.
					 */
					if (c == tp->t_modes.c_cc[VSTATUS]) {
						ldterm_dosig(q, SIGINFO, '\0',
						    M_PCSIG, FLUSHRW);
						continue;
					}
```

[what POSIX specifies for terminal special characters]: http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap11.html#tag_11_01_09
[Unix history repository]: https://github.com/dspinellis/unix-history-repo/
