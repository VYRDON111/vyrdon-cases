# CASE-005 — NO_PASS

## 1. Case Title

**Merchant Payout / ACH R03 — Platform Says COMPLETED, Originating Bank Returns the File**

A scheduled payout where the platform's settlement engine says `COMPLETED`, the merchant's bank shows no deposit, and the platform's own originating bank received an R03 return file that was never propagated to the settlement engine. Two contradictions: routing/account mismatch + platform ledger vs. originating-bank return.

---

## 2. Surface Signal

Platform dashboard: `COMPLETED`. Merchant dashboard: `COMPLETED`. Customer support script: "the payout was sent — please wait 1–3 business days." Surface is green on both sides.

Underneath: the originating bank's R03 return file ("no account / unable to locate account") has been sitting in the platform's incoming bank feed since day 3, but the settlement engine's reconciliation job has not consumed it, and the platform's customer-facing surfaces have not been informed. **Surface status is not Root truth.**

---

## 3. Root Failure

- `beneficiary_credit_confirmation` = **MISSING** (the merchant's bank never credited the account)
- `routing_destination_match` = **FALSE** (platform stored a truncated account number; required `-Z` suffix dropped during normalization)
- `reconciliation_consistency` = **FALSE** (platform ledger says COMPLETED; originating bank says RETURNED; both signed by their respective sources)

This is **False Root + Contradiction**, not pure Missing Root. The contradiction is between two signed sources within the platform's own evidence chain.

---

## 4. Missing Safeguard

- **`reconcile_ach_returns_before_marking_completed`** — the settlement engine should not transition a payout to `COMPLETED` until the originating bank's return-file window has elapsed and the file has been ingested.
- **`verify_account_number_round_trip`** — the verified-on-file account (`merchant_account_5519-Z`) should be compared byte-for-byte against the account written into the outbound ACH; a silent truncation during data normalization is a structural integrity failure.
- **`propagate_return_to_customer_surfaces`** — when an ACH return is received, the merchant's dashboard and the platform's customer support tooling must be updated within minutes, not at the next reconciliation cycle.
- **`anomaly_counter_on_repeated_returns`** — repeated unreconciled returns against the same routing-rule must escalate to a structural anomaly counter, not be handled per-merchant as one-off support tickets.

---

## 5. VYRDON Root Map

```
ROOT      = FALSE   (routing/account match is False; account number was truncated)
GATE      = FALSE   (the gate accepted platform-ledger COMPLETED without consuming the bank return)
VALID     = MIXED   (debit and transmission fields are present; credit-confirmation is missing; reconciliation is contradicted)
CERTIFIED = FALSE   (no honest certificate can be issued when two signed sources contradict each other)

Contradiction = TRUE  (platform ledger COMPLETED  vs.  originating bank RETURNED)
```

Synthesis factor mapping:

| FinalExecutionPass factor | State |
|---|---|
| MachineGreen | 1 — platform and merchant dashboards both green |
| HumanRedSeal | N/A — payouts are scheduled, not per-event human-sealed |
| ProofComplete | 0 — beneficiary credit confirmation is missing; routing/account match is False |
| ArchiveFinal | 0 — the platform's archive carries `COMPLETED` while the bank's archive carries `RETURNED` |
| CustodySeal | 1 — every evidence artifact (ledger entry, ACH file, return file) is custody-sealed by its source; the seals are intact, the values disagree |

ProofComplete = 0 and ArchiveFinal = 0 → product = 0. Contradiction = TRUE → **NO_PASS**.

---

## 6. Decision

```
DECISION = NO_PASS
DECISION_CODE = DEC-NOPASS-CONTRADICTION
```

**Reasoning.** Two independent custody-sealed sources (the platform's own originating bank and the merchant's bank open-banking feed) contradict the platform's `COMPLETED` status. Contradiction blocks PASS absolutely. The verdict is not HOLD because the contradicting evidence exists — the platform simply did not consume it.

---

## 7. Output Packet

**Payout Failure Reconciliation Packet** — emitted in NO_PASS state, carries: the platform's ledger entry, the outbound ACH file, the originating bank's R03 return file, the merchant's bank open-banking feed showing no credit, the byte-level routing/account mismatch evidence, and the recommended enforcement: **RETURN** — the platform must mark the payout as failed, re-credit the merchant's platform balance, fix the routing/account match, and re-attempt.

---

## 8. System Boundary

VYRDON does **not** claim:

- that the platform acted in bad faith — the failure mode is a settlement-engine reconciliation gap, not fraud
- that R03 is the only return code that produces this pattern — R02, R04, R29 produce similar gaps
- that this case explains why the account-number truncation occurred upstream — that is a separate data-pipeline question

VYRDON **does** claim:

- that `sent` is not `received`, and platforms that treat them as the same will produce this NO_PASS at audit time
- that two custody-sealed sources contradicting each other is the textbook case for the contradiction rule
- that the anomaly counter is the structural detection surface; per-merchant support tickets are not

---

## 9. Audit Note

Reviewer notes:

- A reviewer with access to the platform's settlement engine can verify the gap by joining the outbound-ACH table against the return-file table for the same trace ID; any rows where the outbound table says COMPLETED and the return table says RETURNED are this pattern.
- The merchant has a direct interest in the contradiction (their funds); the platform has a counter-interest (preferring to count payouts as completed). The doctrine sides with the contradicted evidence, not with the interested party.

Maturity: **REVIEW-READY**. The case is reproducible from public-domain ACH mechanics; no real platform, merchant, or bank is referenced.

---

# Detailed Case Record (transaction-decision form)

## Scenario

A merchant operating on a payment platform expects a scheduled weekly payout to their bank account. The payment platform's dashboard shows the payout was sent on the scheduled date. The merchant's bank statement, retrieved through the bank's open-banking feed, shows no incoming deposit. The platform's customer support team initially confirms "the payout was sent — please wait 1-3 business days." After 4 business days the merchant escalates. The platform's settlement engine claims the payout is complete and refuses to re-issue.

This case differs from CASE-002 (HOLD) because here the evidence is not merely missing — it is **contradicted** by the merchant's bank record. Missing root produces HOLD; contradiction produces NO_PASS.

---

## Visible Claim

```
claim_type: MERCHANT_PAYOUT_COMPLETED
claim_status: COMPLETED
claim_source: payment_platform_settlement_engine
transaction_id: PO-2024-10-22091
amount: $7,432.18 USD
currency: USD
beneficiary_account: merchant_account_5519
beneficiary_routing: routing_8821
beneficiary_bank: bank_merchant_us
claimed_action: ACCEPT_PAYOUT_AS_FINAL
timestamp: 2024-10-22T18:00:00Z
```

---

## Required Root

For this claim to PASS, the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| sender_debit_confirmation | YES | The platform's funding account debited the payout amount |
| routing_destination_match | YES | The routing/account number on the payout matches the merchant's verified bank account on file |
| transmission_proof | YES | ACH file or wire instruction transmitted to the platform's originating bank |
| beneficiary_credit_confirmation | YES | The merchant's bank credited the merchant account |
| reconciliation_consistency | YES | The platform-side ledger entry and the bank-side credit reconcile to the same amount and timestamp |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| sender_debit_confirmation | PROVIDED | Platform funding account debited $7,432.18 at 2024-10-22T18:00:15Z; platform internal ledger reference LEDG-22091 |
| routing_destination_match | CONTRADICTED | Payout instruction shows routing `routing_8821` and account `merchant_account_5519`. Merchant's verified-on-file routing is `routing_8821` but verified-on-file account is `merchant_account_5519-Z` (the platform stored a truncated account number; the trailing `-Z` is required by the merchant's bank to route to the correct sub-ledger) |
| transmission_proof | PROVIDED | ACH file transmitted from platform originating bank at 2024-10-22T18:04:00Z; ACH trace TRC-22091; settlement date 2024-10-23 |
| beneficiary_credit_confirmation | MISSING | Merchant's bank open-banking feed shows no incoming credit on 2024-10-23, 2024-10-24, 2024-10-25, or 2024-10-26 against `merchant_account_5519-Z`. A returned ACH was received by the platform's originating bank on 2024-10-25 with return reason code `R03` (no account / unable to locate account) but this return was not propagated back to the merchant or to the settlement engine |
| reconciliation_consistency | CONTRADICTED | Platform ledger says COMPLETED. Originating bank's ACH return file says RETURNED (R03). Both are signed by their respective sources. Internal contradiction within the platform's own evidence chain. |

