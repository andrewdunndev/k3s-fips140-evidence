# E10: kubectl exec over websocket and SPDY fallback

**Claim.** Against an exempt `only` server the websocket upgrade fails at the kubelet-to-containerd hop and kubectl falls back to SPDY, so exec works. kubectl running under `only` panics in gorilla `computeAcceptKey` when it takes the websocket path.

**Build.** k3s `main` (v1.37.2), exempt build, GOFIPS140=v1.0.0, one server and an agent, client `kubectl -v=8` with GODEBUG `off` or `only` and `KUBECTL_REMOTE_COMMAND_WEBSOCKETS=true`. Server mode `only` or `on` as labelled.

**Commands.**

```
GODEBUG=fips140=<off|only> KUBECTL_REMOTE_COMMAND_WEBSOCKETS=true kubectl exec web -v=8 -- echo hi
```

**Log lines.**

`exempt server, only: both clients exec rc 0, SPDY fallback`

```
== client=off exec_rc=0
I1114 10:51:45.964370  145771 fallback.go:56] "RemoteCommand fallback" err="unable to upgrade streaming request: "
== client=only exec_rc=0
I1114 10:51:53.156824  146261 fallback.go:56] "RemoteCommand fallback" err="unable to upgrade streaming request: "
```

`exempt server, only: agent log, the failing hop`

```
E1114 10:51:53.035671      48 upgradeaware.go:428] "Error proxying data from client to backend" err="readfrom tcp <ip>-><ip>: websocket: close 1006 (abnormal closure): unexpected EOF"
```

`exempt server, on: client under only`

```
== client=off exec_rc=0
== client=only exec_rc=2
```

`exempt server, on: client panic`

```
panic: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
github.com/gorilla/websocket.computeAcceptKey({0xad0d2eb2d08?, 0x1b89a89?})
```

**Note.** The cause of the websocket failure at the kubelet hop is not identified.
