# E31: external etcd 3.5 with TLS under `on` and `only`

**Claim.** A k3s server and agent on a 3-member external etcd (v3.5.21, TLS) work under `off`, `on` and `only` on a fresh datastore: 2 nodes Ready, a pod Running, `kubectl exec` ok, a marker key written in etcd. The stock build fails the same pod and exec cells with the mode off.

**Build.** k3s with the series (x, GOFIPS140=v1.0.0) and stock (s), one server and one agent against the external etcd on an Ubuntu 24.04 cloud VM.

**Commands.**

```
k3s server --datastore-endpoint=https://<etcd members> --datastore-cafile ... # 1a stock; 1b off; 1c on; 1d only, fresh datastore
# read the marker in etcd, create a pod, kubectl exec
```

**Result table.** Copied from the run summary.

```
Cell Build GODEBUG                 Nodes Ready  Marker  Pod      Exec
1a   s     off                     2            yes     Fail     Fail
1b   x     off                     2            yes     Running  ok
1c   x     on                      2            yes     Running  ok
1d   x     only, fresh datastore   2            yes     Running  ok
```

**Note.** A cell restarted under `only` on a datastore and agent volume written under `on` (1e) is not part of the claim: the agent's node password is an old 8-byte-salt hash and the server returns `403` for it, the case E20 covers.
