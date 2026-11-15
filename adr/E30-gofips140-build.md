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

## Round 8c: the series, with pods

**Claim.** On the series tree, the commit reduces to its two Makefile lines: the build stage of the `Dockerfile` already declares `ARG GOFIPS140` and `ENV GOFIPS140=$GOFIPS140`, so the cherry-pick conflicted on that hunk (a duplicate `ARG`) and resolved to the series' lines. With the series, the FIPS binary runs a pod, `kubectl exec` and a ClusterIP service with `GODEBUG` unset. On `main` the same build reaches Ready, but every pod sandbox fails on runc's integrity check. An arm64 build carries the same settings.

**Build.** S1 is the series tip with the build commit cherry-picked, built with `GOFIPS140=v1.0.0 make binary` on an Ubuntu 24.04 cloud VM, Go 1.26.10. S0 (control) is the build commit on stock `main`, same build. The arm64 binary comes from the same `docker buildx build` invocation as `multiarch-binary` (same build args, `--target=multiarch-result`) with `--platform linux/amd64,linux/arm64`. It was built, not run.

**Commands.**

```
GOFIPS140=v1.0.0 make binary      # S1, and S0 on main
go version -m dist/artifacts/k3s
go version -m data/<hash>/bin/runc
k3s server                        # GODEBUG unset, read /proc/<pid>/environ
kubectl exec web -- hostname
curl <ClusterIP of web>
```

**Log lines.**

`go version -m, S1 k3s`

```
	build	-tags=urfave_cli_no_docs,fips140v1.0
	build	DefaultGODEBUG=fips140=on
	build	GOFIPS140=v1.0.0-c2097c7c
```

`go version -m, S1 bundled runc (S0's is identical)`

```
	build	-tags=apparmor,seccomp,selinux,netgo,osusergo,fips140v1.0
	build	DefaultGODEBUG=fips140=on
	build	GOFIPS140=v1.0.0-c2097c7c
```

`S1 run`

```
pid_environ_GODEBUG_lines=0
node Ready after 15s
pod Running after 10s
```

`S1 pod, exec and Service`

```
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          5s
web
ok-from-pod
```

`S0 control, runc under containerd and run directly with GODEBUG unset`

```
panic: fips140: verification mismatch

goroutine 1 [running]:
crypto/internal/fips140/v1.0.0-c2097c7c/check.init.0()
	crypto/internal/fips140/v1.0.0-c2097c7c/check/check.go:93 +0x31f
```

`S0 kubelet`

```
"RunPodSandbox from runtime service failed"
"Failed to create sandbox for pod"
"CreatePodSandbox for pod failed"
```

`go version -m, arm64 build (not run)`

```
	build	-tags=urfave_cli_no_docs,fips140v1.0
	build	DefaultGODEBUG=fips140=on
	build	GOARCH=arm64
	build	GOFIPS140=v1.0.0-c2097c7c
```

**Note.** The S0 node reaches Ready and its binaries carry the same three settings, runc included, so the commit's Makefile lines reach the bundled runc; the failure is runc's own integrity check, which the series addresses and `main` lacks. No fatal or panic line in either server log.
