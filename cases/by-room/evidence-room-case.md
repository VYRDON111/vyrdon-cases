# Evidence Room Case: Stale Proof Chain

## Room: Evidence
## Domain: Proof chains — coverage, checksums, signatures, staleness

---

## Scenario

A previously valid proof chain is re-evaluated. The evidence was valid when first recorded, but the attestation has expired. The staleness metric exceeds the maximum allowable age.

## Pipeline Processing

### Stage 1: Ingest
- Proof chain retrieved for re-evaluation
- 4 evidence artifacts in the chain: source document, attestation, checksum, signature

### Stage 2: Compute
- Coverage metric: 100% (all 4 artifacts present)
- Checksum metric: PASS (all checksums verify)
- Signature metric: PASS (all signatures verify)
- Staleness metric: FAIL (attestation age exceeds threshold)

### Stage 3: Explain
- "All evidence artifacts are present and structurally valid. However, the attestation is 47 days old. The maximum allowable age for this evidence type is 30 days. The proof chain is stale."

### Stage 4: Transition
- State transition: `VERIFIED` → `STALE`
- Recorded with reason: "Attestation staleness exceeds threshold"

### Stage 5: Reconcile
- Cross-room check with Commercial room: the transaction this evidence supports is still active
- Reconciliation result: stale evidence supporting an active transaction requires refresh or hold

### Stage 6: Derive
- Summary: "Proof chain structurally valid but stale. Attestation requires refresh."
- Room status: STALE

### Stage 7: Route
- Evidence refresh request routed to source
- Review notification routed to control authority

### Stage 8: Enforce
- Four-pillar check:
  - ROOT: TRUE (evidence root exists)
  - GATE: TRUE (gate valid)
  - VALID: FALSE (evidence is stale — validity has lapsed)
  - CERTIFIED: TRUE (original certification still in chain)
- Result: **PASS.FALSE**

### Stage 9: Export
- Staleness report exported
- Evidence refresh request documented

## Decision

```
ROOT.TRUE + GATE.TRUE + VALID.FALSE + CERTIFIED.TRUE -> PASS.FALSE
```

**NO_PASS** — Evidence chain is structurally complete but the attestation has expired. Proof cannot pass until the evidence is refreshed.

---

## Evidence Components Used

| Component | Status |
|-----------|--------|
| Source | Present ✓ |
| Attestation | Expired ✗ |
| Checksum | Valid ✓ |
| Signature | Valid ✓ |

## Metrics Computed

| Metric | Value | Status |
|--------|-------|--------|
| Coverage | 100% | ✓ |
| Checksum | Verified | ✓ |
| Signature | Verified | ✓ |
| Staleness | 47 days (max: 30) | ✗ |
