You can produce `.core` (Core Erlang) with `erlc` by invoking with the
undocumented `+dcore` or `+to_core` options.  The former is before
some optimizations and inlining, the later is after.  See
`select_passes/2` and so on in `otp/lib/compiler/src/compile.erl` for
more options.
