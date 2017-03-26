Someone, who was being lazy, asked me what happened when they did
`sizeof` on a [variable-length array].  (BTW, `sizeof` is, sometimes,
the most wonderful thing in C, and if you've only been applying it to
types and not expressions, you're missing out!)

My assumption was that this would fail at compile time, because the
result isn't a constant like other uses of `sizeof`.

[C11] says:

> If the type of the operand is a variable length array type, the
> operand is evaluated; [...]

So, the semantics are totally different than what you've come to
expect, all so you can avoid repeating whatever expression you put in
the VLA's declaration.

This reinforces my contention that `sizeof` is the strangest operator
in C.

(Condensed) example from [C11]:

``` c
size_t fsize3(int n)
{
    char b[n+3];       // VLA
    return sizeof b;   // execution time sizeof
}

[...]
    size_t size = fsize3(10); // returns 13
```


[C11]: http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1548.pdf
[variable-length array]: https://gcc.gnu.org/onlinedocs/gcc/Variable-Length.html
