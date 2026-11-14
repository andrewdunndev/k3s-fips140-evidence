# E03: embedded etcd needs the SHA-1 member ID

**Claim.** Under `only`, etcd computes its member ID with `sha1.Sum`. Without an exemption the first server never serves the API; with one, three servers and an agent are 4/4 Ready.

**Build.** k3s v1.36.6 (etcd 3.6, k3s fork) with the series. "patched" has no exemption, "exempt" has `WithoutEnforcement` around the member ID. GOFIPS140=v1.0.0, GODEBUG=fips140=only. Control: stock binary, `off`. Containers on an Ubuntu 24.04 VM.

**Commands.**

```
k3s server --cluster-init                     # first server
k3s server --server https://<first>:6443       # servers 2 and 3
k3s agent  --server https://<first>:6443
kubectl get nodes   # count Ready
```

**Log lines.**

`patched, only: etcd stops right after bootstrap`

```
{"level":"info","ts":"2026-11-13T21:43:14.341148Z","caller":"etcdserver/bootstrap.go:94","msg":"bootstrapping cluster"}
time="2026-11-13T21:43:14Z" level=info msg="Shutdown request received"
```

`patched, only: harness stack at the same call`

```
PANIC: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
go.etcd.io/etcd/server/v3/etcdserver/api/membership.computeMemberID({0x132aeea36000, 0x1, 0x132aeeac1d50?}, {0x7f4c49, 0xc}, 0x0)
  ~/go/pkg/mod/github.com/k3s-io/etcd/server/v3@v3.6.14-k3s3/etcdserver/api/membership/member.go:74 +0x34e
```

`patched, only: three servers and an agent`

```
== patched only <vm>:v1.36.6-k3s1 2026-11-14T03:44:44Z
ready_nodes=0
s1 running=true panics=0
s2 running=false panics=0
s3 running=false panics=0
a running=true panics=0
```

`exempt, only`

```
== exempt only <vm>:v1.36.6-k3s1 2026-11-14T03:51:17Z
ready_nodes=4
```

`control: stock, off`

```
== stock none <vm>:v1.36.6-k3s1 2026-11-14T03:36:32Z
ready_nodes=4
```

**Note.** The control containers set no GODEBUG, so with GOFIPS140=v1.0.0 they ran in the default mode, `on`. They show that three servers and an agent form a quorum, not an `off` baseline.
