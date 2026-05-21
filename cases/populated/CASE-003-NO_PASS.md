# CASE-003 — NO_PASS

## Decision: NO_PASS

---

## Scenario

A digital exchange processes a withdrawal request. A user claims they deposited funds, traded, and now want to withdraw profits. The exchange's internal system shows the account balance as sufficient and marks the withdrawal as "approved." However, the root chain reveals a critical contradiction in the deposit evidence.

---

## Visible Claim

```
claim_type: WITHDRAWAL_APPROVED
claim_status: APPROVED
claim_source: exchange_withdrawal_engine
transaction_id: WDR-2024-06-19441
amount: $28,750.00 USD
currency: USD
account: user_account_4418
claimed_action: PROCESS_WITHDRAWAL
timestamp: 2024-06-19T11:05:00Z
```

---

## Required Root

For this claim to PASS, the following root chain is required:

| Root Element | Required | Description |
|-------------|----------|-------------|
| deposit_proof | YES | Proof that the original deposit entered the exchange from a verified source |
| deposit_source_verification | YES | Verification that the deposit source is a known, compliant funding source |
| trade_execution_log | YES | Verifiable log of trades that produced the claimed profit |
| balance_consistency | YES | Current balance matches the sum of deposits + trades - withdrawals |
| identity_verification | YES | User passed KYC and the withdrawal account matches the depositing identity |
| withdrawal_limit_check | YES | Withdrawal does not exceed daily/monthly limits |

---

## Provided Evidence

| Evidence | Status | Detail |
|----------|--------|--------|
| deposit_proof | PROVIDED | Bank wire reference BWR-44182; $20,000 deposited 2024-05-01T09:00:00Z |
| deposit_source_verification | PROVIDED | Source bank account verified at onboarding; matches KYC identity |
| trade_execution_log | PROVIDED | 47 trades logged between 2024-05-01 and 2024-06-18; net P&L shows +$8,750 |
| balance_consistency | CONTRADICTED | See contradiction check below |
| identity_verification | PROVIDED | KYC completed; withdrawal destination matches depositing identity |
| withdrawal_limit_check | PROVIDED | $28,750 is within daily limit of $50,000 |

---

## Missing Evidence

No root elements are missing. All required elements have a status entry. However, one element is contradicted rather than confirmed.

---

## Contradiction Check

| Check | Result |
|-------|--------|
| Does the deposit amount + trade P&L equal the withdrawal amount? | NO — $20,000 + $8,750 = $28,750, which matches the withdrawal. But... |
| Are there other withdrawals or debits against this account? | YES — a previous withdrawal of $5,000 was processed on 2024-06-10 (WDR-2024-06-10-2291) |
| Does the current balance reflect the prior withdrawal? | NO — the balance still shows $28,750 as if the prior withdrawal never occurred |
| What is the true available balance? | $20,000 + $8,750 - $5,000 = $23,750 |
| Does the claimed withdrawal amount exceed the true balance? | YES — $28,750 > $23,750 by $5,000 |
| Is there a database inconsistency? | YES — the balance record was not decremented after the prior withdrawal; this is a ledger integrity failure |

**Contradiction count: 1** (balance record contradicts withdrawal history)

This is a critical contradiction. The account balance is overstated by $5,000 due to a ledger integrity failure. The withdrawal request, if approved, would release $5,000 more than the account actually holds.

---

## RootPass Decision

```
root_valid       = TRUE
gate_valid       = TRUE
evidence_valid   = TRUE (all evidence provided)
certified_valid  = FALSE (balance_consistency contradicted)
contradiction    = TRUE (ledger integrity failure)

DECISION = NO_PASS
```

Contradiction blocks PASS. The visible claim ("APPROVED", balance = $28,750) is contradicted by the withdrawal history. The exchange's internal approval is overridden by the gate decision.

The system does not care that the exchange approved the withdrawal. The system does not care that the balance field shows sufficient funds. The root chain reveals a ledger contradiction, and contradiction blocks PASS absolutely.

---

## Enforcement Result

```
action: BLOCK_WITHDRAWAL
status: NO_PASS
reason: balance_consistency_contradiction
detail: prior withdrawal WDR-2024-06-10-2291 ($5,000) not reflected in balance; true balance is $23,750, not $28,750
enforcement_type: withdrawal_block
next_action: flag_account_for_ledger_audit
secondary_action: notify_compliance_team
anomaly_recorded: YES
circuit_breaker_check: anomaly_count incremented (current: 3/10 threshold)
```

Withdrawal blocked. Account flagged for ledger audit. Compliance team notified. The exchange's "APPROVED" status is not accepted by the gate.

---

## Institution Relevance

This case demonstrates a scenario where internal approval is not sufficient for acceptance. The exchange's withdrawal engine approved the transaction based on a balance field that was incorrect. Without RootPass enforcement, the withdrawal would proceed, resulting in a $5,000 overpayment — effectively creating money from a database error.

This is the core value proposition: **execution is not truth.** The exchange executed an approval, but the approval was based on contradicted evidence. RootPass caught the ledger integrity failure that the exchange's own system missed.

Institutions affected: exchanges, trading platforms, custodial wallets, any system where balance records drive withdrawal authorization.

The contradiction also triggered an anomaly counter increment. If the exchange accumulates 10 such anomalies, the circuit breaker trips and halts all operations until a system-level audit is completed.

---

## Redaction Note

All identifiers in this case are anonymized. Account references, transaction IDs, wire references, and amounts are sanitized. No real exchange names, user identities, or financial institution identifiers appear in this document. The scenario is modeled on common exchange withdrawal patterns and does not reference any specific platform, user, or transaction. The ledger integrity failure pattern is a well-documented class of operational risk in exchange operations.
