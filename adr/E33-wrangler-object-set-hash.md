# E33: wrangler's object set hash is SHA-1 (source reading)

**Claim.** `wrangler/v3` computes the apply object set hash with `sha1.New`. The value is persisted as a label on applied objects, so changing the algorithm changes stored values. It stays SHA-1 inside `fips140.WithoutEnforcement`, at the narrowest call.

**Kind of evidence.** A source reading, not a run.

**Source.** `github.com/rancher/wrangler/v3@v3.7.0/pkg/apply/desiredset_apply.go:276` (`sha1.New`). The wrap is proposed in https://github.com/rancher/wrangler/pull/797.

**Note.** E02 shows the exempted build under `only` with this call included.
