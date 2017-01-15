Although this is obvious in retrospect, I was moderately surprised
when I downloaded [this year's battlecode client] to my NixOS system
and it didn't work.  For example, I thought the reference to `ld.so`
in the ELF header (as the ELF interpreter) would be relative to some
system-configured path or something, but this is not the case; the
path is [effectively part of the ABI].

On top of that, I can't just set the RPATH to one path and expect it
to find all its libraries.  All this is discussed in depth [on Sander
van der Burg's blog].

What I'm leaving here is a little hack which is not at all appropriate
for general reuse but which might speed up getting such a binary
working:

```shell
patchelf --set-interpreter \
  $(find ~/.nix-profile/ -name ld-linux-x86-64.so.2 | head -1) \
  battlecode-client-17
patchelf --set-rpath \
  $(for i in $(ldd battlecode-client-17 | \
               awk '/not found/ { print $1 } END { print "libudev.so" }'); do \
      dirname $(find /nix/store -name $i | head -1); done \
    | paste -s -d:):$(patchelf --print-rpath battlecode-client-17):. \
  battlecode-client-17
```

I'm sure I'll later learn the appropriate `nix-instantiate` magic to
do this better.  Also, the extra udev library is there because these
electron apps load `libudev` after startup with `dlopen`; upon typing
that, I realized I probably could have copied [the nix package for
Slack] to do this much more cleanly.

[this year's battlecode client]: https://www.battlecode.org/
[effectively part of the ABI]: https://sourceware.org/ml/libc-help/2013-08/msg00004.html
[on Sander van der Burg's blog]: http://sandervanderburg.blogspot.ca/2015/10/deploying-prebuilt-binary-software-with.html
[the nix package for Slack]: https://github.com/NixOS/nixpkgs/blob/master/pkgs/applications/networking/instant-messengers/slack/default.nix