---

## Missing Evidence

| Missing Element | Impact |
|----------------|--------|
| beneficiary_credit_confirmation | The merchant's bank never credited the account. The funds did not reach the beneficiary. The platform's "COMPLETED" status is based on transmission, not on receipt. |

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Does platform status (`COMPLETED`) match bank-side evidence? | NO — bank received no deposit; platform's own originating bank received a returned ACH |
| Does the payout routing/account on file match the destination of the ACH? | NO — account number on file requires `-Z` suffix; ACH was sent without the suffix |
| Did the platform receive the ACH return? | YES — return arrived 2024-10-25 with R03, but was not propagated to the settlement engine or to the merchant |
| Is the platform's ledger entry consistent with the originating bank's return file? | NO — ledger says COMPLETED; originating bank says RETURNED |
| Does the amount match between the debit and the (now-returned) ACH? | YES — both $7,432.18; the contradiction is on settlement status, not amount |

**Contradiction count: 2** (routing/account mismatch + platform vs originating bank settlement-status contradiction)

This is a critical contradiction. Two independent evidence sources — the merchant's bank feed and the platform's own originating bank — both contradict the platform's "COMPLETED" status. The payout was transmitted, returned, and never credited. The platform's settlement engine never reconciled the return.

---

## RootPass Decision

