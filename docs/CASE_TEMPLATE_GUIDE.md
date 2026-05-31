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

Reference: `cases/populated/CASE-004-PASS.md` through `cases/populated/CASE-010-NO_PASS.md` (skip CASE-001 to CASE-003 — those use the older 8-step form).

A **Detailed Case Record** (transaction-decision form, binary fields) may be appended to a 9-section case when there is a specific anonymized scenario with concrete provided/missing evidence. Doctrine-form cases (CASE-007 to CASE-010) use the 9-section form alone. Field mapping rules are in `cases/populated/README.md` ("Field mapping between the two forms").

---

## Legacy form: 8-step transaction record (CASE-001 to CASE-003 only)

The first three cases (CASE-001 / CASE-002 / CASE-003) were authored under an older 8-step structure and are kept in that form for historical traceability. Do not use this form for new cases.

1. **Public record** — facts from public sources
2. **Claimed state** — what verification state is being claimed
3. **Required root** — what roots are needed to evaluate
4. **Contradictions** — conflicts found in the evidence
5. **RootPass decision** — PASS, NO_PASS, or HOLD with reasoning
6. **Enforcement simulation** — what would happen if the decision were enforced
7. **Sources** — all sources used
8. **Disclaimer** — scope and limitations

The 8-step form maps onto the 9-section form (Public record ⊂ Surface Signal, Claimed state ⊂ Surface Signal, Required root ⊂ Root Failure + Root Map, Contradictions ⊂ Root Map row, RootPass decision ≡ Decision, Enforcement simulation ⊂ Output Packet, Sources / Disclaimer ⊂ System Boundary + Audit Note). A future migration may rewrite CASE-001 to CASE-003 into the 9-section form; until then, both forms coexist.

See `vyrdon-rootpass-proof/cases/live-court/case-template/` for the original 8-step template, kept for reference.

---

## Which form to use

| Situation | Form |
|---|---|
| Any new case | 9-section doctrine form |
| Editing CASE-001 to CASE-003 | Stay in 8-step until a deliberate migration commit |
| Doctrine case with no specific transaction | 9-section only (no Detailed Case Record) |
| Anonymized transaction case with concrete fields | 9-section + Detailed Case Record |

If you are unsure, default to the 9-section doctrine form.
