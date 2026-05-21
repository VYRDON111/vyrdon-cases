# CASE-002 — HOLD

## Decision: HOLD

---

## Scenario

A payment service provider (PSP) processes a cross-border remittance. The sender's bank claims funds were debited and sent. The recipient bank has not confirmed receipt. The PSP marks the transaction as "completed" in its dashboard and claims settlement is done.

---

## Visible Claim

```
claim_type: REMITTANCE_SETTLED
claim_status: COMPLETED
claim_source: psp_settlement_engine
transaction_id: RMT-2024-11-03882
amount: €12,500.00 EUR
currency: EUR
sender_bank: bank_alpha_ref
recipient_bank: bank_beta_ref
claimed_action: MARK_SETTLED
timestamp: 2024-11-03T16:40:00Z
```

---

## Required Root

For this claim to PASS, the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| sender_debit_confirmation | YES | Proof that sender bank debited funds from sender account |
| correspondent_transfer_proof | YES | Proof that funds moved through the correspondent banking chain |
| recipient_credit_confirmation | YES | Proof that recipient bank credited funds to beneficiary account |
| fx_rate_lock_evidence | YES | Evidence of the exchange rate applied and when it was locked |
| compliance_screening_result | YES | AML/sanctions screening completed for both parties |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| sender_debit_confirmation | PROVIDED | MT103 message reference from bank_alpha; debit timestamp 2024-11-03T14:00:00Z |
| correspondent_transfer_proof | PROVIDED | SWIFT GPI tracker shows funds entered correspondent chain at 2024-11-03T14:30:00Z |
| recipient_credit_confirmation | MISSING | No MT910 or credit advice received from bank_beta. Bank_beta has not confirmed receipt. |
| fx_rate_lock_evidence | PROVIDED | FX rate locked at 1.0842 EUR/USD at 2024-11-03T13:55:00Z; rate lock ticket on file |
| compliance_screening_result | PROVIDED | Both parties cleared AML screening; screening IDs on file |

---

## Missing Evidence

| Missing Element | Impact |
|----------------|--------|
| recipient_credit_confirmation | Without recipient bank confirmation, there is no proof that funds reached the beneficiary. The sender bank debited, the correspondent chain received, but final delivery is unconfirmed. Settlement cannot be declared complete. |

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Does the claimed status ("COMPLETED") match the evidence chain? | NO — status says completed but recipient credit is unconfirmed |
| Is there a timing conflict between debit and claimed settlement? | POSSIBLE — debit was 2.5 hours before settlement claim, which is fast for cross-border but not impossible |
| Are there conflicting status records? | YES — PSP dashboard shows "COMPLETED" but SWIFT GPI tracker shows "IN_TRANSIT" |
| Does the amount match across all legs? | CANNOT VERIFY — final leg amount unknown without recipient confirmation |

**Contradiction count: 1** (dashboard status contradicts GPI tracker status)

However, the primary issue is missing root (recipient confirmation), not the contradiction. The contradiction is a secondary concern that would be resolved if recipient confirmation arrives.

---

## RootPass Decision

```
root_valid       = TRUE
gate_valid       = TRUE
evidence_valid   = MISSING (recipient_credit_confirmation absent)
certified_valid  = TRUE
contradiction    = TRUE (status conflict between PSP dashboard and GPI tracker)

DECISION = HOLD
```

Missing root blocks PASS. Even though contradiction also exists, the primary reason for HOLD is the missing recipient credit confirmation. The system locks the settlement claim and awaits evidence.

Note: If contradiction = TRUE were the only issue (no missing evidence), the decision would be NO_PASS, not HOLD. The HOLD decision prioritizes the possibility that the missing evidence may still arrive.

---

## Enforcement Result

```
action: LOCK_SETTLEMENT_CLAIM
status: HELD
reason: recipient_credit_confirmation missing
secondary_reason: status_contradiction (dashboard vs GPI tracker)
enforcement_type: settlement_hold
next_action: request_mt910_from_bank_beta
escalation_trigger: if no confirmation within 48 hours, escalate to operations room
```

Settlement claim is locked. No "COMPLETED" status is accepted. PSP dashboard label is overridden by gate decision. Awaiting recipient bank confirmation before re-evaluation.

---

## Institution Relevance

This case demonstrates why status labels are insufficient for settlement finality. The PSP dashboard said "COMPLETED" but the proof chain was incomplete. In traditional systems, a dashboard status of "COMPLETED" would be accepted at face value, potentially triggering downstream accounting, reporting, and even further transactions based on a settlement that has not actually completed.

RootPass HOLD prevents premature finality. The claim is neither rejected nor accepted — it is locked until the missing proof element arrives. This is the correct behavior for cross-border remittance where correspondent banking introduces inherent delays.

Institutions affected: PSPs, correspondent banks, treasury operations, compliance teams, remittance platforms.

---

## Redaction Note

All identifiers in this case are anonymized. Bank names, SWIFT references, transaction IDs, and amounts are sanitized. No real institution names, BIC codes, or account numbers appear in this document. The scenario is modeled on common cross-border remittance patterns and does not reference any specific transaction or institution.
