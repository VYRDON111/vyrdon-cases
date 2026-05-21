# CASE-001 — PASS

## Decision: PASS

---

## Scenario

A marketplace escrow platform processes a service completion claim. A freelancer completed a deliverable for a client. The platform claims the transaction is verified and ready for payout release.

---

## Visible Claim

```
claim_type: SERVICE_COMPLETED
claim_status: VERIFIED
claim_source: marketplace_escrow_platform
transaction_id: TXN-2024-08291-A
amount: $4,200.00 USD
currency: USD
beneficiary: freelancer_account_7291
claimed_action: RELEASE_PAYOUT
timestamp: 2024-08-29T14:22:00Z
```

---

## Required Root

For this claim to PASS, the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| proof_of_delivery | YES | Evidence that the deliverable was submitted and received |
| client_approval | YES | Explicit client sign-off on the deliverable |
| escrow_deposit_confirmation | YES | Proof that funds were deposited into escrow before work began |
| dispute_window_elapsed | YES | The contractual dispute window (72 hours) has passed without dispute |
| identity_verification | YES | Both parties passed KYC at onboarding |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| proof_of_delivery | PROVIDED | SHA-256 hash of deliverable file uploaded to platform storage; timestamp matches contract window |
| client_approval | PROVIDED | Client clicked "Approve Deliverable" at 2024-08-27T09:15:00Z; approval event logged with client session ID |
| escrow_deposit_confirmation | PROVIDED | $4,200 deposited at 2024-08-15T10:00:00Z; bank reference BNK-8827361; escrow ledger entry exists |
| dispute_window_elapsed | PROVIDED | 72-hour window started 2024-08-27T09:15:00Z, ended 2024-08-30T09:15:00Z; no dispute filed; claim submitted after window close |
| identity_verification | PROVIDED | Both parties completed KYC at account creation; verification records on file |

---

## Missing Evidence

None. All required root elements are present and consistent.

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Does the claimed amount match escrow deposit? | YES — $4,200 matches |
| Does the approval timestamp precede the release request? | YES — approval 2024-08-27, release request 2024-08-29 |
| Was any dispute filed during the window? | NO |
| Does the deliverable hash match the contract specification? | YES — hash verified against contract attachment |
| Are there conflicting status records? | NO — single consistent status chain |

**Contradiction count: 0**

---

## RootPass Decision

```
root_valid       = TRUE
gate_valid       = TRUE
evidence_valid   = TRUE
certified_valid  = TRUE
contradiction    = FALSE

DECISION = PASS
```

All four pillars are TRUE. No contradiction detected. Root chain is complete and consistent.

---

## Enforcement Result

```
action: RELEASE_PAYOUT
amount: $4,200.00
beneficiary: freelancer_account_7291
enforcement_type: escrow_release
timelock_check: ELAPSED (72h dispute window passed)
circuit_breaker: NOT_TRIPPED
```

Funds released to beneficiary. Evidence record written. Decision logged to append-only journal.

---

## Institution Relevance

This case demonstrates RootPass applied to marketplace escrow. Institutions operating marketplace platforms, freelance platforms, or gig economy settlement systems face this exact pattern: a service claim must be backed by delivery proof, client approval, deposit confirmation, and dispute window compliance before payout is authorized.

Without RootPass enforcement, platforms typically release on status label alone ("VERIFIED"), which permits release even when root evidence is incomplete or contradicted.

---

## Redaction Note

All identifiers in this case are anonymized. Transaction IDs, account references, and amounts are sanitized. No real party names, addresses, or financial institution identifiers appear in this document. The scenario is modeled on common marketplace escrow patterns and does not reference any specific platform or transaction.
