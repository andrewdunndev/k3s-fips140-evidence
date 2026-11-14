# E05: external SQL authentication under only

**Claim.** Under `only`, PostgreSQL `md5`, MySQL `mysql_native_password` and MariaDB stop at startup on a non-approved hash; PostgreSQL `scram-sha-256` passes. Under `on` all pass.

**Build.** k3s `main` (v1.37.2), exempt build, GOFIPS140=v1.0.0, two servers and an agent in containers on an Ubuntu 24.04 VM, `--datastore-endpoint` to PostgreSQL 16, MySQL 8.4 or MariaDB 11.4.

**Commands.**

```
k3s server --datastore-endpoint=<dsn> --token <t>
k3s server --datastore-endpoint=<dsn> --token <t> --server ...   # second server
k3s agent --server ...
kubectl get nodes   # three Ready expected
```

**Log lines.**

`PostgreSQL md5 user, only`

```
time="2026-11-14T05:47:02Z" level=fatal msg="Error: server panicked: crypto/md5: use of MD5 is not allowed in FIPS 140-only mode"
```

`MySQL mysql_native_password, only`

```
time="2026-11-14T05:33:02Z" level=fatal msg="Error: server panicked: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode"
```

`MariaDB 11.4, only`

```
time="2026-11-14T05:56:24Z" level=fatal msg="Error: server panicked: crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode"
```

`MySQL caching_sha2_password, only (RSA-OAEP on the full-auth path)`

```
time="2026-11-14T05:28:01Z" level=warning msg="failed to check existence of database k3s, going to attempt create: crypto/rsa: use of hash functions other than SHA-2 or SHA-3 is not allowed in FIPS 140-only mode"
```

`PostgreSQL scram-sha-256, only`

```
== postgres modern build=x mode=only <vm>:v1.37.2-k3s- 2026-11-14T05:46:30Z
ready_nodes=3
```

`MariaDB, on`

```
== mariadb modern build=x mode=on <vm>:v1.37.2-k3s- 2026-11-14T05:55:45Z
ready_nodes=3
```

**Note.** A cached fast-path `caching_sha2_password` login passed under `only` on v1.36.6, so that plugin is not listed as unsupported in the ADR.
