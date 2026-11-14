# E18: moving servers to `on` one at a time

**Claim.** Three servers and an agent hold quorum while servers move one at a time from the mode off to `on`, and a stock server restarted on the rewritten data stays Ready when it joins live peers.

**Build.** k3s v1.37.2 series build, GOFIPS140=v1.0.0, GODEBUG `off` then `on` per server; stock binary where marked `s/off`. Three embedded-etcd servers and an agent in containers on an Ubuntu 24.04 VM.

**Commands.**

```
# start all nodes on the stock binary, mode off; then, one server at a time:
docker stop <server>; docker run <server> with the series binary, GODEBUG=fips140=on
kubectl get nodes; read a marker configmap from every server; grep logs for "illegal base64"
```

**Log lines.**

`step ledger (x = series build, s = stock)`

```
STEP 0 all s/off nodes_ready=4 => PASS
STEP 1 moved=srv1->x/on nodes_ready=4 illegal_base64=0 cluster: srv1=x/on srv2=s/off srv3=s/off agt=s/off => PASS
STEP 1a2 moved=srv1->x/on nodes_ready=4 illegal_base64=0 cluster: srv1=x/on srv2=s/off srv3=s/off agt=s/off => PASS
STEP 1b srv3 stock s/off restart on rewritten data: illegal_base64=0 others: nodes_ready=4
STEP 2 moved=srv2->x/on nodes_ready=4 illegal_base64=0 cluster: srv1=x/on srv2=x/on srv3=s/off agt=s/off => PASS
STEP 3 moved=srv3->x/on nodes_ready=4 illegal_base64=0 cluster: srv1=x/on srv2=x/on srv3=x/on agt=s/off => PASS
```

**Note.** Step 1b is a stock joiner with live peers. A stock server that must read the datastore itself fails; see E19.
