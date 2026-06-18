# vyrdon-cases

Applied evidence layer of the VYRDON system. Real-shaped transaction scenarios traced against the four-pillar law, with PASS / HOLD / NO_PASS decisions and enforcement outcomes.

Cases here are the public, traceable demonstrations of the methodology. They are **not** legal opinions, financial audit results, or accusations against any party.

---

## What is in this repo

- **`cases/populated/`** — fully populated cases (the load-bearing content of this repo):
  - [`CASE-001-PASS.md`](cases/populated/CASE-001-PASS.md) — marketplace escrow, all 5 roots provided, no contradiction → PASS → release payout
  - [`CASE-002-HOLD.md`](cases/populated/CASE-002-HOLD.md) — cross-border PSP remittance, recipient confirmation missing + status contradiction → HOLD
  - [`CASE-003-NO_PASS.md`](cases/populated/CASE-003-NO_PASS.md) — exchange withdrawal, ledger contradiction (prior withdrawal un-decremented) → NO_PASS
- **`cases/by-room/`** — cases from the perspective of each control room (campaign, commercial, evidence, market, operations)
- **`markets/`** — recurring failure modes and root patterns by market (banking, escrow, exchanges, marketplaces, payments, remittance, treasury)
- **`patterns/`** — short failure-pattern one-liners (payout-not-received, refund-claimed-no-root, settlement-status-contradiction, etc.)
- **`failures/`** — failure modes the case framework surfaces
- **`registry/`** — case-related type indices
- **`docs/`** — case scope, case template guide, anonymization standards
- **`tests/`** — case structure validation (placeholder)
- **`open-review/`** — case challenge protocol

---

## Read in this order

1. [`docs/CASE_SCOPE.md`](docs/CASE_SCOPE.md) — what cases are and are not
2. [`docs/CASE_TEMPLATE_GUIDE.md`](docs/CASE_TEMPLATE_GUIDE.md) — the 8-section case structure
3. [`cases/populated/CASE-001-PASS.md`](cases/populated/CASE-001-PASS.md) — PASS example
4. [`cases/populated/CASE-002-HOLD.md`](cases/populated/CASE-002-HOLD.md) — HOLD example
5. [`cases/populated/CASE-003-NO_PASS.md`](cases/populated/CASE-003-NO_PASS.md) — NO_PASS example

---

## Maturity

| Artifact | Status |
|----------|--------|
| Case template | **REVIEW-READY** |
| CASE-001, CASE-002, CASE-003 | **REVIEW-READY** (anonymized, all fields complete) |
| Market-specific patterns | **REFERENCE** (pattern descriptions, no populated case data per market yet) |
| Room-specific cases | **REFERENCE** (structural descriptions) |

Bank wire / payout-not-received / refund-dispute cases exist in short form in [`vyrdon-rootpass-proof/cases/`](https://github.com/VYRDON111/vyrdon-rootpass-proof/tree/initial-build/cases) and are being populated to CASE-001-style depth.

Full matrix: [`vyrdon-portfolio/docs/AUDIT_READINESS.md`](https://github.com/VYRDON111/vyrdon-portfolio/blob/initial-build/docs/AUDIT_READINESS.md).

---

## Submitting a case

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the case structure (public record, claimed state, required root, contradictions, RootPass decision, enforcement simulation, sources, disclaimer). All cases must use only public sources and follow the anonymization standards.

---

## Family of repos

| Repo | Role |
|------|------|
| [vyrdon-portfolio](https://github.com/VYRDON111/vyrdon-portfolio) | Front door — system map, audit-readiness, positioning |
| [vyrdon-methodology](https://github.com/VYRDON111/vyrdon-methodology) | Doctrine — four-pillar law, decision model, frozen lexicon |
| [vyrdon-rootpass-proof](https://github.com/VYRDON111/vyrdon-rootpass-proof) | Proof — TLA+, Circom, Solidity |
| [vyrdon-mechanism](https://github.com/VYRDON111/vyrdon-mechanism) | Enforcement — escrow, timelock, authority separation |
| **vyrdon-cases** (this repo) | Applied evidence — populated cases, market patterns |
| [vyrdon-technology](https://github.com/VYRDON111/vyrdon-technology) | How the system is built — domains, planes, services |
| [vyrdon-registry](https://github.com/VYRDON111/vyrdon-registry) | Taxonomy — decision codes, claim types, schemas |
| [vyrdon-commercial](https://github.com/VYRDON111/vyrdon-commercial) | Buyer language — segments, pilot model, comparisons |
| [vyrdon-open-review](https://github.com/VYRDON111/vyrdon-open-review) | Challenge — help-wanted, submissions, AI room |

---

## License

[Apache 2.0](LICENSE). Commercial licensing available separately.
