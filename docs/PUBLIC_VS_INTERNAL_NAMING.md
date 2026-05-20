# Public vs. Internal Naming

## Rule

Internal operating names are not public product language.

## Internal names — stay internal

These names exist in code, infrastructure, ops docs, migration notes, and internal architecture docs. They do not lead public repos or public copy.

| Internal Name | Where It Stays |
|---|---|
| VYRDX | code, infra, ops docs, internal architecture |
| VXStation | code, infra, internal build environment |
| KITTY | code, infra, internal build environment |
| ASUS | code, infra, authority environment |
| ConsoleLab | code, infra, internal review surface |
| control-room | code, infra, operating review surface |
| runtime shell names | code, infra only |
| machine/path names | code, infra only |

## Public names — lead with these

| Public Name | Purpose |
|---|---|
| VYRDON | The system |
| RootPass | The proof gate |
| Methodology | How the protocol works |
| Mechanism | How the gate works |
| Proof | What is proved |
| Cases | What was tested |
| Registry | What is recorded |
| Open Review | How the system is challenged |
| Technology | What the system is built with |
| Portfolio | Organizational overview |

## Translation table

| Internal | Public |
|---|---|
| VYRDX | RootPass engine / proof gate |
| VXStation | review surface / operator review |
| KITTY | internal build environment |
| ASUS | authority environment |
| ConsoleLab | discussion and evidence review surface |
| control-room | operating review surface |
| runtime | execution system |
| seal | law / non-negotiable constraints |
| attestation | evidence proof / signed evidence |
| cutover | transition / deployment phase |

## Enforcement

If a public sentence uses an internal name as its lead framing, rewrite it using the public name instead. Internal names may appear in technical context where their function is being explained, but they must not be the headline, hero text, or first impression.
