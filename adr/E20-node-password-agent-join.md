# E20: a legacy node password under `only` returns an error that names the remedy

**Claim.** With an 8-byte-salt node password, an agent join under `only` is answered `403 ... delete the node password secret so the node can re-register`, with no panic. Deleting the secret fixes it and writes a 16-byte salt. The control build panics.

**Build.** k3s v1.37.2 (series, call-site exemptions "cxn"), GOFIPS140=v1.0.0; stock v1.36.6 for phase 1; control "cx" is the same tree without the salt check. One server and one agent on an Ubuntu 24.04 VM. Phases: stock `off`, series `on`, series `only`.

**Commands.**

```
# phase 1: stock v1.36.6 server and agent, GODEBUG=fips140=off
# phase 2: series build, fips140=on, same data
# phase 3: series build, fips140=only, same data; restart the agent
kubectl -n kube-system delete secret agt.node-password.k3s; restart the agent
```

**Log lines.**

`result ledger, node password case`

```
CELL B1.1 phase1 server build=stock mode=off outcome=READY alive=yes
CELL B1.1 phase1 agent stock off agt_ready_rc=0 agt_hash=[nfields=6 lens=2 16 2 1 1 86 prefix=$1] srv_hash=[nfields=6 lens=2 16 2 1 1 86 prefix=$1] agtpw_bytes=33
CELL B1.1 phase2 server build=cxn mode=on outcome=READY alive=yes
CELL B1.1 phase2 agent build=cxn mode=on agt_ready_rc=0 agt_hash=[nfields=6 lens=2 16 2 1 1 86 prefix=$1]
CELL B1.1 phase3 server build=cxn mode=only outcome=READY alive=yes
CELL B1.1 phase3 build=cxn mode=only server_alive=yes join_result=NAMED-ERROR-AGT-403
CELL B1.1 remedy delete_rc=0 out=[secret "agt.node-password.k3s" deleted from kube-system namespace ]
CELL B1.1 remedy-agent restart agt_ready_rc=0 new_agt_hash=[nfields=6 lens=2 32 2 1 1 86 prefix=$1] server_alive=yes
CELL B1.3 phase3 server build=cx mode=only outcome=READY alive=yes
CELL B1.3 phase3 build=cx mode=only server_alive=no join_result=PANIC
```

`server log under only: the 403 returned to the agent`

```
time="2026-11-14T16:59:19Z" level=error msg="Sending HTTP/1.1 403 response to <ip>: unable to verify password for node agt: hash salt is 8 bytes, fips140=only requires at least 16; delete the node password secret so the node can re-register"
```

`control under only: the panic`

```
time="2026-11-14T17:02:02Z" level=error msg="Observed a panic" error="<nil>" logger=k3s panic="crypto/pbkdf2: use of salts shorter than 128 bits is not allowed in FIPS 140-only mode" panicGoValue="&errors.errorString{s:\"crypto/pbkdf2: use of salts shorter
```

**Note.** Hash field lengths `2 16 ...` are an 8-byte salt and `2 32 ...` a 16-byte salt.
