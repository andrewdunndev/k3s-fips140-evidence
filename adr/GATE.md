# Series test run

The whole series was applied to k3s main and to release-1.36 and run once on the final heads. Each build used `GOFIPS140=v1.0.0`.

| Ref | sha |
|---|---|
| k3s main base | c55a7df0fc49e826072b163d2c91ab4f67f06325 |
| k3s release-1.36 base | 7c835148e87f4403dd87fbc4c0a09792b9a26021 |
| main with the series | fdf72b660e0555a0b7a6e6f4473e97c80f2c6a9e |
| release-1.36 with the series | 264f5fd66bba6cea61d0ea5b19fd563207bac42c |

The tested heads are on https://github.com/andrewdunndev/k3s, branch `fips-series-main` (fdf72b660e0555a0b7a6e6f4473e97c80f2c6a9e) and branch `fips-series-release-1.36` (264f5fd66bba6cea61d0ea5b19fd563207bac42c).

The tips end in one test-only `go.mod replace` commit, which points wrangler, dynamiclistener and kube-router at their open fixes.

| Check | Line | Result |
|---|---|---|
| build | main, release-1.36, both stock bases | pass |
| unit tests under off, `on` and `only` | main | pass |
| unit tests under off, `on` and `only` | release-1.36 | pass |
| `Test_DockerFIPS` under `only` and `on` | both | pass, 7 of 7 |
| `Test_DockerFIPSMigration` | both | pass, 8 of 8 |
| `Test_DockerFIPSMixedVersion`, older server is stock | both | pass, 8 of 8 |
| stock binary, `Test_DockerFIPS` under `only` | both bases | fails at cluster setup: `crypto/sha1: use of SHA-1 is not allowed in FIPS 140-only mode` |
| `k3s token rotate` to an 8-byte token under `only` | release-1.36 | refused: `secret is shorter than 14 bytes`, exit 1 |

On release-1.36, three handler tests use node passwords and tokens shorter than 14 bytes, which the release-branch guard rejects under `only`. The run adapts only those test fixtures.
