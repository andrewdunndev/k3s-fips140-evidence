# E27: NATS as the datastore, external and embedded, under `on` and `only`

**Claim.** An external NATS server (`nats:2.11-alpine -js`) works under `on` and `only`: the node is Ready, a pod runs, `kubectl exec` returns rc 0. Embedded NATS (the `nats` build tag, `--datastore-endpoint=nats://`) does the same. Stock under `only` stops: at the node password salt with external NATS, at SHA-1 with embedded NATS. The default binary has no embedded NATS; adding the tag took the binary from 82276514 to 85209250 bytes, over the 80 MiB limit the Dockerfile enforces (83886080).

**Build.** k3s `main` with the series (fdf72b660e), GOFIPS140=v1.0.0, Go 1.26. "tip" is that build; "stock" is the base (c55a7df0fc) built the same way. Server as a host process on an Ubuntu 24.04 cloud VM, agent in a privileged container on the VM. The embedded cells use tip and stock rebuilt with `TAGS+=nats`, with the size check removed from the Dockerfile for that build only.

**Commands.**

```
k3s server --datastore-endpoint=nats://127.0.0.1:4223     # N: external server; NE: embedded
# wait for Ready, create a pod, kubectl exec into it
```

**Log lines.**

`checks, N.1 (tip, on); N.2 (tip, only) is identical`

```
server READY alive=yes
pod-running rc=0
pod-exec rc=0
```

`checks, NE.1 (embedded, tip, on); NE.2 is identical`

```
server READY alive=yes
pod-running rc=0
pod-exec rc=0
```

`control, N.3 (stock, only, external NATS)`

```
level=fatal msg="Error: server panicked: crypto/pbkdf2: use of salts shorter than 128 bits is not allowed in FIPS 140-only mode"
```

`control, NE.3 (stock + nats tag, only)`

```
panic: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
```

**Note.** Every arm greps both logs for a FIPS panic or `not allowed`; the six tip arms have none. Two earlier attempts failed on the harness (pods created before the service account existed; embedded NATS state surviving between cells), not on k3s.
