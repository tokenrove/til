Today at work Mina observed that you can `putenv(3)` from gdb and
modify a running process's environment.  I pointed out that since
`putenv` puts the supplied pointer directly in the environment, unlike
`setenv` which copies (see ["the setenv fiasco"]), this was probably
dangerous.  But it seemed to work, and then I realized I'd never
thought about where gdb allocates temporaries when evaluating
expressions.

After a little thought, I decided it would have to be in the address
space of the program under inspection (the "inferior", in gdb's
terminology), since you can invoke functions from the inferior, and
gdb traps crashes in evaluated statements.  Could it evaluate locals
on the stack?  Would there be a stack frame?  Curtis pointed out that
since you can inspect locals with evaluation, it must happen in the
current stack frame.  So stack allocation might be possible.

But the string in `p putenv("LOL=waaat")` continued to live after gdb
was detached and the program did other things, so it wasn't on the
stack.  Inspecting the return of `getenv("LOL")`, I saw that it was on
the inferior's heap.  gdb must have allocated the string; with malloc?
Would it ever be freed?

Entering `eval.c` as a likely starting point for gdb's "C REPL"
feature, we see that [eval.c:`evaluate_subexp_standard`] calls
`value_string()`.

Above [valops.c:`value_string`] we see the following comment:

```c
/* Create a value for a string constant by allocating space in the
   inferior, copying the data into that space, and returning the
   address with type TYPE_CODE_STRING.  PTR points to the string
   constant data; LEN is number of characters.

   Note that string types are like array of char types with a lower
   bound of zero and an upper bound of LEN - 1.  Also note that the
   string may contain embedded null bytes.  */
```

This leads us to `value.c`, where allocations appear to be tracked,
have reference counts, and so on.

```c
  /* The number of references to this value.  When a value is created,
     the value chain holds a reference, so REFERENCE_COUNT is 1.  If
     release_value is called, this value is removed from the chain but
     the caller of release_value now has a reference to this value.
     The caller must arrange for a call to value_free later.  */
  int reference_count;
```

But if that's so, why does this appear to work?  I guess either gdb
leaks memory in this case, or the area does actually get reused and I
haven't done enough to trigger the bug.

Trying to simulate this under valgrind has a fun result:

```
--5953-- VALGRIND INTERNAL ERROR: Valgrind received a signal 11 (SIGSEGV) - exiting
--5953-- si_code=1;  Faulting address: 0x0;  sp: 0x802cadef0

valgrind: the 'impossible' happened:
   Killed by fatal signal
```

(Note, since you don't want to gdb valgrind itself, you need to use
something like `--vgdb-stop-at=startup` and then use the instructions
valgrind gives you to attach.)

Finally, tiptoeing around the process delicately, I did the `putenv`
and then quit without segfaulting valgrind or gdb; and with
`--leak-check=full`, we get:

```
==6105== 10 bytes in 1 blocks are definitely lost in loss record 2 of 9
==6105==    at 0x4C2D136: memalign (vg_replace_malloc.c:858)
==6105==    by 0xFFEFFFE1E: ???
==6105==    by 0x1: ???
==6105==    by 0x325B67FF: ???
==6105==    by 0x137FF: ???
==6105==    by 0x7ABC6F064852FFFF: ???
==6105==    by 0x165: ???
==6105==    by 0x325AEFFF: ???
==6105==    by 0x1FF: ???
==6105==
==6105== LEAK SUMMARY:
==6105==    definitely lost: 10 bytes in 1 blocks
==6105==    indirectly lost: 0 bytes in 0 blocks
==6105==      possibly lost: 0 bytes in 0 blocks
==6105==    still reachable: 1,751 bytes in 55 blocks
==6105==         suppressed: 0 bytes in 0 blocks
```

That's our string.

So I suspect it's not a good idea to keep the temporaries in gdb's
evaluator around.  But it's interesting to note they end up on the
inferior's heap.

["the setenv fiasco"]: http://www.club.cc.cmu.edu/~cmccabe/blog_the_setenv_fiasco.html
[eval.c:`evaluate_subexp_standard`]: https://github.com/bminor/binutils-gdb/blob/8b10b0b3e100c25322a083248c7a18bf5a1f3527/gdb/eval.c#L834-L840
[valops.c:`value_string`]: https://github.com/bminor/binutils-gdb/blob/61baf725eca99af2569262d10aca03dcde2698f6/gdb/valops.c#L1670-L1691
