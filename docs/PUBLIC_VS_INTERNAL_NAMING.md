# Public vs. Internal Naming

## Rule

Internal operating names are not public product language.
When public value is being described, prefer function over codename.

## Public-safe system names

These names carry architectural meaning and may appear in public repos:

| Name | Public Meaning |
|------|---------------|
| VYRDON | Parent proof system and institutional structure |
| VYRDX | Proof-enforced transaction gate |
| RootPass | Institutional proof logic inside the gate |
| ConsoleLab | Control room, review, and evidence discussion surface |
| ASUSX | Authority and signing plane |
| VYRDEN | Intelligence and reasoning layer |

## Internal names — stay internal

These names exist in code, infrastructure, ops docs, and internal architecture. They must not lead public repos or public copy.

| Internal Name | Where It Stays | Public Replacement |
|---|---|---|
| ASUS | code, infra, authority environment | ASUSX (authority plane) |
| DELL | code, infra, runtime environment | execution node |
| KITTY | code, infra, terminal environment | terminal layer (or remove) |
| VXStation | code, infra, internal monitoring | observer layer (if retained) |
| DEBLOOD | code, infra, internal service | remove — use function description |
| node-blood | code, infra, internal service | gate reference service |
| t79 | code, infra, operator paths | remove entirely |
| machine IPs | code, infra only | remove entirely |
| hostnames | code, infra only | remove entirely |
| droplet names | code, infra only | cloud instance / compute node |
| shell names | code, infra only | terminal layer |
| runtime shell names | code, infra only | execution system |
| cutover shorthand | code, infra only | transition / deployment phase |

## Translation table

| Internal | Public |
|---|---|
| ASUS | ASUSX / authority plane |
| DELL | execution node |
| KITTY | terminal layer (do not reference publicly) |
| VXStation | observer layer (if still needed) |
| DEBLOOD | local intelligence runtime / VYRDEN local runtime |
| node-blood | gate reference service |
| ConsoleLab | control room / review surface |
| control-room | operating review surface |
| runtime | execution system |
| seal | law / non-negotiable constraints |
| attestation | evidence proof / signed evidence |
| cutover | transition / deployment phase |

## Enforcement

If a public sentence uses an internal name as its lead framing, rewrite it using the public name or function description instead. Internal names may appear in technical context where their function is being explained, but they must not be the headline, hero text, or first impression.
