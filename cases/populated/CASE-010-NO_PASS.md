# CASE-010 — NO_PASS

## 1. Case Title

**Ransomware Coverage Denial — Loss Real, Policy Match False**

An organization is hit by ransomware. Files are encrypted, business is disrupted, the loss is unambiguous. The cyber insurance carrier denies coverage. The denial is not because the loss was fabricated; it is because the policy's damage definition does not match the loss type, the formal claim notice did not meet the policy's notification trigger, and the coverage-artifact archive was never produced.

---

## 2. Surface Signal

```
loss_event: TRUE
    files_encrypted: TRUE
    business_disrupted: TRUE
    ransom_note_received: TRUE
    incident_response_engaged: TRUE
policy_in_force_at_time_of_loss: TRUE
claim_filed: TRUE
policy_dashboard_status: "Active" (Green)
carrier_decision: DENIED
```

The surface is fully green up to the moment of carrier decision. Policy is active, premium is paid, dashboard says "covered," loss is real. **Reliance on inaccurate portals**: the dashboard "Green / Active" is not the same as the legal "Red / Sealed-as-payable." Coverage is not a portal status; it is a sealed match between a specific loss type and a specific policy trigger.

**Machine green is not execution authority.** The portal does not bind the carrier.

---

## 3. Root Failure

The required Root for a coverage-eligibility PASS is:

```
ROOT(coverage_eligibility) =
    loss_event_verified
  ∧ policy_trigger_matched_to_loss_type
  ∧ damage_definition_matched_under_policy_language
  ∧ formal_claim_notice_within_window
  ∧ coverage_artifact_archive_written
```

In the ransomware-denial scenario:

- `loss_event_verified` = **TRUE** — the encryption event and business disruption are evidentially clear
- `policy_trigger_matched_to_loss_type` = **FALSE** — the policy covers, say, "unauthorized data exfiltration" or "system reconstruction costs," but the actual loss is "operational downtime from encryption-only event"; the named perils do not match
- `damage_definition_matched_under_policy_language` = **FALSE** — the policy's definition of covered damage explicitly excludes the loss category that occurred (encryption-only, ransom payment, third-party recovery cost)
- `formal_claim_notice_within_window` = **FALSE** or **MISSING** — the notification was sent through the wrong channel, after the policy's notification deadline, or without the required documentation set
- `coverage_artifact_archive_written` = **MISSING** — the insured organization did not produce a coverage-readiness artifact at the time of the incident that the carrier could ingest as a single sealed object

Two of the five required roots are **FALSE** (definitive mismatches, not gaps). This is **False Root**, not Missing Root. The doctrine `FALSE ROOT → NO_PASS` applies.

---

## 4. Missing Safeguard

The safeguards that should have existed and did not:

- **`verify_policy_trigger_matches_loss`** — before relying on coverage, the insured should run a policy-trigger match check at incident time, comparing the specific loss type to the specific named perils in the policy.
- **`classify_damage_under_policy_language`** — the actual damage (encryption only, exfiltration, business interruption, regulatory fine, third-party reconstruction) should be classified against the policy's damage definitions, with the result sealed.
- **`send_formal_claim_notice`** — claim notice must be sent through the policy-specified channel, within the policy-specified window, with the policy-specified documentation, and the notice itself must be sealed (timestamp, custody binding, transmission proof).
- **`archive_coverage_artifacts`** — at the time of incident, a single coverage-readiness artifact should be archived: incident timeline, IR engagement record, encrypted-asset inventory, business-interruption measurement, and the formal claim notice — all bundled and sealed.

In conventional ransomware response, none of these are produced in audit-ready form. The portal dashboard is treated as the coverage assurance, and the formal proof is constructed retroactively (or never).

---

## 5. VYRDON Root Map

```
ROOT      = FALSE   (policy trigger and damage definition do not match the loss)
GATE      = FALSE   (the gate cannot release coverage on a mismatched root)
VALID     = MIXED   (loss event fields are present; coverage-match fields are present but False;
                     formal-notice fields are partial)
CERTIFIED = N/A    (no honest certificate can be issued under a False Root)

Contradiction = TRUE  (portal-status="Active/covered" while policy-trigger-match=FALSE)
```

### Math

```
CoverageEligibility
  = LossEvent
  × PolicyTrigger
  × DamageDefinitionMatch
  × FormalNotice
  × ArchiveProof

If DamageDefinitionMatch = 0 or FormalNotice = 0, then CoverageEligibility = 0.
```

### Code

