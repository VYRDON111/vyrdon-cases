# CASE-004 — PASS

## 1. Case Title

**Corporate Treasury Cross-Border Wire — Full SWIFT Chain Proved**

The proof-positive complement to CASE-002 (HOLD): the same SWIFT mechanics, but with the recipient credit confirmation actually present.

---

## 2. Surface Signal

Treasury platform shows `SETTLED`. Originator bank shows `DEBIT_POSTED`. Destination bank shows `CREDIT_POSTED`. SWIFT GPI tracker shows the full path with monotonic timestamps. Surface and Root agree. **Surface status is not Root truth — but in this case the Root is also TRUE, and the agreement is sealed.**

---

## 3. Root Failure

None. All four pillars are TRUE. See §5 for the full Root map.

---

## 4. Missing Safeguard

None. All required safeguards were exercised:

- Dual-control sender authorization (two distinct authorizer keys, authority separation enforced)
- Sender debit confirmation (MT103 from originator bank)
- Correspondent transfer proof (SWIFT GPI tracker)
- Beneficiary credit confirmation (MT910 from destination bank)
- Sanctions screening (OFAC + UN + local, with rescreen at boundary)
- Purpose-code match (SUPP code matches vendor classification and invoice line items)

---

## 5. VYRDON Root Map

```
ROOT      = TRUE    (sender authorization sealed with dual control)
GATE      = TRUE    (sanctions + purpose-code + screening boundary all clear)
VALID     = TRUE    (all 6 required evidence items provided and internally consistent)
CERTIFIED = TRUE    (MT910 credit advice seals the beneficiary credit; decision record is bound to the evidence)

Contradiction = FALSE  (no conflicting records across legs)
```

Synthesis factor mapping:

| FinalExecutionPass factor | State |
|---|---|
| MachineGreen | 1 — platform and bank dashboards consistent |
| HumanRedSeal | 1 — two distinct treasury authorizers, dual control enforced |
| ProofComplete | 1 — every required Root present with matching amounts and monotonic timestamps |
| ArchiveFinal | 1 — payment-finality record sealed under append-only journal |
| CustodySeal | 1 — each evidence artifact (MT103, GPI, MT910, screening reference) carries a custody-sealed source |

All factors = 1 → product = 1 → PASS.

---

## 6. Decision

```
DECISION = PASS
DECISION_CODE = DEC-PASS
```

**Reasoning.** All four pillars TRUE, no contradiction. Wire chain is complete end-to-end with matching amounts and monotonic timestamps. Sealed credit advice from the destination bank is the certifying artifact.

---

## 7. Output Packet

**Wire Finality Packet** — emitted in PASS state, carries: the MT103 / GPI / MT910 chain, the dual-control authorization seals, the sanctions screening references, the purpose-code match, and the sealed decision record. The packet is the artifact a downstream auditor or counterparty needs to confirm payment finality without consulting the originating platform's dashboard.

---

## 8. System Boundary

VYRDON does **not** claim:

- that all SWIFT wires of this shape are PASS — only that this specific evidence chain produces PASS
- that the treasury platform's `SETTLED` label was the basis of decision — the decision rests on the MT910 credit advice, not on the dashboard status
- that future state cannot drift from this snapshot — reversal, recall, or fraud disclosure on the destination side would re-open the decision

VYRDON **does** claim:

- that the decision is reproducible: anyone with the same six evidence artifacts would reach PASS
- that dual-control authorization is a structural property of this PASS, not a procedural nicety
- that the MT910 credit advice is the certifying artifact, not the platform's dashboard

---

## 9. Audit Note

This is the audit-clean shape of a corporate-treasury wire. Reviewer notes:

- The same shape is reachable under fraudulent collusion (both authorizers compromised, MT910 forged); the doctrine catches contradictions, not insider collusion at the seal-issuing layer.
- This case demonstrates **what PASS looks like** as the complement to CASE-002 (HOLD) and CASE-005 (NO_PASS on the payout side).

Maturity: **REVIEW-READY**. The case is reproducible from public-domain SWIFT mechanics; no real party, bank, or transaction is referenced.

---

# Detailed Case Record (transaction-decision form)

## Scenario

A corporate treasury team initiates a USD-denominated wire transfer to a vendor in another country to settle an outstanding supplier invoice. The wire passes through a domestic ACH cutover, a correspondent bank, and a destination bank. The treasury platform claims the wire is settled and ready to be released as evidence of payment. Unlike the cross-border PSP case (CASE-002 HOLD), every leg of this wire has matching evidence at the time the claim is evaluated.

---

## Visible Claim

```
claim_type: BANK_WIRE_SETTLED
claim_status: SETTLED
claim_source: corporate_treasury_platform
transaction_id: BW-2024-09-03291
amount: $185,000.00 USD
currency: USD
sender_account: corp_treasury_4421
sender_bank: bank_originator_us
correspondent_bank: bank_correspondent_intl
beneficiary_account: vendor_acct_8814
beneficiary_bank: bank_destination_de
purpose_code: SUPP (supplier payment)
claimed_action: RECORD_PAYMENT_AS_FINAL
timestamp: 2024-09-03T15:47:00Z
```

---

## Required Root

