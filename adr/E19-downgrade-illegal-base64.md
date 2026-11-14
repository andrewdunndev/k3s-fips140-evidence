# E19: stock server on data an `on` server rewrote

**Claim.** After any server has run under `on`, a stock server that must read the datastore itself fails with `illegal base64 data at input byte 32`; a joiner with live peers is Ready; clusters that only ran with the mode off downgrade fine.

**Build.** Stock k3s `main` (v1.37.2) and stock v1.36.6 for the downgrade; series build with GOFIPS140=v1.0.0 for `on`. Three embedded-etcd servers in containers, or one SQLite server, on an Ubuntu 24.04 VM.

**Commands.**

```
# 1) stock, GODEBUG=fips140=off: start 3 servers, then stop each
# 2) restart each server on the series binary, GODEBUG=fips140=on (key rewritten)
# 3) restart one server on the stock binary, GODEBUG=fips140=off, count "illegal base64" in its log
```

**Log lines.**

`cells: a = joiner with peers up, b = first server, c = joiner with peers down, d = same as b on v1.36.6, e and g = never ran on, f and h = single SQLite server`

```
CELL 2.a joiner-peers-up srv3 bin=s/off outcome=READY illegal_base64_lines=0
CELL 2.b initial-no-join-peers-up srv1 bin=s/off outcome=ILLEGAL illegal_base64_lines=1
CELL 2.c joiner-peers-down srv3 bin=s/off outcome=ILLEGAL illegal_base64_lines=1
CELL 2.d initial-no-join-peers-up-v1.36 srv1 bin=s136/off outcome=ILLEGAL illegal_base64_lines=1
CELL 2.e never-on-initial-peers-up srv1 bin=s/off outcome=READY illegal_base64_lines=0
CELL 2.f single-sqlite x/on then s/off tokenpath=flag first=READY second=ILLEGAL illegal_base64_lines=1
CELL 2.g single-sqlite x/off then s/off tokenpath=flag first=READY second=READY illegal_base64_lines=0
CELL 2.h single-sqlite x/on then s/off tokenpath=file first=READY second=ILLEGAL illegal_base64_lines=1
```

`2.b stock server restart`

```
time="2026-11-14T14:55:45Z" level=info msg="Starting k3s v1.37.2+k3s- ()"
time="2026-11-14T14:55:45Z" level=fatal msg="Error: preparing server: failed to bootstrap cluster data: failed to reconcile with local datastore: illegal base64 data at input byte 32"
```

`2.d v1.36.6 stock restart`

```
time="2026-11-14T14:59:04Z" level=info msg="Starting k3s v1.36.6+k3s1 (fa45d6a5)"
time="2026-11-14T14:59:04Z" level=fatal msg="Error: preparing server: failed to bootstrap cluster data: failed to reconcile with local datastore: illegal base64 data at input byte 32"
```

**Note.** The `v1.37.2+k3s-` version string is a local build label.
