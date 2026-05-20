# VYRDON Cases

This repository defines the VYRDON case layer.

It exists to make real-world applications of the RootPass methodology visible, testable, and challengeable.

RootPass asks not whether a transaction was said to be verified, but whether the verification itself can pass.

---

## What This Repo Proves

Methodology without cases is theory. Cases prove the methodology works on real data — and document where it fails.

Each case applies the four-pillar law to a real-world transaction scenario and produces a RootPass decision: PASS, NO_PASS, or HOLD.

```
TRUE ROOT  = PASS
FALSE ROOT = NO_PASS
MISSING ROOT = HOLD
```

No root, no pass.

---

## Repository Structure

| Path | What it contains |
|---|---|
| `docs/` | Case scope, selection policy, disclaimers, source rules, template guide |
| `markets/` | Market-specific analysis (banking, payments, exchanges, marketplaces, escrow, treasury, remittance) |
| `cases/` | Cases organized by market, claim type, root gap, contradiction, and live court |
| `patterns/` | Recurring failure patterns across markets |
| `registry/` | Case, market, claim, contradiction, and institution indexes |
| `tests/` | Case consistency and source sufficiency checklists |
| `failures/` | Bad case selection, weak source cases, overclaim risks |
| `open-review/` | Submit, challenge, or request cases |
| `sources/` | Source library and verification |

---

## Markets Covered

| Market | Key Patterns |
|---|---|
| Banking | Wire not received, settlement mismatch, authority conflict |
| Payments | Refund dispute, payout delay, merchant settlement gap |
| Exchanges | Deposit not credited, withdrawal not confirmed, balance contradiction |
| Marketplaces | Delivery dispute, escrow release conflict, multi-party contradiction |
| Escrow | Release without proof, premature release, timeout dispute |
| Treasury | Reconciliation failure, end-of-day mismatch, multi-currency gap |
| Remittance | Cross-border wire uncertainty, status contradiction, authority gap |

---

## How to Submit a Case

See `open-review/SUBMIT_A_CASE.md`.

## How to Challenge a Case

See `open-review/CHALLENGE_A_CASE.md`.

---

## License

Apache 2.0 — see [LICENSE](LICENSE)
