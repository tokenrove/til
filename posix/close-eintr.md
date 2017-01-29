# `close(2)` and `EINTR`

I was reading the source to Factor's unix module when I noticed this:

```factor
             ! Bug #908
             ! Allow EINTR for close(2)
             errno EINTR = [
```

I remembered handling error returns from `close` is a classic problem, but what happens in the case of `EINTR`?

The end of the `close(2)` manpage under Linux has a couple of "POSIX will fix this RSN" bits.  The `EINTR` case is interesting enough to quote wholesale, although it's long:

> The EINTR error is a somewhat special case.  Regarding the EINTR error, POSIX.1-2013 says:

> If close() is interrupted by a signal that is to be caught, it shall return -1 with errno  set  to  EINTR and the state of fildes is unspecified. 

> This permits the behavior that occurs on Linux and many other implementations, where, as with other errors that may be reported by close(), the file descriptor is guaranteed to be closed.  However, it also permits another possibility: that the implementation returns an EINTR error and keeps the file descriptor open.  (According to its documentation, HP-UX's close() does this.)  The caller must then once more use close() to close the file descriptor, to avoid file descriptor leaks.  This divergence in implementation behaviors provides a difficult hurdle for portable applications, since on many implementations, close() must not be called again after an EINTR error, and on at least one, close() must be called again.  There are plans to address this conundrum for the next major release of the POSIX.1 standard.

Why does `close` block?  [Rich Felker writes:](https://ewontfix.com/4/)

> Some genius way back thought it would be clever to overload the close function with responsibility for rewinding tape devices, and block until rewinding is complete. Never mind that there are perfectly good ways to rewind the device before closing it. If not for this historical blunder, close would never have been a blocking function, would never have been subject to interruption-by-signal, would never fail with EINTR, and would not have been specified by POSIX as a cancellation point. And this would have in turn made programming with file descriptors, signals, and thread cancellation a lot easier and less error-prone...
