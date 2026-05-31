# Case Template Guide

This repo holds populated cases in two compatible forms. Use this guide to know which form a new case should use.

---

## Canonical form (current): 9-section doctrine form

All cases authored from CASE-004 onward use the **9-section doctrine form**. Use this for any new case.

1. **Case Title** — short identifier and verdict in the title
2. **Surface Signal** — what the surface looks like (UI status, portal label, dashboard claim)
3. **Root Failure** — which root is FALSE / MISSING / contradicted, and why
4. **Missing Safeguard** — which doctrine-level control should have existed at the gate (cross-link to `vyrdon-mechanism/mechanism/MISSING_SAFEGUARDS.md`)
5. **VYRDON Root Map** — full four-pillar table (ROOT / GATE / VALID / CERTIFIED + Contradiction) with evaluation per pillar. For doctrine-form cases without an appended Detailed Case Record (CASE-007 to CASE-010), §5 additionally carries `### Math` and `### Code` subsections — the math subsection shows the four-pillar arithmetic on the case's inputs, and the code subsection shows three blocks (insufficient gate code as written, the missing-safeguard code that should have been there, and the corrected right-control code) framed as **defensive governance logic only** (no exploit steps, no attack workflow). Cases that carry a Detailed Case Record (CASE-004 to CASE-006) may omit the math+code subsections because the appended Detailed Case Record covers the same illustrative ground in transaction-decision form.
6. **Decision** — verdict (PASS / HOLD / NO_PASS), decision code (DEC-PASS / DEC-HOLD-MISSING_ROOT / DEC-NOPASS-FALSE_ROOT / DEC-NOPASS-CONTRADICTION), and reasoning
7. **Output Packet** — the audit-bound output record the system produces (verdict, code, blocking roots, action)
8. **System Boundary** — what this case does and does not claim
9. **Audit Note** — what an external auditor should look at first

Reference: `cases/populated/CASE-004-PASS.md` through `cases/populated/CASE-010-NO_PASS.md` (skip CASE-001 to CASE-003 — those use the older 10-section transaction-record form).

A **Detailed Case Record** (transaction-decision form, binary fields) may be appended to a 9-section case when there is a specific anonymized scenario with concrete provided/missing evidence. Doctrine-form cases (CASE-007 to CASE-010) use the 9-section form alone. Field mapping rules are in `cases/populated/README.md` ("Field mapping between the two forms").

---

## Legacy form: 10-section transaction record (CASE-001 to CASE-003 only)

The first three cases (CASE-001 / CASE-002 / CASE-003) were authored under an older 10-section transaction-record structure and are kept in that form for historical traceability. Do not use this form for new cases.

1. **Scenario** — the operational scenario being evaluated
2. **Visible Claim** — what the surface state is claiming
3. **Required Root** — which roots must be TRUE for the claim to PASS
4. **Provided Evidence** — evidence the actor offered to support the claim
5. **Missing Evidence** — required evidence that was not produced
6. **Contradiction Check** — contradictions detected across the evidence set
7. **RootPass Decision** — verdict (PASS / HOLD / NO_PASS) with reasoning
8. **Enforcement Result** — what the gate would do if the decision were enforced
9. **Institution Relevance** — which institutional class this case applies to
10. **Redaction Note** — which fields are anonymized and to what degree

The 10-section form maps onto the 9-section form: Scenario + Visible Claim ⊂ Surface Signal; Required Root + Missing Evidence ⊂ Root Failure + Root Map; Provided Evidence ⊂ Root Map; Contradiction Check ⊂ Root Map (Contradiction row); RootPass Decision ≡ Decision; Enforcement Result ⊂ Output Packet; Institution Relevance + Redaction Note ⊂ System Boundary + Audit Note. A future migration may rewrite CASE-001 to CASE-003 into the 9-section form; until then, both forms coexist.

A **separate 8-step template** lives in `vyrdon-rootpass-proof/cases/live-court/case-template/` (Public record / Claimed state / Required root / Contradictions / RootPass decision / Enforcement simulation / Sources / Disclaimer). That template is **not** what CASE-001 to CASE-003 use — it is an older live-court scaffold kept in the proof repo for reference and is not authoritative for this repo's populated cases.

---

## Which form to use

| Situation | Form |
|---|---|
| Any new case | 9-section doctrine form |
| Editing CASE-001 to CASE-003 | Stay in 10-section transaction-record form until a deliberate migration commit |
| Doctrine case with no specific transaction | 9-section only (no Detailed Case Record) |
| Anonymized transaction case with concrete fields | 9-section + Detailed Case Record |

If you are unsure, default to the 9-section doctrine form.

---

## Decision code selection (primary-finding rule)

A case may have both `ROOT = FALSE` and `Contradiction = TRUE`. In that case, `DECISION_CODE` is chosen by the **primary doctrinal finding** — the failure that the case turns on — not by which line you read first.

| Primary finding | Decision code |
|---|---|
| Missing required root (one or more roots are MISSING) | `DEC-HOLD-MISSING_ROOT` |
| Fabricated, substituted, or otherwise FALSE root (the gate is being asked to PASS on an invalid root) | `DEC-NOPASS-FALSE_ROOT` |
| Contradiction between two custody-sealed sources (the gate cannot pick a winner; both seals are intact, the values disagree) | `DEC-NOPASS-CONTRADICTION` |

Worked examples in the populated set:

- **CASE-005** — both False Root (truncated account number) and Contradiction (platform ledger COMPLETED vs. originating bank RETURNED). The case turns on the contradiction between two custody-sealed sources, so `DEC-NOPASS-CONTRADICTION` is used.
- **CASE-008** — both False Root (fabricated authority — token re-use treated as new human seal) and Contradiction (machine green TRUE vs. human red seal FALSE for the second transfer). The case turns on the fabricated authority, so `DEC-NOPASS-FALSE_ROOT` is used.
- **CASE-010** — both False Root (damage-definition-match FALSE) and Contradiction (portal status contradicts policy-trigger match). The case turns on the policy-mismatch False Root, so `DEC-NOPASS-FALSE_ROOT` is used.

The §6 reasoning paragraph in each case names the primary finding explicitly, so a reviewer can verify the decision-code choice without inferring intent.
