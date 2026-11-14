# E07: logs and exec through the tunnel

**Claim.** Without the exemption, `kubectl logs` and `exec` fail with a 502 under `only`; with it they work and no tunnel errors appear.

**Build.** k3s `main` (v1.37.2), one server and an agent, GOFIPS140=v1.0.0, GODEBUG=fips140=only, default egress selector. "patched" has no exemption, "exempt" keeps the websocket SHA-1 inside `WithoutEnforcement`.

**Commands.**

```
k3s server; k3s agent --server ...
kubectl run bb --image=busybox -- sleep 3600
kubectl logs bb; kubectl exec bb -- echo EXEC-OK
```

**Log lines.**

`exempt, only`

```
=== A build=k3s-exempt mode=only
ready 1 after 3s
ready 2 after 9s
pod Running
logs:  rc=0
exec: EXEC-OK
server panics: 0  agent panics: 0  tunnel errs s/a: 0/0
```

`patched, only`

```
=== A build=k3s-patched mode=only
ready 1 after 3s
ready 2 after 9s
pod Running
logs: Error from server: Get "https://<ip>/containerLogs/default/bb/bb": proxy error from 127 rc=0
exec: error: unable to upgrade connection: error dialing backend: proxy error from <ip> while dialing <ip>, code 502: 502 B
server panics: 0  agent panics: 0  tunnel errs s/a: 6/4
```

**Note.** Run with `--egress-selector-mode=disabled` the patched build also passes; the failure is the tunnel path only.
