# Round 1: the startup test under `fips140=only`, and `Test_DockerFIPSMigration`

**Claim.** On `main` with the series, the integration startup test passes under `GODEBUG=fips140=only` (60 of 60) and adds no measurable time. The same test on stock `main` fails at the first server start. `Test_DockerFIPSMigration` fails as it stands (it ran 4 of 8 specs, 3 passed, 1 failed) and passes 8 of 8 with two fixes: the agent's mode is set and checked after each change, and the test waits for the bootstrap rewrite before `only`.

**Build.** An Ubuntu 24.04 cloud VM, Go 1.26.10, Docker 29.1.3, one run of each arm on 2026-11-16. The series tip is fdf72b660e, which is the ADR series plus a test-only `go.mod` replace that points at the unmerged library fixes. Stock is `main` at c55a7df0fc49. Both k3s binaries came from `GOFIPS140=v1.0.0 make binary`. That is a plain build: `make` does not pass `GOFIPS140` through (see [E30](E30-gofips140-build.md)), and `go version -m` shows no `GOFIPS140` setting, no `fips140v1.0` tag and no `DefaultGODEBUG` on either ([binary-buildinfo.txt](round1/binary-buildinfo.txt)). The integration test processes ran with `GOFIPS140` unset. Only the Docker test binary, compiled by `GOFIPS140=v1.0.0 go test`, carries it. Under `GODEBUG=fips140=only` the k3s server runs the Go release's module in enforcement mode, not a build pinned to the validated version.

**Commands.**

```
GOFIPS140=v1.0.0 make binary
cd tests/integration/startup
sudo -E env "PATH=$PATH" [GODEBUG=fips140=only] go test -count=1 -timeout=45m ./... -run Integration -ginkgo.v -test.v
GOFIPS140=v1.0.0 go test -count=1 -timeout 45m -v ./tests/docker/fipsmigration -run Test_DockerFIPSMigration -args -ci
```

The harness starts k3s without setting its environment, so the server inherits `GODEBUG` from the test process. The Docker host needs `kubectl` on `PATH`; a two-line wrapper runs `k3s kubectl` from the tip build.

**Results.**

| Arm | Tree | `GODEBUG` | Result | Wall time | Log |
|---|---|---|---|---|---|
| B (control) | series tip | unset | PASS, 60 of 60 | 662 s | [B.log](round1/B.log), [k3s log](round1/B.k3s-log.txt) |
| A | series tip | `fips140=only` | PASS, 60 of 60, no panic | 594 s | [A.log](round1/A.log), [k3s log](round1/A.k3s-log.txt) |
| C (control) | stock `main` | `fips140=only` | FAIL, 1 passed, 1 failed, 58 skipped | 132 s | [C.log](round1/C.log), [k3s log](round1/C.k3s-log.txt) |
| D0 | tip, migration test as it stands | Docker env sets the mode | FAIL, 4 of 8 ran, 3 passed, 1 failed | 162 s | [D0.log](round1/D0.log) |
| D3 | tip, migration test with the fix | same | PASS, 8 of 8 | 157 s | [D3.log](round1/D3.log), [patch](round1/D3-fipsmigration-test.patch) |

Wall time includes the `go test` compile. A ran 68 s faster than B here and 5 s faster on an earlier run the same day, so the difference is noise and the mode adds no measured time. The Docker migration test is under three minutes plus the binary build every Docker suite already pays.

**Arm C.** The first spec, `has the default pods deployed` (`startup_int_test.go:42`), times out at 120 s and the ordered suite skips the other 58. The server log ([C.k3s-log.txt](round1/C.k3s-log.txt), line 36):

```
panic: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode
crypto/sha1.Sum                                  go/src/crypto/sha1/sha1.go:278
github.com/rancher/dynamiclistener/factory.(*TLS).generateCert   factory/gen.go:252
```

That is the certificate SubjectKeyId SHA-1 in dynamiclistener ([dynamiclistener#390](https://github.com/rancher/dynamiclistener/pull/390)), which the series replaces. The test stops at the first panic on stock, so later gaps are not exercised there.

**Arm D0.** The agent never ran under the mode (`setMode` wrote `GODEBUG` only to the server's env file), the test never checked the mode after `on`, and the bootstrap rewrite is deferred until every control-plane node carries its annotation and retries about once a minute. The suite moved to `only` as soon as the node was Ready, and the server stopped ([D0.log](round1/D0.log)):

```
level=fatal msg="Error: preparing server: failed to bootstrap cluster data: bootstrap data was written with SHA-1 and fips140=only forbids reading it, start once with GODEBUG=fips140=on to migrate it"
```

An earlier run of the same code on the same tip passed 8 of 8, so the test depends on timing.

**Arm D3.** The patch adds `expectMode(mode)`, which reads `GODEBUG` from `/proc/<pid>/environ` of the server (`k3s`) and the agent (`k3s-agent`) after `on`, `only`, the rejoin and the return to `on`. It writes the agent's env file in `setMode`, and waits up to 180 s for `migrated bootstrap key` in the server journal before leaving `on`. That wait is the assertion that the migration happened, and it closes the race.

**Limits.** The environment variable shows the configured mode, not `fips140.Enabled()` inside the process. The runtime evidence is the named-remedy journal line under `only` and arm C's panic. The integration arms ran on a plain build, which is the CI default; a `GOFIPS140` build of the startup test was not run.
