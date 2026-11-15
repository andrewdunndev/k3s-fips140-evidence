# E28: `host-gw` flannel backend under `on` and `only`

**Claim.** With `--flannel-backend=host-gw`, two nodes come up Ready under `on` and `only`: the backend is `host-gw` on both, the route to the other node's pod CIDR is installed, a pod runs on each node, `kubectl exec` returns rc 0, a cross-node ping and a ClusterIP curl succeed. Stock under `only` panics on SHA-1.

**Build.** k3s `main` with the series (fdf72b660e), GOFIPS140=v1.0.0, Go 1.26. "tip" is that build; "stock" is the base (c55a7df0fc) built the same way. Server as a host process on an Ubuntu 24.04 cloud VM, agent in a privileged container on the VM.

**Commands.**

```
k3s server --flannel-backend=host-gw                      # H.1 on, H.2 only, H.3 stock only
k3s agent in the container, same backend
# pod on each node, exec, ping across nodes, curl a ClusterIP
```

**Log lines.**

`checks, H.1 (tip, on); H.2 (tip, only) matches`

```
server READY alive=yes
agent READY
backend-type srv=host-gw
backend-type agt=host-gw
route 10.42.1.0/24 via 172.30.0.2 dev r8br
pod-srv-running rc=0
pod-agt-running rc=0
pod-exec rc=0
cross-node-ping rc=0
clusterip-curl rc=0 ok
```

`control, H.3 (stock, only)`

```
panic: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
```

**Note.** H.2 logs one `level=error` line (`Error while syncing C...`) that is not a FIPS line; the cell's checks pass.
