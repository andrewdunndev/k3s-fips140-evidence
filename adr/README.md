# Evidence for the Go FIPS 140-3 mode ADR for k3s

Each file holds one claim from the ADR, the build, the commands, and the log lines that show the result, with the control's failing line where one exists. Lines are copied from the run logs; only machine identifiers are removed. The numbers in the file names are labels local to this pack.

## How the runs were made

- Test VMs on a public cloud, Ubuntu 24.04 and RHEL 10, each a cloud VM of a general-purpose class. The runs excerpted here are on Ubuntu 24.04; clusters ran in containers or as host processes under systemd.
- Every build sets `GOFIPS140=v1.0.0` (Go 1.26). The mode is set with `GODEBUG=fips140=off|on|only`.
- "Series" is k3s with the ADR's pull requests applied. "Stock" is the branch or release as is, with its runc fixed where the control needs a running pod (k3s-io/k3s#14980). "Patched" replaces each SHA-1 with SHA-256 and has no exemption. "Exempt" keeps the protocol SHA-1 inside `fips140.WithoutEnforcement`.
- Suites are the k3s `tests/docker` and `tests/e2e` suites, run unchanged.

## Claim to file

| ADR claim | File |
|---|---|
| `only` stops on the node password salt; `on` and `off` read stock hashes; deleting the secret regenerates a 16-byte salt | E14 |
| Under `only`, a legacy node password returns an error naming the remedy, with no panic | E20 |
| The websocket accept key's SHA-1 must stay SHA-1: replacing it breaks the tunnel upgrade | E02, E07 |
| etcd computes member IDs with SHA-1; embedded etcd needs the wrap | E03 |
| `kubectl` under `only` panics on the websocket accept key; exec works over SPDY fallback | E10, E25 |
| Datastores under `on` and `only`: PostgreSQL, MySQL, MariaDB; unsupported under `only`: PostgreSQL `md5`, MySQL `mysql_native_password`, MariaDB | E05, E24 |
| `wireguard-native` works under `on`, stops on X25519 under `only` | E24 |
| Snapshot restore works under `only` | E25 |
| On release branches a short token is refused at `token rotate` under `only` | E22 |
| Upgrade every server with the mode off, then set `on` | E18 |
| A server from an earlier release that reads the rewritten data fails with `illegal base64` | E19 |
| 1.34 to 1.37 apply the series | E21, E22 |
| `Test_DockerFIPS` (7 specs) and `Test_DockerFIPSMigration` (8 specs) pass on the series and fail on stock | E26 |

## Claims with no log excerpt here

- runc fails the module's integrity check on `main`: shown in the runc pull request, not in these logs.
- release-1.33 pins Go 1.25, which lacks `WithoutEnforcement`; SPDY is deprecated with no removal scheduled (KEP-4006): from `go.mod` and the Go and Kubernetes release notes.
- Labels and fingerprints unchanged across modes, the bootstrap rewrite's compare-and-swap, and the call-site wrap locations: from code and from run summaries whose logs record lengths, not values.
