# E21: release-1.37 with the series

**Claim.** The six k3s docker suites that run unchanged pass under `on` and `only` on release-1.37 with the series applied.

**Build.** k3s `release-1.37` plus the series, call-site exemptions, GOFIPS140=v1.0.0, Go 1.26.10. Image environment bakes `GODEBUG=fips140=<mode>`. Ubuntu 24.04 VM.

**Commands.**

```
go test ./tests/docker/<suite>/... -ginkgo.v   # image GODEBUG=fips140=on, then only
# suites: basics bootstraptoken cacerts gatewayapi lazypull t4
```

**Log lines.**

`suite results (rc 0 is pass)`

```
basics on image_env=GODEBUG=fips140=on rc=0 [Ran 5 of 5 Specs in 50.651 seconds]
basics only image_env=GODEBUG=fips140=only rc=0 [Ran 5 of 5 Specs in 54.739 seconds]
bootstraptoken on image_env=GODEBUG=fips140=on rc=0 [Ran 3 of 3 Specs in 59.429 seconds]
bootstraptoken only image_env=GODEBUG=fips140=only rc=0 [Ran 3 of 3 Specs in 49.448 seconds]
cacerts on image_env=GODEBUG=fips140=on rc=0 [Ran 3 of 3 Specs in 45.898 seconds]
cacerts only image_env=GODEBUG=fips140=only rc=0 [Ran 3 of 3 Specs in 43.024 seconds]
gatewayapi on image_env=GODEBUG=fips140=on rc=0 [Ran 7 of 7 Specs in 58.764 seconds]
gatewayapi only image_env=GODEBUG=fips140=only rc=0 [Ran 7 of 7 Specs in 58.843 seconds]
lazypull on image_env=GODEBUG=fips140=on rc=0 [Ran 5 of 5 Specs in 61.246 seconds]
lazypull only image_env=GODEBUG=fips140=only rc=0 [Ran 5 of 5 Specs in 54.980 seconds]
t4 on image_env=GODEBUG=fips140=on rc=0 [Ran 5 of 5 Specs in 107.490 seconds]
t4 only image_env=GODEBUG=fips140=only rc=0 [Ran 5 of 5 Specs in 103.986 seconds]
```

`toolchain`

```
go version go1.26.10 linux/amd64
```

