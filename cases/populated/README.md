# Populated Cases

Fully populated, anonymized case studies that exercise the four-pillar law (ROOT / GATE / VALID / CERTIFIED) on realistic transaction and proof scenarios. Every case in this directory uses the **9-section Root Language structure** defined in `vyrdon-methodology/docs/ROOT_LANGUAGE.md`:

```
1. Case title
2. Surface signal
3. Root failure
4. Missing safeguard
5. VYRDON Root map        (with math + bad-code + missing-code + right-code)
6. Decision               (PASS / HOLD / NO_PASS)
7. Output packet
8. System boundary        (what VYRDON does NOT claim)
9. Audit note             (what would strengthen the review)
```

These are **not legal opinions** and **not accusations**. They are illustrations of how the doctrine produces PASS / HOLD / NO_PASS under realistic but anonymized inputs. The redaction note in each case spells out which fields are sanitized. The math and code blocks in each Root map are **defensive governance logic only** — no exploit steps, no malware instructions, no bypass workflow.

## Index

| Case | Decision | Domain | Output packet | Doctrine signal |
|---|---|---|---|---|
| [CASE-001](CASE-001-PASS.md) | PASS | Marketplace escrow | Escrow release record | All four pillars TRUE; no contradiction → PASS |
| [CASE-002](CASE-002-HOLD.md) | HOLD | Cross-border PSP remittance | Hold record | Missing recipient credit (MISSING outranks contradiction) → HOLD |
| [CASE-003](CASE-003-NO_PASS.md) | NO_PASS | Digital exchange withdrawal | Withdrawal return record | Ledger contradiction (prior withdrawal not reflected) → NO_PASS |
| [CASE-004](CASE-004-PASS.md) | PASS | Corporate-treasury bank wire | Wire Finality Packet | All SWIFT legs proved end-to-end with dual-control sender authorization → PASS |
| [CASE-005](CASE-005-NO_PASS.md) | NO_PASS | Merchant payout (ACH return) | Payout Failure Reconciliation Packet | Platform `COMPLETED` contradicted by originating bank's R03 return file → NO_PASS |
| [CASE-006](CASE-006-NO_PASS.md) | NO_PASS | Card-network refund dispute | Contradicted Evidence Dispute Packet | Merchant usage logs contradicted by customer travel records; gate refuses to pick a winner → NO_PASS |
| [CASE-007](CASE-007-HOLD.md) | HOLD | "Nothing happened" legal failure | Negative Claim Proof Packet | Negative-state seals MISSING; system structurally positive-only → HOLD |
| [CASE-008](CASE-008-NO_PASS.md) | NO_PASS | Zeus-class banking trojan | Transfer Protection Review Packet | Machine green TRUE while human red seal FALSE for the second transfer → NO_PASS |
| [CASE-009](CASE-009-HOLD.md) | HOLD | Crypto-theft legal void | Wallet Loss Evidence Packet | Ownership binding + pre-loss control proof + sealed artifact all MISSING → HOLD |
| [CASE-010](CASE-010-NO_PASS.md) | NO_PASS | Ransomware coverage denial | Coverage Readiness Packet | Policy-trigger match FALSE; portal status contradicts policy match → NO_PASS |

## Doctrine coverage

| Outcome | Count | Cases |
|---|---|---|
| PASS | 2 | CASE-001, CASE-004 |
| HOLD | 3 | CASE-002, CASE-007, CASE-009 |
| NO_PASS | 5 | CASE-003, CASE-005, CASE-006, CASE-008, CASE-010 |

Within each outcome class:

**HOLD class** (Missing Root):
- CASE-002 — recipient credit not yet received (timing gap; recoverable)
- CASE-007 — negative-state seals never produced (architectural gap; not recoverable without changing host system)
- CASE-009 — ownership binding never sealed (artifact gap; partially recoverable)

**NO_PASS class** (False Root or Contradiction):
- CASE-003 — internal ledger contradiction within one platform's own state
- CASE-005 — contradiction across two related systems (platform ledger vs. originating bank return file)
- CASE-006 — contradiction across two parties' signed evidence (merchant usage vs. customer travel records)
- CASE-008 — contradiction between machine-green and human red seal within a single session
- CASE-010 — contradiction between portal status and policy-trigger match

## What these cases prove (and don't prove)

These cases prove that the doctrine **produces the right verdict** on realistic inputs. They do not prove that any specific institution applies the doctrine correctly today, and they do not certify any real-world transaction. The cases are reproducible: anyone reading the surface signal, root failure, and Root map should arrive at the same decision the case records.

## Doctrine signal: fraud through broken route, not stolen key

