# E14: node password salt under only

**Claim.** A stock 8-byte-salt node password hash verifies unchanged under `on`. Under `only` the servers panic in `hash.SCrypt.VerifyHash`. After the secrets are deleted under `on` they regenerate with a 16-byte salt and `only` gives 4/4 Ready.

**Build.** k3s `main` (v1.37.2), three servers on embedded etcd plus an agent in containers on an Ubuntu 24.04 VM. P0 stock `off`; later phases the series build (GOFIPS140=v1.0.0) under `on` then `only`. The hash field lengths are `$1`, salt (hex), cost, ... so `2 16 ...` is an 8-byte salt and `2 32 ...` a 16-byte salt.

**Commands.**

```
# per phase: start 3 servers and an agent, wait for 4 Ready, then
kubectl -n kube-system get secret s1.node-password.k3s -o jsonpath={.data.hash} | base64 -d   # print prefix and field lengths only
kubectl -n kube-system delete secret --all-node-passwords; restart under on; restart under only
```

**Log lines.**

`phases, readiness and one hash per phase`

```
== P0-s-off 2026-11-14T10:24:59Z
ready_nodes=4
s1.node-password.k3s prefix=$1 fieldlens=2 16 2 1 1 86
== P1-x-on 2026-11-14T10:26:16Z
ready_nodes=4
s1.node-password.k3s prefix=$1 fieldlens=2 16 2 1 1 86
== P2-x-only 2026-11-14T10:27:19Z
ready_nodes=0
== P3 (b/c) x on, delete node-password secrets, restart agent, then only
== P3-x-on-again 2026-11-14T10:31:41Z
ready_nodes=4
s1.node-password.k3s prefix=$1 fieldlens=2 16 2 1 1 86
s1.node-password.k3s prefix=$1 fieldlens=2 32 2 1 1 86
== P4-x-only-after-delete 2026-11-14T10:33:09Z
ready_nodes=4
s1.node-password.k3s prefix=$1 fieldlens=2 32 2 1 1 86
s1.node-password.k3s prefix=$1 fieldlens=2 32 2 1 1 86
```

`panic and stack frame under only`

```
panic: crypto/pbkdf2: use of salts shorter than 128 bits is not allowed in FIPS 140-only mode [recovered, repanicked]
  /go/src/github.com/k3s-io/k3s/pkg/authenticator/hash/scrypt.go:99 +0x365
```

