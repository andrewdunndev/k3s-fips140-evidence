# E22: `token rotate` guard on release-1.36 under `only`

**Claim.** With the guard an 8-byte rotate is refused (rc 1), the token file is unchanged and a restart on the old token is Ready. Without it the rotate is accepted and the next start is fatal.

**Build.** k3s v1.36.6 (release-1.36, x/crypto v0.47.0) plus the series, GOFIPS140=v1.0.0, GODEBUG=fips140=only. "rtx" has the guard, "rtx0" does not (control). One server on an Ubuntu 24.04 VM.

**Commands.**

```
k3s token rotate --new-token <8 bytes>      # B2.1 guard, B2.3 control
k3s token rotate --new-token <23 bytes>     # B2.2
# restart k3s on the old or new token and read readiness
```

**Log lines.**

`result ledger`

```
CELL B2.0 build=rtx0 mode=only fresh-start=READY
CELL B2.1 start build=rtx mode=only outcome=READY
CELL B2.1 rotate8 rc=1 token_file_changed=no server_alive=yes
CELL B2.1 restart-on-old-token build=rtx outcome=READY
CELL B2.2 start build=rtx mode=only outcome=READY
CELL B2.2 rotate23 rc=0
CELL B2.2 restart-on-23byte-token outcome=READY
CELL B2.3 start build=rtx0 mode=only outcome=READY
CELL B2.3 rotate8 rc=0 token_file_changed=yes server_alive=yes
CELL B2.3 restart-on-new-8byte-token build=rtx0 outcome=EXITED
```

`guard: CLI output`

```
time="2026-11-14T17:03:42Z" level=fatal msg="Error: Internal error occurred: token error ID 99018"
```

`control: next start`

```
time="2026-11-14T17:05:20Z" level=fatal msg="Error: failed to hash password for username 'node' in password file '/var/lib/rancher/k3s/server/cred/passwd': secret is shorter than 14 bytes, which fips140=only mode requires"
```

**Note.** The refusal text reaches the operator in the server log only; the CLI prints `token error ID 99018`. The retained server log for the guard case does not hold that line, so it is not quoted here. `main` (x/crypto v0.54.0) needs no guard.
