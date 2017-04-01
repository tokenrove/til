[Compound literals] are a great feature of modern C, introduced in
C99.

This is useful for situations like this:

```c
rv = setsockopt(fd, SOL_SOCKET, SO_LINGER,
                &(struct linger){.l_onoff = 1, .l_linger = 0},
                sizeof(struct linger));
```

But you might disagree with my love for them if you're one of the
people who's shocked by something like this:

```c
if (0 != setsockopt(fd, SOL_SOCKET, SO_REUSEPORT, (int[]){1}, sizeof(int)))
```

Anyway, today I pushed this to a level unacceptable even to the
compiler (although it turns out clang accepts this!).  To allocate a
`max_token_len` buffer on the stack, inside this `string` type, I
wrote something like this:

```c
struct string path = {.bytes = (char[max_token_len]){}},
```

but, gcc complains, and indeed, [quoth the standard], in section
6.5.2.5:

> The type name shall specify a complete object type or an array of
> unknown size, but not a variable length array type.

and so my obscuritan impulses were luckily foiled again.

[Compound literals]: https://gcc.gnu.org/onlinedocs/gcc/Compound-Literals.html
[quoth the standard]: http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1548.pdf
