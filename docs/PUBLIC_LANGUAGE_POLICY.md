PUBLIC LANGUAGE POLICY

1. Internal operating names are not public product language.
2. Public repositories must lead with method, mechanism, proof, cases, tests, failures, registry, and review.
3. Do not present VYRDON as a badge, certifier, or trust authority.
4. Do not say "we verify" or "we certify" as identity claims.
5. Prefer verbs such as:
   - model
   - test
   - constrain
   - prove
   - enforce
   - record
   - evidence
   - challenge
   - review
6. RootPass doctrine:
   TRUE ROOT = PASS
   FALSE ROOT = NO_PASS
   MISSING ROOT = HOLD
7. No root, no pass.
8. Public writing must be readable by institutions, researchers, developers, auditors, and contributors.
9. Public repos must be open to challenge, not closed as slogans.
10. If a sentence sounds like a badge claim, rewrite it as a method claim.

---

## Decision Record

### Yes

We change the names of the old internal-facing terms for the new public VYRDON layer.

### Yes

We change the tone away from "verified / certified / trusted."

### Yes

We reframe the whole org around:

- method
- mechanism
- proof
- cases
- tests
- failures
- registry
- open review

### No

We do not present the new VYRDON as a certifier.

### No

We do not lead with internal runtime names anymore.

---

## Renaming Table

| Old Internal Name | New Public Name | Use In Public Repos | Never Use In Public Headlines |
|---|---|---|---|
| ASUS | ASUSX / authority plane | Reference only as "ASUSX" or "authority plane" | Never use "ASUS" |
| DELL | Execution node | "Execution node" or "runtime node" | Never use "DELL" |
| KITTY | Terminal layer | Do not reference publicly | Never |
| VXStation | Observer layer | "Observer layer" or "read-only observer" (if retained) | Never as product name |
| DEBLOOD | Local intelligence runtime | Do not reference publicly | Never |
| node-blood | Gate reference service | Do not reference publicly | Never |
| ConsoleLab | Control room / review surface | Describe function, not name | Never as product name |
| control-room | Operating review surface | Describe function, not name | Never as product name |
| runtime | Execution system | "Execution system" or "proof engine" | Never as standalone identity |
| seal | Law / non-negotiable constraints | "Sealed law" or "constraint" | Never as badge |
| attestation | Evidence proof / signed evidence | "Signed evidence" or "evidence proof" | Never as trust claim |
| cutover | Transition / deployment phase | "Deployment phase" or "transition" | Never |

---

## Banned Words in Headlines and Hero Text

The following words must NOT appear in README headlines, hero sections, or opening lines:

- verified
- certified
- trusted
- secure
- guaranteed
- confirmed
- validated by us

These words may only appear in:

- Quoted third-party claims being evaluated
- Case descriptions (describing what someone else claimed)
- Comparative analysis (contrasting trust claims vs proof chains)
- Input claims being tested by the methodology

---

## Required Voice

### Not:
> We verify transactions.

### Yes:
> We publish the logic, constraints, and enforcement paths used to determine whether a claimed verification state can pass.

### Not:
> We certify payments.

### Yes:
> We evaluate whether the required root chain exists, whether contradictions are present, and whether PASS is legally and logically allowed under the model.
