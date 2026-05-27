# CASE-006 — NO_PASS

## 1. Case Title

**Refund Dispute / Contradictory Evidence — Gate Refuses to Pick a Winner**

A card-network refund dispute where the merchant's usage logs and the customer's travel records are both custody-sealed by their respective sources, both internally consistent, and mutually contradictory. The gate's job is not to determine who is right; the gate's job is to refuse to PASS a finality decision over a contradicted evidence chain. **VYRDON does not pick a winner when evidence contradicts; it returns NO_PASS or HOLD.**

---

## 2. Surface Signal

Merchant dashboard: `DISPUTE_RESPONDED, MERCHANT_FAVOR`. Customer dashboard: `DISPUTE_FILED, AWAITING_RESOLUTION`. Card-network status: `IN_REVIEW`. Processor decision engine recommendation: `DENY` (refund unwarranted). Customer issuing-bank recommendation: `APPROVE` (refund warranted).

The surface is a tie. Both sides have signed evidence. Both dashboards reflect their owner's view. **Surface status is not Root truth, and in a contradicted-evidence case, no surface is Root truth.**

---

## 3. Root Failure

- `service_usage_evidence` = **CONTRADICTED** (merchant: 14 sessions from country A; customer: travel records placing them in country B for the entire period; both sources internally consistent)
- `identity_consistency` = **CONTRADICTED** (email on subscription matches cardholder; IP addresses on usage do not match cardholder's verified geography)

This is **False Root via Contradiction**, not Missing Root. Both contradicting artifacts exist, both are sealed by their producing party, both are internally consistent. The doctrine's contradiction priority rule applies: contradiction blocks PASS absolutely, regardless of which side appears more credible.

---

## 4. Missing Safeguard

- **`require_independent_identity_anchor_before_usage`** — a subscription's usage logs should anchor to an identity proof that is independent of the email-on-file (device attestation, signed login challenge, biometric attestation, or out-of-band confirmation).
- **`record_geography_at_login_with_custody_seal`** — IP-address geography at login is a signal; a sealed geography record (network-level attestation, telco/ISP-issued, or carrier-grade NAT trace) is closer to a root.
- **`require_cardholder_present_for_high_risk_actions`** — if usage and billing geography drift apart by more than a configured threshold, the merchant should issue a cardholder-present challenge before continuing service.
- **`escalate_contradicted_evidence_to_dispute_packet`** — the processor's decision engine should not produce a `DENY` recommendation when the evidence chain is contradicted; it should produce a Contradicted Evidence Packet and return NO_PASS to the gate, leaving the resolution to a higher-authority process.

---

## 5. VYRDON Root Map

```
ROOT      = FALSE   (identity_consistency is contradicted by geography mismatch)
GATE      = FALSE   (the gate cannot pass a finality decision over contradicted evidence)
VALID     = MIXED   (payment fields are present; usage and identity fields are contradicted)
CERTIFIED = FALSE   (no honest certificate can be issued when two signed sources contradict each other)

Contradiction = TRUE  (merchant usage logs  vs.  customer travel records;
                      email-on-file identity  vs.  geography-on-usage identity)
```

Synthesis factor mapping:

| FinalExecutionPass factor | State |
|---|---|
| MachineGreen | 1 — merchant decision engine says DENY |
| HumanRedSeal | 1 — customer affidavit is signed under penalty of perjury |
| ProofComplete | 0 — service_usage and identity_consistency are contradicted, not complete |
| ArchiveFinal | 0 — no final archive can be honestly sealed under contradicted roots |
| CustodySeal | 1 — every artifact carries a custody seal; the seals are valid, the values disagree |

ProofComplete = 0 and ArchiveFinal = 0 → product = 0. Contradiction = TRUE → **NO_PASS**.

---

## 6. Decision

```
DECISION = NO_PASS
DECISION_CODE = DEC-NOPASS-CONTRADICTION
```

**Reasoning.** The gate cannot certify a refund decision — in either direction — over a contradicted evidence chain. Contradiction blocks PASS absolutely. The verdict is not HOLD because the contradicting evidence exists; it is also not a winner-selection between the parties because the gate has no doctrinal authority to choose between two custody-sealed signed sources.

---

## 7. Output Packet

**Contradicted Evidence Dispute Packet** — emitted in NO_PASS state, carries: the merchant's usage logs (custody-sealed), the customer's travel records (custody-sealed), the geography mismatch between email-on-file identity and usage-on-network identity, the absence of an independent identity anchor (device attestation, signed login challenge), and a recommendation: **ESCALATED** — the resolution requires a higher-authority process (network arbitration, regulator, or court) that has the authority to weigh the contradicting evidence, which the doctrine does not.

---

## 8. System Boundary

VYRDON does **not** claim:

- to know who is right — the merchant or the customer
- that the customer's affidavit is dispositive — only that it is signed under penalty of perjury, which is a custody seal in the legal-forum sense
- that the merchant's logs are fraudulent — only that they are contradicted by an independent source the merchant did not have visibility into
- that this case has a verdict the system should produce in the future — the doctrinal verdict is NO_PASS; the resolution authority sits outside the system

VYRDON **does** claim:

- that the doctrine refuses to pick a winner when evidence contradicts
- that the most controversial property of RootPass — not picking a winner — is also its most honest property
- that the missing safeguard (independent identity anchor at usage time) would have prevented the contradiction from arising in the first place

---

## 9. Audit Note

Reviewer notes:

- A reviewer with access to the merchant's logs and the customer's travel records can independently confirm the contradiction; both sources are internally consistent and mutually contradictory.
- The case demonstrates that NO_PASS is not always a finding of fault; it is sometimes a finding that the evidence chain itself does not support a finality decision.
- A subsequent stage (issuer arbitration, network ruling, small-claims court) may resolve the contradiction by introducing evidence the gate did not have access to (cardholder-present challenge after the fact, biometric on file, device fingerprint history).

Maturity: **REVIEW-READY**. The case is reproducible from public-domain card-network dispute mechanics; no real party, merchant, or network is referenced.

---

# Detailed Case Record (transaction-decision form)

## Scenario

A consumer purchases a digital service from an online merchant — a 30-day subscription to a software-as-a-service product. The customer initiates a refund dispute with the issuing card network, claiming the service was never delivered. The merchant disputes the chargeback, claiming the service was delivered and used. The payment processor sits between the two and must produce a final decision: release the refund or deny it. The processor's internal "decision engine" recommends DENY because the merchant's records show the customer logged in and used the service. The customer's bank insists APPROVE because the customer signed a chargeback affidavit.

RootPass is invoked as the gate. The gate's job is not to determine "who is right" — it is to evaluate whether the evidence chain is internally consistent enough to PASS a refund decision in either direction.

---

## Visible Claim

```
claim_type: REFUND_DISPUTE_RESOLVED
claim_status: RESOLVED
claim_source: payment_processor_decision_engine
transaction_id: REF-2024-12-04471
disputed_amount: $89.00 USD
currency: USD
merchant: merchant_saas_3344
customer: cust_acct_77291
issuing_bank: bank_issuer_us
network: card_network_v
processor_recommendation: DENY (claim refund unwarranted)
customer_assertion: APPROVE (claim service never delivered)
claimed_action: RECORD_RESOLUTION_AS_FINAL
timestamp: 2024-12-04T13:00:00Z
```

---

## Required Root

For this claim to PASS in either direction (refund APPROVED or refund DENIED), the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| payment_completion_proof | YES | Card payment was authorized, captured, and settled |
| service_provisioning_proof | YES | The subscription was provisioned to an identity that matches the cardholder |
| service_usage_evidence | YES | Logs showing whether the service was accessed during the subscription period |
| customer_affidavit | YES | Customer's signed dispute statement |
| merchant_dispute_response | YES | Merchant's signed response to the dispute with supporting evidence |
| identity_consistency | YES | The identity that used the service matches the cardholder identity |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| payment_completion_proof | PROVIDED | Card auth `AUTH-12-04-77291` approved at 2024-11-04T08:15:00Z; capture `CAP-12-04-77291` at 2024-11-04T08:15:42Z; settlement file confirms $89.00 settled on 2024-11-05 |
| service_provisioning_proof | PROVIDED | Subscription `SUB-12-04-3344` provisioned to email `customer@example_redacted.tld` at 2024-11-04T08:16:11Z; provisioning event signed by merchant identity service |
| service_usage_evidence | CONTRADICTED | Merchant logs show 14 sessions between 2024-11-04 and 2024-11-22 from IP addresses in country A, user-agent string consistent across all sessions. Customer affidavit states the customer was abroad (country B) for the entire subscription period and never logged in. Customer provides flight records and country-B hotel records covering 2024-11-03 through 2024-11-25. |
| customer_affidavit | PROVIDED | Signed dispute statement filed 2024-12-01; customer attests under penalty of perjury that service was never accessed and no household member or authorized user accessed it on their behalf |
| merchant_dispute_response | PROVIDED | Merchant filed dispute response 2024-12-02 with session logs, IP records, and a screenshot of the most recent session activity |
| identity_consistency | CONTRADICTED | Email on subscription matches cardholder. IP addresses on usage logs (country A) do not match cardholder's verified billing address (country B). User-agent consistent with a single device that the customer denies owning. |

---

## Missing Evidence

No root elements are absent. However, two elements (`service_usage_evidence`, `identity_consistency`) are contradicted by the customer's records rather than confirmed.

The contradiction is not resolvable from the evidence on file alone. Either:
- (a) the merchant's logs are accurate and the customer is committing dispute fraud, or
- (b) the customer's records are accurate and the merchant's logs reflect account takeover / credential compromise by a third party.

Both possibilities are consistent with parts of the evidence chain. Neither is consistent with the whole.

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Was the card payment captured and settled? | YES |
| Was the subscription provisioned to the cardholder's email? | YES |
| Did the merchant's records show usage? | YES — 14 sessions |
| Do the merchant's usage records match the cardholder's geography? | NO — sessions from country A; cardholder verified in country B |
| Does the customer's affidavit support the merchant's usage records? | NO — customer denies all usage; provides flight + hotel records placing them outside country A |
| Did the merchant verify the session-time identity against a strong second factor (MFA, device binding, biometric)? | NO — sessions authenticated by email + password only; no MFA on the subscription tier |
| Has the merchant disclosed prior account-takeover incidents on similar subscription tiers? | NOT EVALUATED — no public record submitted as evidence |
| Are both the merchant response and the customer affidavit signed and timely? | YES |

**Contradiction count: 2** (service_usage_evidence contradicted by customer records; identity_consistency contradicted by geography mismatch)

The gate cannot determine which signed statement is accurate. Both are internally consistent. They are mutually inconsistent.

---

## RootPass Decision

```
root_valid       = TRUE   (payment, provisioning, customer affidavit, merchant response all present)
gate_valid       = TRUE   (procedural gates met — dispute filed within network window)
evidence_valid   = FALSE  (service_usage_evidence and identity_consistency contradicted)
certified_valid  = FALSE  (no independently certified record resolves the contradiction)
contradiction    = TRUE   (two contradictions detected, neither resolvable from current evidence)

DECISION = NO_PASS
```

Contradiction blocks PASS absolutely. NO_PASS here does **not** mean "the customer's claim is denied" — it means "the resolution claim (`RESOLVED`) cannot be accepted as final in its current form." The payment processor's recommendation (`DENY`) and the customer's assertion (`APPROVE`) are both rejected as final outputs; the gate refuses to certify the resolution until the contradiction is resolved by additional evidence.

This is the most important behavioural difference between RootPass and conventional dispute systems: the methodology does not pick a winner. It blocks finality until contradiction is removed.

---

## Enforcement Result

```
action: BLOCK_RESOLUTION_FINALITY
status: NO_PASS
reason: usage_evidence_contradiction + identity_consistency_contradiction
detail_a: merchant_session_logs (14 sessions, country A) contradict customer_affidavit + travel_records (country B)
detail_b: subscription_email matches cardholder; session geography does not
enforcement_type: dispute_resolution_block
next_action_1: escalate_to_human_review (network's dispute arbitration tier)
next_action_2: require_additional_evidence (merchant MFA logs if any; customer device binding logs from issuer; ISP IP geolocation timeline)
next_action_3: hold_disputed_funds in escrow pending arbitration
secondary_action: flag_merchant_for_authentication_pattern_review (single-factor auth on session-time identity for subscription products is a systemic risk signal)
anomaly_recorded: YES
circuit_breaker_check: anomaly_count incremented (current: 4/10 threshold)
```

The disputed amount is held in escrow under the timelock contract. The case is escalated to the network's human-review tier with a complete audit trail of every contradiction the gate detected. The merchant is flagged for systemic review (not penalized) on the authentication pattern. If a human-review arbitrator obtains additional evidence that resolves either contradiction, the case can be re-submitted to the gate for re-evaluation.

The customer is notified that the dispute is **not denied** — it is **held pending additional evidence**. This is materially different from the conventional outcome where one party loses by default.

---

## Institution Relevance

This case demonstrates the most controversial property of RootPass when applied to dispute resolution: **the gate refuses to declare a winner when the evidence chain is contradicted**. Conventional dispute systems are forced to resolve every dispute (typically by network rule, on a deadline) and therefore must pick a side. RootPass does not pick; it blocks finality and demands more evidence.

The institutional implication is significant. RootPass does not eliminate dispute resolution; it forces the resolution to be evidence-grounded rather than deadline-grounded. For institutions that operate as the issuer, the merchant, or the payment processor, this changes the operating model:

- **Issuers** must collect strong session-time evidence (device binding, geolocation, MFA proofs) and submit it as part of the dispute, not just the cardholder affidavit.
- **Merchants** must instrument authentication strongly enough that session-time identity is independently provable, not just session-time access.
- **Processors** must surface contradictions explicitly to both parties and to the network's arbitration tier, rather than silently picking a side under network rule.

This case also demonstrates the **anomaly counter** in dispute context: a single contradicted dispute is recorded once. A pattern of contradicted disputes against the same merchant or the same authentication tier trips the circuit breaker, halting automated dispute processing on that surface until the systemic issue is reviewed.

---

## Redaction Note

All identifiers in this case are anonymized. Card auth/capture/settlement references, subscription IDs, dispute IDs, and country names are illustrative. The `country A` / `country B` framing is used deliberately to avoid implying any specific jurisdictional context. No real customer, merchant, network, issuer, or processor is referenced.
