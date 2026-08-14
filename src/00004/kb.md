# Firefox: Allow RFC 1918 private addresses in TRR responses

In `about:config` set

```
network.trr.allow-rfc1918 = false
```

By default Firefox does not allow [RFC 1918][rfc1918] private addresses in
[Trusted Recursive Resolver][trr] (TRR) responses.  The tricky part is that it
does not actually tell the user that this is disallowed, instead it surfaces a
normal DNS resolution error, which is confusing to say the least.

I assume that this only happens in conjuction with `network.trr.mode = 3`, i.e.,
no fallback to the system resolver is allowed, but if TRR isn't enforced, what
is the point anyway?

[rfc1918]: https://datatracker.ietf.org/doc/html/rfc1918
[trr]: https://wiki.mozilla.org/Trusted_Recursive_Resolver
