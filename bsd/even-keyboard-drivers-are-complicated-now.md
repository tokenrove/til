A week or two ago I was having a conversation with a coworker about
[n-key rollover], how keyboards that support both PS/2 and USB, and
related topics.  He mentioned something I hadn't known about before,
which is that [USB HIDs] have two different protocols: report, and
boot.  The report protocol requires the host to parse a fairly complex
descriptor while the boot protocol has a fixed format to simplify
things for hosts that can't deal with that.  We speculated for a while
about what happens in the keyboard firmware for switching between the
two, but I didn't follow up on it.

I didn't put it together at the time, but I have persistently had
problems with my QuickFire keyboard (an affordable and compact
delivery mechanism for Cherry Blues) on DragonFly and FreeBSD, which I
never bothered to track down.

But today I once again found the QuickFire to be dropping key events
while plugged into a FreeBSD machine, and thought back to the
conversation.  Sure enough, as described in [usb_quirk(4)], I was able
to make the keyboard work much more reliably by setting
`UQ_KBD_BOOTPROTO`.

This made me wonder, though.  What did Linux's keyboard driver do
differently?  Was I getting "full use" of my keyboard's capabilities
in both OSes, or neither?

A quick check of [Linux's HID code] revealed that, though there was
certainly a bunch of code for handling report protocol generically,
there was also a lot of custom code, and although my keyboard didn't
seem to be punting to [usbkbd.c], which seems to be a purely boot
protocol driver, `lsusb -v` only showed boot protocol descriptors and
it loaded under `hid-generic`.

[FreeBSD's ukbd code] definitely tries to parse the report protocol
HID, otherwise setting the boot protocol quirk wouldn't help.  So
what's wrong?  Perhaps this keyboard actually sends some incorrect
descriptor or FreeBSD's HID parsing code has a bug.  The next step
would probably be for me to fire up [usbdump] and watch what happens,
but that's enough for me today.

(It turns out this is a known issue: [#181425].  And from the sounds
of the discussion there, both Linux and FreeBSD don't handle the full
scope of possible descriptors.)

It was a little terrifying to discover that [you could deadlock in the
keyboard driver].  I didn't check the USB implementations of other
BSDs, but I think they all come from the NetBSD implementation.

Also of interest: there are open source USB keyboard firmwares, and we
can see the descriptors and how n-key rollover being enabled affects
what gets sent, for example in [firmware/src/USB.c in EasyAVR].

The [HID spec] is actually pretty enlightening, too.  See Appendix F
(Legacy Keyboard Implementation).

[#181425]: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=181425
[firmware/src/USB.c in EasyAVR]: https://github.com/dhowland/EasyAVR/blob/925ba5986b76a6c62aacddd2aa6ce1baa445383b/firmware/src/USB.c
[FreeBSD's ukbd code]: https://github.com/freebsd/freebsd/blob/37b3e9269d80471e7166706bc034099030387d36/sys/dev/usb/input/ukbd.c
[HID spec]: http://www.usb.org/developers/hidpage/HID1_11.pdf
[Linux's HID code]: https://github.com/torvalds/linux/tree/master/drivers/hid
[n-key rollover]: https://en.wikipedia.org/wiki/Rollover_%28key%29#n-key_rollover
[USB HIDs]: https://en.wikipedia.org/wiki/Human_interface_device
[usb_quirk(4)]: https://www.freebsd.org/cgi/man.cgi?query=usb_quirk&sektion=4&n=1
[usbdump]: https://www.freebsd.org/cgi/man.cgi?query=usbdump&sektion=8
[usbkbd.c]: https://github.com/torvalds/linux/blob/master/drivers/hid/usbhid/usbkbd.c
[you could deadlock in the keyboard driver]: https://github.com/freebsd/freebsd/blob/37b3e9269d80471e7166706bc034099030387d36/sys/dev/usb/input/ukbd.c#L1992-L2014
