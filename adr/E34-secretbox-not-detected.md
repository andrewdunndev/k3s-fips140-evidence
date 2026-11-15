# E34: `secretbox` is not detected by `only` (source reading)

**Claim.** K3s offers a `secretbox` encryption provider. Go's `only` mode does not check it, so a job under `only` passes without testing it and does not show that the provider is approved.

**Kind of evidence.** A source reading, not a run.

**Source.**

```
k3s pkg/cli/cmds/server.go:643                   offers the secretbox provider
golang.org/x/crypto@v0.54.0/nacl/secretbox       0 files match "fips140"
golang.org/x/crypto@v0.54.0 salsa20, scrypt, bcrypt   0 files match "fips140" each
```