A pattern that runs through CASE-005 through CASE-010: **fraud and loss do not require a stolen key**. They more often happen through a **broken route** — a path from surface to root that fails one of the five Route Rule proofs (`vyrdon-mechanism/mechanism/ROUTE_LAW.md` §5: authorized route, valid scope, custody seal, evidence path, authority/root proof) while a valid credential is still in possession of the right party.

The Window Principle (`mechanism/ROUTE_LAW.md` §2) names this: *a key is not enough if the route is broken; an attacker does not need the key to the house if the window is open*.

Case-by-case applications:

| Case | The "valid key" present | The broken route |
|---|---|---|
| CASE-005 | Platform's `COMPLETED` status (valid surface label) | Reconciliation timing — the return file is not pulled before the platform marks the payout complete |
| CASE-006 | Merchant's signed usage logs and customer's signed travel records (both valid) | No independent identity anchor — the gate cannot rule one custody-sealed source over the other |
| CASE-007 | The system's positive-event log (valid for everything that happened) | No negative-state seal — the system structurally cannot prove that nothing happened |
| CASE-008 | The authenticator token (valid for one transfer) | Session lifecycle and per-event authority — the same token is reused for a second transfer the human did not authorize |
| CASE-009 | On-chain wallet movements (valid as raw blockchain data) | Identity binding and pre-loss control — the wallet is not custody-sealed to a legal identity, so the data is not legal evidence |
| CASE-010 | Loss event (valid as a real incident) | Policy-trigger match — the loss type does not match the policy's damage definition |

The doctrine signal across the case set: **the gate must seal the route, not only check the key**. The Route Rule's five proofs are the route's seals; Code Hash Seal (`mechanism/CODE_HASH_SEAL.md`) extends the principle into the build layer so that even the **verifier** that checks routes is itself sealed against substitution.

## Two-layer protection across the case set

| Sealing layer | What is sealed in these cases | Root Language artifact |
|---|---|---|
| **Surface seal** (the door) | The five Route Rule proofs at intake: authorized route, valid scope, custody seal, evidence path, authority/root proof. CASE-005 through CASE-010 all turn on one or more of these. | `mechanism/ROUTE_LAW.md` |
| **Build seal** (the bricks) | The code that runs the verifiers, validators, schemas, decision engines, and gate compositions. None of the cases turns on this layer directly, because the cases describe failures at the gate, not failures of the gate. A case in which the verifier itself was substituted would turn on Code Hash Seal. | `mechanism/CODE_HASH_SEAL.md` |

The cases in this directory are the **surface-seal exercise**. They establish that the doctrine produces correct verdicts when the verifier is intact. The build-seal exercise (a case in which the verifier itself is the failure point) is left as a follow-up — naming the failure class is enough for the current PR series.

## Authoring style

If you add a new populated case, follow the 9-section structure exactly. Cases that omit a section, mix in weak language (see the forbidden list in `vyrdon-methodology/docs/ROOT_LANGUAGE.md` §6), or relabel verdicts to fit a desired outcome do not belong in this directory.

The math and code blocks in section 5 must be defensive governance logic only:
- The "What systems do today (insufficient)" block shows the gap, not the attack
- The "Missing safeguard (not attack code)" block names the controls that should have existed, with `pass` bodies — it is a checklist, not an implementation
- The "Right defensive control" block shows the safeguard composed correctly

No exploit steps, no malware instructions, no bypass workflow under any circumstances.

See `vyrdon-portfolio/docs/AUDIT_READINESS.md` for the cross-repo maturity matrix that records each populated case.

---

## Public review

Each populated case is open to public challenge through the review console.

- Case review guide: [`vyrdon-open-review/review/CASE_REVIEW_GUIDE.md`](https://github.com/VYRDON111/vyrdon-open-review/blob/initial-build/review/CASE_REVIEW_GUIDE.md)
- Case-challenge issue template: [`vyrdon-open-review/.github/ISSUE_TEMPLATE/case-challenge.yml`](https://github.com/VYRDON111/vyrdon-open-review/blob/initial-build/.github/ISSUE_TEMPLATE/case-challenge.yml)
- Counterexample issue template: [`vyrdon-open-review/.github/ISSUE_TEMPLATE/counterexample.yml`](https://github.com/VYRDON111/vyrdon-open-review/blob/initial-build/.github/ISSUE_TEMPLATE/counterexample.yml)
- Per-case honest limits: [`vyrdon-open-review/review/CASE_LIMITATIONS.md`](https://github.com/VYRDON111/vyrdon-open-review/blob/initial-build/review/CASE_LIMITATIONS.md)

These cases are review artifacts, not official legal/financial determinations. CASE-007 through CASE-010 are doctrine forms of well-documented failure classes; they cite no specific incident and name no specific real party. Counterexamples are welcome and may be submitted via the counterexample issue template.
