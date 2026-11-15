# E32: bootstrap data encryption stops under `only` on PBKDF2-SHA-1

**Claim.** Stock k3s derives the bootstrap data key with `pbkdf2.Key(..., 4096, 32, sha1.New)` and an 8-byte salt (`pkg/cluster/encrypt.go:31` and `:60`). Under `only` the server's first error is the PBKDF2 hash check, on every datastore, with or without the SHA-1 exemptions. The series moves the key to SHA-256.

**Build.** k3s release-1.36 (v1.36.6) and `main` before the bootstrap change, GOFIPS140=v1.0.0, GODEBUG=fips140=only, on an Ubuntu 24.04 cloud VM. The same suites pass under `only` on v1.36.6 with the bootstrap change applied.

**Log line.**

```
level=fatal msg="Error: server panicked: crypto/pbkdf2: use of hash functions other than SHA-2 or SHA-3 is not allowed in FIPS 140-only mode"
```

**Note.** The panic comes after etcd is ready and during bootstrap save. It reproduces with the SHA-1 call sites patched and with them exempt, so it is the bootstrap key and not the exemptions. This is also why the release-1.36 build cannot reach the `token rotate` guard (E22) without the bootstrap change.
