# CASE-008 — NO_PASS

## 1. Case Title

**Zeus-Class Banking Trojan — Authorized Token, Unauthorized Movement**

A retail-banking session in which a man-in-the-browser intercepts and re-uses a valid authentication token to initiate a second transfer to a beneficiary the human never authorized. The machine reports green; the gate releases; the loss is the bank's because the authority chain was broken without the gate noticing.

---

## 2. Surface Signal

```
session_id: session_active
auth_token: PRESENT and VALID (not yet expired, not yet revoked)
device_fingerprint: matches enrolled device
ip_geo: matches user's home country
two_factor_status: PASSED at login (T-0)
first_transfer:
    amount: legitimate
    beneficiary: pre-existing
    human_red_seal: PRESENT (T-0+30s)
    outcome: SETTLED
second_transfer:
    amount: legitimate-looking
    beneficiary: NEW (never used before by this principal)
    human_red_seal: ABSENT for THIS transfer
    machine_green: TRUE (token still valid, session still active)
    outcome: SETTLED  (release fired on machine_green alone)
```

The surface is fully green. The token is valid, the session is open, the device is trusted, and the geography matches. **Machine green is not execution authority.** The gate fired on a signal that was correct in form but wrong in substance: the human red seal for the first transfer was being treated as if it covered all transfers in the session.

---

## 3. Root Failure

The required Root for a high-risk transfer PASS is:

```
ROOT(high_risk_transfer) =
    token_possession
  ∧ human_presence_now
  ∧ session_closure_after_prior_transfer
  ∧ beneficiary_verification_for_this_transfer
  ∧ release_approval_human_red_seal_for_this_transfer
```

In the trojan scenario:

- `token_possession` = **TRUE** (the token is present — but possession by whom is the wrong question; the trojan has it too)
- `human_presence_now` = **FALSE** (the human authorized the first transfer; they did not authorize the second)
- `session_closure_after_prior_transfer` = **FALSE** (the session was never forced to close; the first authorization is being re-used)
- `beneficiary_verification_for_this_transfer` = **FALSE** (the new beneficiary was never independently verified by the human)
- `release_approval_human_red_seal_for_this_transfer` = **FALSE** (no fresh seal was produced for the second movement)

This is **False Root**, not Missing Root. The system has a positive answer for every field — but the answer for the authority pillar is fabricated. The trojan's continued possession of the token is being treated as the human's continued authorization. They are not the same thing.

---

## 4. Missing Safeguard

The safeguards that should have existed and did not:

- **`require_token_removal_after_completion`** — after every successful high-risk transfer, the token should be invalidated so that re-use produces a session restart rather than a free pass.
- **`force_session_termination`** — after every high-risk transfer, the session should terminate; a new transfer requires a new login from scratch.
- **`require_human_reconfirmation_for_next_transfer`** — within a single live session, the second transfer should require a fresh human red seal regardless of token validity.
- **`verify_new_beneficiary_before_release`** — a never-used beneficiary should not release on the same seal as the previous transfer; it should require an independent verification step (out-of-band channel, cooling period, or both).
- **`gate_must_distinguish_token_possession_from_authority`** — the gate must not accept "token still valid" as a proxy for "human still authorizing"; these are different roots.

The bank's gate collapsed two distinct roots — *token possession* and *human authority* — into one signal. That collapse is the False Root.

---

## 5. VYRDON Root Map

```
ROOT      = FALSE   (authority for the second transfer is fabricated; not the human's seal)
GATE      = FALSE   (the gate accepted machine green as authority)
VALID     = TRUE    (all fields are present and well-formed; the data is just wrong)
CERTIFIED = FALSE   (no honest decision record can be sealed under a fabricated authority)

Contradiction = TRUE  (machine_green=TRUE while human_red_seal=FALSE for THIS transfer)
```

### Math

```
TransferProtection
  = TokenPossession
  × HumanPresence
  × SessionClosure
  × BeneficiaryVerification
  × ReleaseApproval

If HumanPresence = 0 or SessionClosure = 0, then TransferProtection = 0.
```

### Code