```python
# What systems do today (insufficient)
def coverage_pass(
    files_encrypted: bool,
    business_disrupted: bool,
) -> bool:
    return files_encrypted and business_disrupted
```

```python
# Missing safeguard (not attack code)
def verify_policy_trigger_matches_loss():
    pass

def classify_damage_under_policy_language():
    pass

def send_formal_claim_notice():
    pass

def archive_coverage_artifacts():
    pass
```

```python
# Right defensive control
def coverage_pass(
    loss_event_verified: bool,
    policy_trigger_matched: bool,
    damage_definition_matched: bool,
    formal_notice_sent: bool,
    claim_archive_written: bool,
) -> bool:
    return (
        loss_event_verified
        and policy_trigger_matched
        and damage_definition_matched
        and formal_notice_sent
        and claim_archive_written
    )
```

Synthesis factor mapping:

| FinalExecutionPass factor | State in this case |
|---|---|
| MachineGreen | 1 — portal says active |
| HumanRedSeal | 0 — no carrier-side authorization was issued; the carrier denied |
| ProofComplete | 0 — policy-trigger match and damage-definition match are FALSE |
| ArchiveFinal | 0 — no coverage-readiness artifact was archived |
| CustodySeal | 0 — formal notice (if sent) was not sealed against the policy's notification trigger |

Four zeros → product is zero → not PASS. Contradiction between portal status (Green) and policy match (False) → **NO_PASS** is the verdict per the contradiction rule.

---

## 6. Decision

```
DECISION = NO_PASS
DECISION_CODE = DEC-NOPASS-FALSE_ROOT
```

**Reasoning.** This is False Root: two required roots (policy_trigger_match, damage_definition_match) are definitively FALSE under the policy's own language, and the portal status contradicts those roots. Per doctrine, FALSE ROOT → NO_PASS, and contradiction blocks PASS absolutely. The verdict is not HOLD because the policy's language is itself a sealed artifact (the policy document) against which the loss type can be compared — the gap is not "we have not yet observed" but "we have observed and the answer is no."

---

## 7. Output Packet

**Coverage Readiness Packet** — emitted in NO_PASS state in audit form, but is also the artifact that should have been produced **before** the incident as a precondition for any coverage relying on machine-green signals. Carries:

- The policy document hash (which the readiness packet is being matched against)
- The classified loss type
- The named perils in the policy and the match-or-no-match for each
- The damage definition in the policy and the match-or-no-match against the actual loss
- The formal claim notice (or the absence thereof) with transmission proof
- The auditable conclusion: which roots are TRUE, which are FALSE, which are MISSING

The packet's role is to expose the portal-versus-policy contradiction at incident time, so that the insured organization is not surprised by the denial and can either: (a) accept that the loss is not in fact covered and move to alternative remediation, or (b) negotiate scope clarification before the policy renewal.

---

## 8. System Boundary

VYRDON does **not** claim:

- that coverage will be granted — VYRDON cannot bind a carrier
- that the carrier's denial is incorrect — in many cases the denial is technically correct under policy language, and the failure is at the policy-selection stage upstream of any incident
- that the portal "Green / Active" status is fraudulent — only that it is a status signal, not a sealed coverage decision
- that this case is a substitute for policy review by an insurance broker or counsel

VYRDON **does** claim:

- that portal status is not policy match
- that the policy-trigger match and damage-definition match should be computed at incident time, not at denial time
- that the coverage-readiness artifact should be produced as a precondition of the policy, not as a forensic reconstruction afterward
- that NO_PASS in this case is a Root Language verdict on the **proof chain**, not a verdict on the loss itself

---

## 9. Audit Note

Evidence that would strengthen this review:

- A specific ransomware loss with both the policy document and the carrier's denial letter, allowing exact comparison of named-perils language to the loss type (this version of the case is generic)
- A coverage-readiness packet produced **before** the incident, against which the portal status can be audited at any point in time
- A broker-issued attestation of the policy-trigger match for a specific named-perils set
- An incident-response artifact with custody-sealed timestamps

Reconstructed-from-public-reporting elements: this case is the doctrine form of a recurring pattern in cyber-insurance ransomware claims, well-documented in legal and insurance trade press. The carrier denial rate on ransomware claims, and the named-perils-versus-loss-type gap, are a real public reference for this failure mode. The scenario is generic; no specific carrier, insured, or denial letter is referenced.

Maturity: **REVIEW-READY**. The doctrine and the safeguard inventory are defined. AUDIT-READY status would require an operating insured organization that produces the coverage-readiness packet as a precondition of every policy and refreshes it at every renewal.
