# E02: tunnel SHA-1 panic without the exemption, suites pass with it

**Claim.** With SHA-1 replaced everywhere and no `WithoutEnforcement`, the `/v1-k3s/connect` upgrade panics under `only`; keeping the websocket SHA-1 inside `WithoutEnforcement` lets the k3s systemd-node suites pass under `only`.

**Build.** k3s `main` (v1.37.2) plus every open PR. "patched": SHA-256 replaces each SHA-1, no exemptions. "exempt": the websocket accept key, etcd IDs, wrangler label and dynamiclistener fingerprint stay SHA-1 inside `fips140.WithoutEnforcement` (library patches). GOFIPS140=v1.0.0, Go 1.26, GODEBUG=fips140=only. Ubuntu 24.04 VM.

**Commands.**

```
# k3s tests/docker suites, run unchanged on a systemd node, binary and GODEBUG set by a wrapper
go test ./tests/docker/etcd/... -ginkgo.v   # patched binary, fips140=only
go test ./tests/docker/etcd/... -ginkgo.v   # exempt binary, fips140=only
# same for hardened, secretsencryption, svcpoliciesandfirewall, dualstack
```

**Log lines.**

`patched, only, etcd suite`

```
    time="2026-11-14T07:24:01Z" level=info msg="Cluster-Http-Server 2026/11/14 07:24:01 http: panic serving <ip>: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode"
    time="2026-11-14T07:24:01Z" level=info msg="\t/go/src/github.com/k3s-io/k3s/pkg/daemons/control/tunnel.go:97 +0x165"
Ran 2 of 11 Specs in 37.333 seconds
FAIL! -- 1 Passed | 1 Failed | 0 Pending | 9 Skipped
```

`exempt, only, etcd suite`

```
Ran 11 of 11 Specs in 287.255 seconds
SUCCESS! -- 11 Passed | 0 Failed | 0 Pending | 0 Skipped
```

`exempt, only, hardened suite`

```
Ran 4 of 4 Specs in 92.558 seconds
SUCCESS! -- 4 Passed | 0 Failed | 0 Pending | 0 Skipped
```

`exempt, only, secretsencryption suite`

```
Ran 20 of 20 Specs in 388.654 seconds
SUCCESS! -- 20 Passed | 0 Failed | 0 Pending | 0 Skipped
```

`exempt, only, dualstack suite`

```
Ran 16 of 16 Specs in 332.123 seconds
SUCCESS! -- 16 Passed | 0 Failed | 0 Pending | 0 Skipped
```

**Note.** The patched etcd run stops after two specs; the panic line is the first failure. The token suite passes 9 of 10 under exempt `only`; the failing spec rotates the token to `1234`, which `only` refuses (14-byte minimum).