```python
# What systems do today (insufficient)
def transfer_pass(
    token_present: bool,
    session_active: bool,
) -> bool:
    return token_present and session_active
```

```python
# Missing safeguard (not attack code)
def require_token_removal_after_completion():
    pass

def force_session_termination():
    pass

def require_human_reconfirmation_for_next_transfer():
    pass

def verify_new_beneficiary_before_release():
    pass

def gate_must_distinguish_token_possession_from_authority():
    pass
```

```python
# Right defensive control
def transfer_pass(
    token_present: bool,
    human_reconfirmed: bool,
    prior_session_closed: bool,
    beneficiary_verified: bool,
    final_release_signed: bool,
) -> bool:
    return (
        token_present
        and human_reconfirmed
        and prior_session_closed
        and beneficiary_verified
        and final_release_signed
    )
```

Synthesis factor mapping:

| FinalExecutionPass factor | State for the second transfer |
|---|---|
| MachineGreen | 1 — token valid, session active, device matches |
| HumanRedSeal | 0 — no fresh seal for the second transfer |
| ProofComplete | 0 — beneficiary verification is missing |
| ArchiveFinal | 1 — the transfer record was archived (but archived under a false authority) |
| CustodySeal | 0 — the authorization custody seal is for the first transfer, not the second |

Three zeros → product is zero → not PASS. Contradiction between MachineGreen and HumanRedSeal → **NO_PASS** is the verdict per the contradiction rule.

---

## 6. Decision

```
DECISION = NO_PASS
DECISION_CODE = DEC-NOPASS-FALSE_ROOT
```

**Reasoning.** This is False Root: the system reports authority is present, but the authority is not in fact present for this specific transfer. Contradiction between machine_green (TRUE) and human_red_seal (FALSE) blocks PASS absolutely. The verdict is NO_PASS, not HOLD, because there is contradicting evidence, not merely absent evidence.

---

## 7. Output Packet

**Transfer Protection Review Packet** — emitted in NO_PASS state, carries:

- The session ID and the two transfer events
- The custody seal for the first transfer's human red seal
- The absence of a custody seal for a second human red seal
- The new-beneficiary verification status (absent)
- The recommended enforcement: **RETURN** — funds must not move; if they have already moved, the bank's enforcement contract triggers a fraud return path, anomaly counter increments, and the session is force-terminated for the principal pending out-of-band reconfirmation.

The packet's role is to make the False Root explicit so that the bank's enforcement layer treats the second transfer as a separate authority event rather than a continuation of the first.

---

## 8. System Boundary

VYRDON does **not** claim:

- that all Zeus-class trojans can be detected — the detection question is upstream of the gate
- that token-possession compromise can be prevented — that is an endpoint security problem
- that this verdict prevents the first transfer (which had a valid human red seal) — only the second
- that this case is a substitute for endpoint malware monitoring, EDR, or browser hardening

VYRDON **does** claim:

- that the gate must treat each high-risk transfer as requiring its own fresh human red seal
- that machine green is a signal, not authority
- that a never-used beneficiary requires an independent verification step before release
- that session continuity is not authorization continuity

---

## 9. Audit Note

Evidence that would strengthen this review:

- A bank's transfer pipeline that produces, for every high-risk transfer, a per-transfer human red seal artifact (rather than a per-session seal)
- An out-of-band beneficiary verification channel that is independent of the session (push notification to a separately registered device, phone callback, in-branch confirmation)
- A force-close primitive in the session manager that fires after every high-risk transfer
- Anomaly counter telemetry that escalates after the first new-beneficiary release without out-of-band confirmation

Reconstructed-from-public-reporting elements: this case is the doctrine form of the Zeus / SpyEye / Carberp / Gozi family of banking trojans, well-documented in security research over the last fifteen years. The scenario is generic; no specific real bank, victim, or trojan instance is referenced.

Maturity: **REVIEW-READY**. The doctrine and the safeguard inventory are defined. AUDIT-READY status would require an operating bank's transfer pipeline that issues per-transfer human red seals and an independent observer's custody chain over the seals.
