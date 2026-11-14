# E24: datastores and wireguard-native under `on` and `only`

**Claim.** Under `on` and `only`, PostgreSQL and MySQL start with TLS in use. Under `only`, MariaDB stops on `crypto/sha1` and `wireguard-native` stops on X25519. `wireguard-native` runs under `on`.

**Build.** k3s `main` (v1.37.2) with the series and call-site exemptions, GOFIPS140=v1.0.0, one server per cell on an Ubuntu 24.04 VM, databases on the same VM.

**Commands.**

```
k3s server --datastore-endpoint=postgres://<user>@localhost:5432/k3s?sslmode=require   # GODEBUG=fips140=only
k3s server --datastore-endpoint=mysql://<user>@tcp(localhost:3306)/k3s --datastore-cafile=<ca>
k3s server --flannel-backend=wireguard-native   # on, then only
```

**Log lines.**

`PostgreSQL, only: server reached "up and running"`

```
time="2026-11-14T16:13:01Z" level=info msg="k3s is up and running"
```

`MySQL, only`

```
time="2026-11-14T16:15:03Z" level=info msg="k3s is up and running"
```

`MariaDB, only: fatal at startup`

```
time="2026-11-14T16:18:43Z" level=fatal msg="Error: server panicked: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode"
```

`wireguard-native, on`

```
time="2026-11-14T16:16:29Z" level=info msg="k3s is up and running"
```

`wireguard-native, only`

```
panic: curve25519: crypto/ecdh: use of X25519 is not allowed in FIPS 140-only mode
  /go/pkg/mod/golang.org/x/crypto@v0.54.0/curve25519/curve25519.go:42 +0x169
```

**Note.** The MariaDB call site is inferred from the Go MySQL driver, whose `mysql_native_password` scramble uses SHA-1; no stack trace was captured in this run. A stack-level trace with `crypto/sha1` appears for MariaDB 11.4 in E05.
