# E35: the bootstrap rewrite waits for every control-plane node (source reading)

**Claim.** The SHA-256 rewrite of bootstrap data is deferred until every control-plane node carries the annotation `k3s.io/bootstrap-sha256`, and is retried once a minute. The outcome, with servers moved one at a time and with a stock server reading rewritten data, is in E18 and E19.

**Kind of evidence.** A source reading of the bootstrap change (https://github.com/k3s-io/k3s/pull/14981), commit 647e4ba3de, plus the runs named above.

**Source.**

```
pkg/cluster/storage.go:29    BootstrapSHA256Annotation = version.Program + ".io/bootstrap-sha256"
pkg/cluster/storage.go:439   deferLegacyMigration: waits until every control-plane node has the annotation
pkg/cluster/storage.go:460   MigrateBootstrapData: retries once the apiserver can report on every node
pkg/server/server.go:570     wait.PollUntilContextCancel(ctx, time.Minute, true, ...) calls MigrateBootstrapData
```
