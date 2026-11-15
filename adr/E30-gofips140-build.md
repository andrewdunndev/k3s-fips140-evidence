# E30: `make` passes `GOFIPS140` to the build

**Claim.** On `main`, `GOFIPS140=v1.0.0 make binary` produces an ordinary binary: the variable never reaches `go build`. With commit 5b054838c5 (`Pass GOFIPS140 through to the build`) it does, and the binary carries the FIPS module, the `fips140v1.0` tag and `DefaultGODEBUG=fips140=on`. With the variable unset the commit's build is byte-identical to the base's. The FIPS binary starts and reaches Ready with `GODEBUG` unset.

**Build.** Each commit exported with `git archive` and built through `make binary` on an Ubuntu 24.04 cloud VM, Go 1.26.10. G1 is 5b054838c5 with `GOFIPS140=v1.0.0`. G2 is 5b054838c5 with it unset. G0 is the base, c55a7df0fc (origin/main), with `GOFIPS140=v1.0.0` (control).

**Commands.**

```
GOFIPS140=v1.0.0 make binary      # G1, and G0 on the base
make binary                       # G2
go version -m dist/artifacts/k3s | grep -E 'GOFIPS140|DefaultGODEBUG|fips140v'
k3s server                        # G1 binary, GODEBUG unset, read /proc/<pid>/environ
```

**Log lines.**

`go version -m, G1`

```
	build	-tags=urfave_cli_no_docs,fips140v1.0
	build	DefaultGODEBUG=fips140=on
	build	GOFIPS140=v1.0.0-c2097c7c
```

`go version -m, G2 and G0 (control)`

```
	build	-tags=urfave_cli_no_docs
```

`build command echoed by make`

```
G1  --build-arg="GOFIPS140=v1.0.0"
G2  --build-arg="GOFIPS140="
G0  (no GOFIPS140 argument)
```

`binary sha256 prefix`

```
G2  57fcedac8197b127
G0  57fcedac8197b127      # identical size, 82165922 bytes
```

`G1 run`

```
GODEBUG in environ: 0
READY after ~30s
```

**Note.** No fatal or panic line in the G1 server log. The G1 run disables traefik, metrics-server, local-storage and servicelb to keep the start short.
