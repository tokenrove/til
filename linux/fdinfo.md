So [pv] has a feature where you can watch how fast data is flowing
through another process's FD; [reading the source] I discovered it's
just looking at `/proc/PID/fdinfo/FD`; so I checked `proc(5)`
(top-tier manpage) to see what else one gets from fdinfo, and so TIL
that catting, for example, an inotify fd will give you a list of all
the inotify watches and their flags, which is really handy.

Not mentioned in the documentation is what happens when you cat an
fdinfo file corresponding to a BPF map or program, but it's actually
very useful:

```
% sudo cat /proc/11921/fdinfo/3
pos:    0
flags:  02000002
mnt_id: 13
map_type:       6
key_size:       4
value_size:     8
max_entries:    256
map_flags:      0x0
memlock:        8192
% sudo cat /proc/11921/fdinfo/4
pos:    0
flags:  02000002
mnt_id: 13
prog_type:      6
prog_jited:     1
prog_tag:       5ae0ba51548cee0f
memlock:        4096
```

Note a bunch of info about how large the map is, whether the program
got JIT'd or not, et cetera.

[pv]: http://www.ivarch.com/programs/pv.shtml
[reading the source]: https://github.com/icetee/pv/blob/8e1592bf5c4df88a1d2b7f6d1045b3a75accce15/src/pv/watchpid.c#L41