For this claim to PASS, the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| sender_authorization | YES | Treasury authorizer signed the wire instruction with the dual-control key |
| sender_debit_confirmation | YES | Originator bank debited the corporate treasury account |
| correspondent_transfer_proof | YES | Correspondent bank received and forwarded funds; SWIFT GPI tracker shows path |
| beneficiary_credit_confirmation | YES | Destination bank credited the vendor account (MT910 or equivalent credit advice) |
| sanctions_screening_result | YES | OFAC/UN sanctions screening completed for sender, beneficiary, and beneficiary bank |
| purpose_code_match | YES | Wire purpose code matches the underlying invoice and vendor classification |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| sender_authorization | PROVIDED | Dual-control signature: authorizer key `key_treas_a1`, releaser key `key_treas_b2`. Authority separation enforced (two distinct human authorizers). Signature timestamps: 15:33:12Z and 15:34:08Z |
| sender_debit_confirmation | PROVIDED | Originator bank MT103 reference MT103-08-291-77541; debit posted at 15:35:21Z to corp_treasury_4421 in the amount of $185,000.00 |
| correspondent_transfer_proof | PROVIDED | SWIFT GPI tracker `GPI-x9k2-77541` shows the funds entered the correspondent chain at 15:39:04Z and departed for the destination bank at 15:41:33Z |
| beneficiary_credit_confirmation | PROVIDED | Destination bank MT910 credit advice received at 15:46:12Z; vendor account vendor_acct_8814 credited $185,000.00; no FX leg (USD-to-USD nostro) |
| sanctions_screening_result | PROVIDED | OFAC + UN + local screening lists all cleared at 15:30:55Z; screening reference SCR-09-03-44188; rescreen at boundary cleared at 15:46:00Z |
| purpose_code_match | PROVIDED | Purpose code SUPP matches vendor's recorded classification; underlying invoice INV-2024-VEN-1188 referenced in wire memo; amount matches invoice line items |

---

## Missing Evidence

None. All required root elements are present with consistent timestamps and matching amounts across every leg.

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Does the amount match across debit, correspondent, and credit legs? | YES — $185,000.00 USD on all three legs |
| Are timestamps monotonically increasing across the chain? | YES — 15:35:21 (debit) → 15:39:04 (correspondent in) → 15:41:33 (correspondent out) → 15:46:12 (beneficiary credit) |
| Do sender authority signatures pass dual-control rules? | YES — two distinct authorizer keys, both within authority validity window |
| Does the beneficiary bank's MT910 reference the originator's MT103? | YES — MT910 carries MT103 reference MT103-08-291-77541 |
| Are there conflicting status records between treasury platform and bank channels? | NO — treasury platform `SETTLED`, originator bank `DEBIT_POSTED`, destination bank `CREDIT_POSTED` are consistent at this stage |
| Does sanctions screening cover the destination beneficiary bank? | YES — screening result includes the beneficiary bank BIC |

**Contradiction count: 0**

---

## RootPass Decision

```
root_valid       = TRUE   (sender authorization complete with dual control)
gate_valid       = TRUE   (sanctions + purpose code + screening boundary all clear)
evidence_valid   = TRUE   (all 6 required evidence items provided and consistent)
certified_valid  = TRUE   (MT910 credit advice confirms beneficiary credit)
contradiction    = FALSE  (no conflicting records across legs)

DECISION = PASS
```

All four pillars are TRUE. No contradiction detected. Wire chain is complete end-to-end with matching amounts and monotonic timestamps.

---

## Enforcement Result

```
action: RECORD_PAYMENT_AS_FINAL
amount: $185,000.00 USD
beneficiary: vendor_acct_8814
enforcement_type: payment_finality_recording
timelock_check: N/A (wire is already cleared at destination)
circuit_breaker: NOT_TRIPPED (anomaly_count: 2/10 threshold)
audit_trail_entry: created (append-only journal entry written)
downstream_effects: invoice INV-2024-VEN-1188 marked PAID; AP ledger reconciled
```

Payment recorded as final. Invoice closed. Evidence record written to append-only journal. Decision logged with all 6 evidence references and 6 contradiction-check results.

---

## Institution Relevance

This case demonstrates RootPass applied to corporate-treasury bank wires where every leg of the SWIFT chain has been independently proved. It is the proof-positive complement to CASE-002 (HOLD): the same SWIFT mechanics, but with the recipient credit confirmation actually present. Institutions operating treasury, AP, or cross-bank settlement systems face exactly this pattern — and the typical failure mode is accepting "treasury platform says SETTLED" as the basis for closing the invoice, without independent MT910 evidence from the destination bank. Under RootPass, the platform's `SETTLED` label is only one input among six. The decision is anchored on the credit confirmation from the beneficiary bank plus the dual-control sender authorization, not on a dashboard status.

This case also illustrates the **authority separation** principle: the sender authorization is split across two distinct keys (`key_treas_a1` authorizer and `key_treas_b2` releaser). Authority separation is recorded on the decision; if a single key had signed both fields, the gate would have failed `gate_valid` even though all bank-side evidence was present.

---

## Redaction Note

All identifiers in this case are anonymized. Bank BICs, account numbers, invoice numbers, GPI tracker IDs, MT103/MT910 references, screening references, and key identifiers are sanitized. The scenario is modeled on common cross-border corporate treasury wire patterns and does not reference any specific bank, vendor, or transaction.
