# Evidence for the Go FIPS 140-3 mode ADR for k3s

Each file holds one claim from the ADR, the build, the commands, and the log lines that show the result, with the control's failing line where one exists. Lines are copied from the run logs; only machine identifiers are removed. The numbers in the file names are labels local to this pack.

## How the runs were made

- Test VMs on a public cloud, Ubuntu 24.04 and RHEL 10, each a cloud VM of a general-purpose class. The runs excerpted here are on Ubuntu 24.04; clusters ran in containers or as host processes under systemd.
- Every build sets `GOFIPS140=v1.0.0` (Go 1.26). The mode is set with `GODEBUG=fips140=off|on|only`.
- "Series" is k3s with the ADR's pull requests applied. "Stock" is the branch or release as is, with its runc fixed where the control needs a running pod (k3s-io/k3s#14980). "Patched" replaces each SHA-1 with SHA-256 and has no exemption. "Exempt" keeps the protocol SHA-1 inside `fips140.WithoutEnforcement`.
- The stock `tests/docker` suites in E21 run unchanged. `Test_DockerFIPS`, `Test_DockerFIPSMigration` and `Test_DockerFIPSMixedVersion` are the series' own suites. On release-1.36 the series run adapts only handler-test fixtures (GATE).

## Claim to file

Rows follow the order the ADR states them.

| ADR claim | File |
|---|---|
| Bootstrap keys derive with PBKDF2-SHA-1 (`pkg/cluster/encrypt.go:31,60`) and `only` stops on it: `crypto/pbkdf2: use of hash functions other than SHA-2 or SHA-3 is not allowed in FIPS 140-only mode` | E32 |
| Node passwords hash with an 8-byte salt, and `only` stops on it: the servers panic on the short salt; deleting the secret regenerates a 16-byte salt | E14 |
| Under `only`, a legacy node password returns an error naming the remedy, with no panic | E20 |
| Under `only`, bootstrap data written with SHA-1 is refused with a message to start once under `on`, with no panic | no excerpt here (run summary) |
| The websocket handshake's SHA-1 is fixed by the protocol: replacing it breaks the tunnel upgrade | E02, E07 |
| etcd's member ID is SHA-1; embedded etcd needs the wrap | E03 |
| The certificate fingerprint stays SHA-1 inside `WithoutEnforcement` | E02 |
| Wrangler's object set hash stays SHA-1, inside `WithoutEnforcement` | E33 |
| Under `on`, PostgreSQL, MySQL and MariaDB work | E05, E24 |
| Under `on`, SQLite, embedded etcd and external etcd work | E19, E03, E14, E31 |
| Under `on`, `wireguard-native` works; under `only` it stops on X25519 | E24 |
| `only` does not cover PostgreSQL `md5`, MySQL `mysql_native_password` or MariaDB | E05, E24 |
| `only` does not detect the `secretbox` provider | E34 |
| Snapshot restore works under `only` | E25 |
| `kubectl exec` works under `only` over SPDY fallback | E10, E25 |
| An external NATS server works under `on` and `only`; the published binary does not include embedded NATS | E27 |
| `host-gw` works under `on` and `only` | E28 |
| Agentless servers work under `on` and `only` | E29 |
| Upgrade every server with the mode off, then set `on`: the recommended order, not a measured sequence. A cluster that only ran with the mode off is untouched by it (E19 cells e and g). E18 shows the stricter direct path, servers moving from stock straight to `on` one at a time, also holds quorum | E19, E18 |
| The bootstrap data rewrite waits until every control-plane node carries `k3s.io/bootstrap-sha256` and retries each minute | E35, E18, E19 |
| A cluster that mixes releases stays on the legacy format; a server from an earlier release that reads the rewritten data fails with `illegal base64`; once `on` has rewritten the bootstrap data, an earlier release cannot read it, so a downgrade needs a backup taken before | E19 |
| The series applies to `main`, release-1.36 and release-1.37, where it was tested; release-1.34 and 1.35 were not tested | GATE (main, release-1.36), E21 (release-1.37) |
| On release branches a short token is refused at `token rotate` under `only` | E22, GATE |
| K3s's build does not pass `GOFIPS140` through today; with the pass-through, the binary carries `GOFIPS140=v1.0.0-c2097c7c` and `DefaultGODEBUG=fips140=on`, so it runs under `on` from its first start with `GODEBUG` unset; a build without the variable is unchanged; with the series, that binary runs a pod, `kubectl exec` and a ClusterIP service with `GODEBUG` unset; on `main` its pods fail runc's integrity check; an arm64 build carries the same settings | E30 |
| First runs of `Test_DockerFIPS` and `Test_DockerFIPSMigration` on the series, and stock failing `Test_DockerFIPS` under `only` at cluster setup | E26 |
| With the series on main and release-1.36: unit tests under off, `on`, `only`; `Test_DockerFIPS` 7 of 7, `Test_DockerFIPSMigration` 8 of 8, `Test_DockerFIPSMixedVersion` 8 of 8; stock fails `Test_DockerFIPS` under `only` | GATE |

## Go documentation facts

Read from https://go.dev/doc/security/fips140 and https://pkg.go.dev/crypto/fips140, not from these runs:

- `GODEBUG=fips140=on` turns the mode on at run time, and a build with `GOFIPS140=v1.0.0` links the module version that holds the validation certificate.
- A build without `GOFIPS140` uses the module packages in its own Go release, which is not the validated version.
- `only` is "a best effort mode meant for testing, assessment, and debugging".
- `fips140.WithoutEnforcement` "should be applied carefully to tightly scoped functions".

## Claims with no log excerpt here

- runc fails the module's integrity check on `main`, so no pod starts: shown in the runc pull request, not in these logs.
- release-1.33 pins Go 1.25, which lacks `WithoutEnforcement`: from `go.mod` and the Go release notes.
- SPDY is deprecated with no removal scheduled (KEP-4006): from the Kubernetes release notes.
- Labels and fingerprints unchanged across modes, the bootstrap rewrite's compare-and-swap, and the call-site wrap locations: from code and from run summaries whose logs record lengths, not values.
- A wrap covers every goroutine started inside it, so `only` does not test embedded etcd, tunnel sessions or `kubectl`; K3s's own code, the apiserver and the kubelet stay enforced: from the wrap locations in the code and the Go documentation, not from a log.
- `secrets-encrypt` checks each node before a one-way step: from code.
- etcd has not agreed to take the narrow wraps: a status of the upstream issue, not a result.