```
root_valid       = FALSE  (routing/account match is False; account number was truncated, so the ROOT chain is broken at the routing pillar)
gate_valid       = FALSE  (the gate accepted platform-ledger COMPLETED without consuming the originating bank's return file)
evidence_valid   = FALSE  (routing/account mismatch + beneficiary credit absent)
certified_valid  = FALSE  (originating bank return contradicts platform status)
contradiction    = TRUE   (platform status contradicts originating bank return file)

DECISION = NO_PASS
```

Contradiction blocks PASS absolutely. The platform's "COMPLETED" label is one input. The originating bank's return file is another. They disagree. The gate does not vote between them — it records the contradiction and refuses to accept the visible claim.

Even if `evidence_valid` had been TRUE (e.g. if the routing had matched and the credit confirmation simply hadn't arrived yet), the standalone contradiction between platform ledger and originating bank return file would still produce NO_PASS, not HOLD. HOLD is reserved for genuinely missing evidence; here the evidence exists and contradicts itself.

---

## Enforcement Result

```
action: BLOCK_PAYOUT_FINALITY
status: NO_PASS
reason: routing_account_mismatch + settlement_status_contradiction
detail: ACH returned R03 (no account / unable to locate) on 2024-10-25; platform settlement engine never reconciled the return; verified-on-file account requires '-Z' suffix that was truncated in the payout instruction
enforcement_type: payout_block
next_action_1: reissue_payout_with_correct_account_suffix
next_action_2: notify_merchant_with_full_audit_trail
next_action_3: open_engineering_ticket_account_number_truncation
secondary_action: flag_routing_destination_match_for_compliance_review
anomaly_recorded: YES
circuit_breaker_check: anomaly_count incremented (current: 6/10 threshold)
escalation_trigger: third unreconciled return in 30 days will trip the breaker
```

Payout finality blocked. The merchant is notified with the full audit trail (ACH return code, return date, account-suffix truncation). The reissue is queued with the correct account number. The engineering ticket flags the truncation as a systemic data-integrity issue, not just a one-off error.

The anomaly counter is incremented; if this pattern repeats two more times within 30 days, the circuit breaker trips and the platform's automated payout system halts until the truncation bug is fixed and re-verified.

---

## Institution Relevance

This case demonstrates two failure modes that are extremely common in real payment platforms:

1. **"Sent" treated as "received."** The platform's settlement engine declared the payout `COMPLETED` the moment the ACH file was transmitted. It did not check for an inbound credit confirmation, and it did not propagate the returned-ACH file back to the ledger. The dashboard label was true with respect to transmission and false with respect to settlement.
2. **Silent data truncation.** The account number on file required a sub-ledger suffix (`-Z`) that was stripped during a data normalization step. The platform's verification system accepted the routing as "matching" because the prefix matched. Under RootPass, `routing_destination_match` requires exact-match against the verified-on-file record, not prefix-match.

Without RootPass enforcement, the merchant is told "the payout was sent, please contact your bank" — a true-but-incomplete answer that hides a platform-side bug. With RootPass enforcement, the platform cannot mark the payout as final until the credit confirmation arrives, the contradiction is escalated immediately when the ACH return is received, and the data-truncation bug surfaces as a systemic anomaly counter rather than a per-merchant support ticket.

---

## Redaction Note

All identifiers in this case are anonymized. Routing numbers, account numbers, ACH trace IDs, ledger references, and reason codes are illustrative. The R03 return reason code is the real NACHA return code for "no account / unable to locate account" and is used here to ground the scenario in industry-standard semantics. No real merchant, platform, or bank is referenced.
