# E25: snapshot, cluster-reset restore and kubectl exec under `only`

**Claim.** Under `only`, a snapshot and `--cluster-reset` restore returns the cluster to the snapshot state, and `kubectl exec` returns rc 0 after falling back to SPDY.

**Build.** k3s `main` (v1.37.2) with the series and call-site exemptions, GOFIPS140=v1.0.0, GODEBUG=fips140=only, one embedded-etcd server on an Ubuntu 24.04 VM.

**Commands.**

```
k3s etcd-snapshot save
# create a second configmap, then stop k3s
k3s server --cluster-reset --cluster-reset-restore-path=<snapshot>; restart without the flag
GODEBUG=fips140=only kubectl exec web -v=8 -- echo exec-ok
```

**Log lines.**

`result ledger`

```
CELL A10 exec_rc=0 stdout=[exec-ok] websocket_lines=1 spdy_lines=1 panic=[] server_panic=[]
CELL A9 phase1 server=READY pods=[Running,] snapshot_rc=0 panic_in_snapshot_log=[]
CELL A9 phase2 reset_exit=0 scan=[]
CELL A9 phase3 restart=READY ns=[test] cm_k=[before] after_cm_present=0 deployment=[web 1/1] pods=[Running,] first_error=[]
```

`exec: fallback line`

```
I1114 16:21:14.249618   88767 fallback.go:56] "RemoteCommand fallback" err="unable to upgrade streaming request: "
```

**Note.** A9 phase 3 shows `after_cm_present=0`: the configmap made after the snapshot is gone, the one before it (`cm_k=before`) is present. The cause of the SPDY fallback is not identified. An earlier attempt of the same cell is void (a leftover server held a port).
