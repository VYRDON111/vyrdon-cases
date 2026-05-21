# Commercial Room Case: Payment Settlement Contradiction

## Room: Commercial
## Domain: Transaction flows — inflow, outflow, pending, receipt, settlement

---

## Scenario

A payment is submitted. The sender's system reports "settled." The receiver's system reports "pending." The settlement metric shows a contradiction between inflow (recorded) and receipt (not confirmed).

## Pipeline Processing

### Stage 1: Ingest
- Transaction data ingested from both sender and receiver
- Sender claims: settlement complete
- Receiver claims: funds not received

### Stage 2: Compute
- Inflow metric: recorded (sender confirmed debit)
- Outflow metric: pending (receiver has not confirmed credit)
- Settlement metric: contradiction detected
- Pending metric: active (settlement in limbo)

### Stage 3: Explain
- "Sender reports settlement complete. Receiver reports funds pending. Settlement metric shows inflow without confirmed receipt. This is a contradiction — both cannot be true simultaneously."

### Stage 4: Transition
- State transition: `SUBMITTED` → `CONTRADICTION_DETECTED`
- Room law check: valid transition ✓
- State law check: recorded with reason and timestamp ✓

### Stage 5: Reconcile
- Cross-room check with Evidence room: evidence of sender settlement exists (bank confirmation). Evidence of receiver receipt does not exist.
- Reconciliation result: contradiction confirmed. Evidence is one-sided.

### Stage 6: Derive
- Summary: "Payment settlement contradicted. Sender evidence exists. Receiver evidence missing."
- Room status: CONTRADICTION

### Stage 7: Route
- Action routed to escalation handler
- Review request routed to control authority
- Evidence gap routed to evidence room for trace

### Stage 8: Enforce
- Four-pillar check:
  - ROOT: TRUE (transaction root exists)
  - GATE: TRUE (gate is valid)
  - VALID: FALSE (claim is contradicted)
  - CERTIFIED: FALSE (certification chain incomplete)
- Result: **PASS.FALSE** — Enforcement blocks release

### Stage 9: Export
- Decision manifest exported
- Evidence bundle exported (sender evidence, missing receiver evidence)
- Audit report generated

## Decision

```
ROOT.TRUE + GATE.TRUE + VALID.FALSE + CERTIFIED.FALSE -> PASS.FALSE
```

**NO_PASS** — Settlement claim cannot pass because the claim is contradicted by missing receiver evidence.

## Financial Effect

Funds held in escrow. Not released to receiver. Not returned to sender. HOLD until evidence resolves the contradiction.

---

## Room Components Used

| Component | Role |
|-----------|------|
| Stamp | Transaction timestamp and authorization record |
| Receipt | Proof of receipt (MISSING — this is the gap) |
| In/Out | Inflow recorded, outflow not confirmed |
| Pending | Transaction remains pending |
| Question Mark | Contradiction flagged for review |

## Metrics Computed

| Metric | Value | Status |
|--------|-------|--------|
| Inflow | Recorded | ✓ |
| Outflow | Not confirmed | ✗ |
| Pending | Active | ⏳ |
| Receipt | Not confirmed | ✗ |
| Settlement | Contradicted | ✗ |
