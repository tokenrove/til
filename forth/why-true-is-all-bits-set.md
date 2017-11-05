Only today I noticed Kevin Porter [left the following comment] on a
jonesforth-related post:

> I wanted to make one remark about a comment in the sources: you
> mention the “strange forth convention” that “the comparison words
> should return all (binary) 1’s for TRUE and all 0’s for FALSE”.

> I think the reason for this convention is that so that the words
> AND, OR, XOR and INVERT can function both as logical operators and
> as bitwise operators.

> By comparison, with the C convention that FALSE = 0 and TRUE = 1,
> you need two sets of operators: && and &, || and |, etc..

I have this inkling that I might have encountered this idea before,
but I could no longer remember it, so hopefully putting it here will
help me remember.  There's an argument there for [Iverson brackets]
returning 0, -1 instead of 0, 1 (but it's all the same if your array
is of one-bit elements).

[left the following comment]: https://rwmj.wordpress.com/2010/08/07/jonesforth-git-repository/#comment-8268
[Iverson brackets]: https://en.wikipedia.org/wiki/Iverson_bracket
