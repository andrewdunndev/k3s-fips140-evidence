# E29: agentless server under `on` and `only`

**Claim.** A `--disable-agent` server, with no node of its own, serves one joined agent under `on` and `only`: pods run on the agent, `kubectl logs` returns the pod's line, `kubectl exec` returns rc 0, a ClusterIP curl succeeds. Stock under `only` panics on SHA-1.

**Build.** k3s `main` with the series (fdf72b660e), GOFIPS140=v1.0.0, Go 1.26. "tip" is that build; "stock" is the base (c55a7df0fc) built the same way. Server as a host process on an Ubuntu 24.04 cloud VM, agent in a privileged container on the VM.

**Commands.**

```
k3s server --disable-agent                                # A.1 on, A.2 only, A.3 stock only
# check zero nodes, join one agent, two pods, logs, exec, curl a ClusterIP
```

**Log lines.**

`checks, A.1 (tip, on); A.2 (tip, only) matches`

```
server READY alive=yes
nodes_before_agent=0
agent READY
node_srv_present=0
pod-a-running rc=0
pod-b-running rc=0
kubectl-logs rc=0
kubectl-exec rc=0
clusterip-curl rc=0 ok
```

`control, A.3 (stock, only)`

```
panic: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
```

**Note.** A.2 logs one `level=error` line (`Error while syncing C...`) that is not a FIPS line; the cell's checks pass.
