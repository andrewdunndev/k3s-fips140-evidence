# Evidence for the Go FIPS 140-3 mode ADR for k3s

Each file holds one claim from the ADR, the build, the commands, and the log lines that show the result, with the control's failing line where one exists. Lines are copied from the run logs; only machine identifiers are removed. The numbers in the file names are labels local to this pack.

## How the runs were made

- Test VMs on a public cloud, Ubuntu 24.04 and RHEL 10, each a cloud VM of a general-purpose class. The runs excerpted here are on Ubuntu 24.04; clusters ran in containers or as host processes under systemd.
- Every build sets `GOFIPS140=v1.0.0` (Go 1.26). The mode is set with `GODEBUG=fips140=off|on|only`.
- "Series" is k3s with the ADR's pull requests applied. "Stock" is the branch or release as is, with its runc fixed where the control needs a running pod (k3s-io/k3s#14980). "Patched" replaces each SHA-1 with SHA-256 and has no exemption. "Exempt" keeps the protocol SHA-1 inside `fips140.WithoutEnforcement`.
- Suites are the k3s `tests/docker` and `tests/e2e` suites, run unchanged.

## Claim to file

Rows follow the order the ADR states them.

| ADR claim | File |
|---|---|
| Bootstrap keys derive with PBKDF2-SHA-1 (`pkg/cluster/encrypt.go:31,60`) and `only` stops on it: `crypto/pbkdf2: use of hash functions other than SHA-2 or SHA-3 is not allowed in FIPS 140-only mode` | E32 |
| Node passwords hash with an 8-byte salt, and `only` stops on it: the servers panic on the short salt; deleting the secret regenerates a 16-byte salt | E14 |
| Under `only`, a legacy node password returns an error naming the remedy, with no panic | E20 |
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
| Upgrade every server with the mode off, then set `on` | E18 |
| The bootstrap data rewrite waits until every control-plane node carries `k3s.io/bootstrap-sha256` and retries each minute | E35, E18, E19 |
| A cluster that mixes releases stays on the legacy format; a server from an earlier release that reads the rewritten data fails with `illegal base64`; there is no downgrade past `on` without a backup | E19 |
| The series applies to release-1.34 to 1.37 | E21, E22 |
| K3s's build does not pass `GOFIPS140` through today | E30 |
| First runs of `Test_DockerFIPS` and `Test_DockerFIPSMigration` on the series, and stock failing `Test_DockerFIPS` under `only` at cluster setup | E26 |
| With the series on main and release-1.36: unit tests under off, `on`, `only`; `Test_DockerFIPS` 7 of 7, `Test_DockerFIPSMigration` 8 of 8, `Test_DockerFIPSMixedVersion` 8 of 8; stock fails `Test_DockerFIPS` under `only` | GATE |

## Claims with no log excerpt here

- runc fails the module's integrity check on `main`, so no pod starts: shown in the runc pull request, not in these logs.
- release-1.33 pins Go 1.25, which lacks `WithoutEnforcement`: from `go.mod` and the Go release notes.
- SPDY is deprecated with no removal scheduled (KEP-4006): from the Kubernetes release notes.
- Labels and fingerprints unchanged across modes, the bootstrap rewrite's compare-and-swap, and the call-site wrap locations: from code and from run summaries whose logs record lengths, not values.
