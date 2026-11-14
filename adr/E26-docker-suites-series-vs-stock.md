# E26: Test_DockerFIPS and Test_DockerFIPSMigration on the series, and on stock

**Claim.** On the series both suites pass (7 of 7 and 8 of 8 specs) under `on` and `only`, on `main` and release-1.36; a stock binary fails at "Setup Cluster" in the same suite.

**Build.** k3s `main` (v1.37.2) and release-1.36 (v1.36.6) with the series; control is the stock base of each. GOFIPS140=v1.0.0, Go 1.26, docker buildx `result` target, suites bind-mount the built binary. Ubuntu 24.04 VM.

**Commands.**

```
go test ./tests/docker/fips/... -ginkgo.v          # Test_DockerFIPS, GODEBUG=fips140=only and on
go test ./tests/docker/fipsmigration/... -ginkgo.v  # Test_DockerFIPSMigration
# control: same command with the stock binary
```

**Log lines.**

`series ledger (suite | result | detail)`

```
fips-only-main | PASS | Ran 7 of 7 Specs in 105.179 seconds
fips-on-main | PASS | Ran 7 of 7 Specs in 105.050 seconds
fipsmigration-main | PASS | Ran 8 of 8 Specs in 148.550 seconds
control-fips-main | FAIL | rc=1 Ran 1 of 7 Specs in 18.826 seconds  [FAIL] Setup Cluster
fips-only-r136 | PASS | Ran 7 of 7 Specs in 133.030 seconds
fips-on-r136 | PASS | Ran 7 of 7 Specs in 119.196 seconds
fipsmigration-r136 | PASS | Ran 8 of 8 Specs in 146.590 seconds
control-fips-r136 | FAIL | rc=1 Ran 1 of 7 Specs in 20.180 seconds  [FAIL] Setup Cluster
```

**Note.** Unit `only` on release-1.36 has one failing package, `pkg/server/handlers`, that stock release-1.36 also fails under `only` (short test passwords).
