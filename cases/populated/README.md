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

## Authoring style

If you add a new populated case, follow the 9-section structure exactly. Cases that omit a section, mix in weak language (see the forbidden list in `vyrdon-methodology/docs/ROOT_LANGUAGE.md` §6), or relabel verdicts to fit a desired outcome do not belong in this directory.

The math and code blocks in section 5 must be defensive governance logic only:
- The "What systems do today (insufficient)" block shows the gap, not the attack
- The "Missing safeguard (not attack code)" block names the controls that should have existed, with `pass` bodies — it is a checklist, not an implementation
- The "Right defensive control" block shows the safeguard composed correctly

No exploit steps, no malware instructions, no bypass workflow under any circumstances.

See `vyrdon-portfolio/docs/AUDIT_READINESS.md` for the cross-repo maturity matrix that records each populated case.
