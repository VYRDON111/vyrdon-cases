# CASE-007 — HOLD

## 1. Case Title

**Nothing Happened — Legal Failure to Prove a Negative**

The "no execution occurred" / "no route was accessed" / "no trade was placed" claim, where the claimant must demonstrate that an event did NOT happen and the standard system architecture has no archive of non-events.

---

## 2. Surface Signal

```
claim_type: NEGATIVE_EXECUTION_CLAIM
claim_status: ASSERTED_BY_CLAIMANT
claim_source: counterparty_dispute
claimed_action: "no trade was placed on date X by user Y"
opposing_party_assertion: "trade was placed and is binding"
visible_system_state: no record of trade in user-facing dashboard
visible_system_state: no record of trade in opposing-party dashboard either
```

The surface appears to support the claimant — neither dashboard shows the disputed trade. The opposing party's assertion is unsupported by visible state, and the claimant's assertion is unsupported by visible state. Both parties are pointing at the same absence and drawing opposite conclusions.

**Surface status is not Root truth.** The absence of a positive record is not the same as a sealed non-execution record.

---

## 3. Root Failure

The required Root for a negative-claim PASS is:

```
ROOT(negative_execution_claim) =
    request_trace_for_disputed_window
  ∧ route_attempt_log_for_disputed_window
  ∧ no_execution_seal_for_disputed_window
  ∧ no_trade_seal_for_disputed_window
  ∧ archive_finality_over_disputed_window
```

In standard transaction-recording systems, every one of these is **MISSING**:

- Request traces only exist for requests that produced a positive event.
- Route attempts are not separately recorded; only successful routes are logged.
- "No execution" is never sealed as an event; it is inferred from the absence of an execution record.
- "No trade" is never sealed as an event; it is inferred from the absence of a trade record.
- Archive finality covers positive events only; the absence of an event is not archived because there is no event to archive.

This is **Missing Root**, not False Root. The standard system does not contradict the claimant's assertion. It is simply silent — and silence is not proof.

---

## 4. Missing Safeguard

The safeguards that should have existed and did not:

- **`record_no_route_state`** — at every authentication or session boundary in the disputed window, the system should have sealed a "no route to trading API was requested by this principal" state if no route was in fact requested.
- **`record_no_access_state`** — at every read-only access in the disputed window, the system should have sealed a "no order-entry surface was accessed" state.
- **`record_no_execution_state`** — at the close of every execution-eligible window in which no execution occurred, the system should have sealed that fact.
- **`record_no_trade_state`** — at the close of every trade-eligible window in which no trade occurred, the system should have sealed that fact.
- **`seal_negative_outcome`** — the negative state seals must be bound to a custody seal (which principal observed them) and written under archive finality.

In conventional systems, every one of these is absent. The system is structurally **positive-only**: it records what happened, never what did not happen. Under Root Language, this is a class of failure that resolves to **HOLD** as long as the architecture remains positive-only; it cannot resolve to PASS, and it cannot honestly resolve to NO_PASS, because no contradicting evidence exists.

---

## 5. VYRDON Root Map

```
ROOT      = MISSING (no negative-state seals exist for the disputed window)
GATE      = N/A    (no gate evaluation can be made without a Root)
VALID     = MISSING (the negative-claim fields have no sealed values)
CERTIFIED = N/A    (nothing can be certified without VALID being TRUE)

Contradiction = FALSE  (no contradiction; the system is silent)
```

### Math

```
NegativeClaimProof
  = RequestTrace
  × RouteTrace
  × NoExecutionSeal
  × NoTradeSeal
  × FinalArchive

If NoExecutionSeal = 0 or NoTradeSeal = 0, then NegativeClaimProof = 0.
```

### Code

```python
# What systems do today (insufficient)
def record_trade_event(trade_executed: bool):
    if trade_executed:
        return "TRADE_RECORDED"
    return None
```

```python
# Missing safeguard (not attack code)
def record_no_route_state():
    pass

def record_no_access_state():
    pass

def record_no_execution_state():
    pass

def record_no_trade_state():
    pass

def seal_negative_outcome():
    pass
```

```python
# Right defensive control
def negative_claim_pass(
    request_logged: bool,
    route_checked: bool,
    no_execution_sealed: bool,
    no_trade_sealed: bool,
    archive_written: bool,
) -> bool:
    return (
        request_logged
        and route_checked
        and no_execution_sealed
        and no_trade_sealed
        and archive_written
    )
```

Synthesis factor mapping:

| FinalExecutionPass factor | State in this case |
|---|---|
| MachineGreen | N/A — no machine state was produced for the disputed window |
| HumanRedSeal | N/A — no authorization was needed because no action was taken |
| ProofComplete | 0 — required negative-state seals are missing |
| ArchiveFinal | 0 — archive is positive-only |
| CustodySeal | 0 — no custodial binding exists for an event that was never recorded |

Three zeros → product is zero → not PASS.

---

## 6. Decision

```
DECISION = HOLD
DECISION_CODE = DEC-HOLD-MISSING_ROOT
```

**Reasoning.** Three required roots (no_execution_seal, no_trade_seal, archive_final) are **MISSING**, not FALSE. The doctrine `MISSING ROOT → HOLD` applies. The verdict is not NO_PASS because there is no contradicting evidence; it is HOLD because the system cannot produce the seals the claim requires. NO_PASS would be reached only if a positive execution record were found (in which case the negative claim is contradicted and authority falls to the positive record).

---

## 7. Output Packet

**Negative Claim Proof Packet** — emitted in HOLD state, carries:

- The disputed window (UTC start, UTC end)
- The principal whose non-action is claimed
- The list of negative-state seals that are MISSING
- The list of positive events found in the window (if any) — used to either confirm HOLD or escalate to NO_PASS
- An audit note describing why the system is structurally unable to issue PASS

The packet's role is not to prove the negative claim; it is to make the absence auditable so that downstream review (legal, regulatory, internal) can decide on grounds other than the system's silence.

---

## 8. System Boundary

VYRDON does **not** claim:

- that the disputed event did not occur — it only claims that the required seals are missing
- that the claimant is correct — only that the opposing-party assertion is also unsupported
- that the archive can retroactively produce negative-state seals — they must be produced at the time the eligible window was open
- that this case is recoverable without changing the host system to a positive-and-negative recording architecture

VYRDON **does** claim:

- that the required Root for a negative-claim PASS is well-defined
- that the absence of that Root is sealable as HOLD with a Negative Claim Proof Packet
- that this verdict is reproducible: anyone with the same window and the same principal would receive the same HOLD

---

## 9. Audit Note

Evidence that would strengthen this review:

- A host system that, going forward, emits `no_route_state` / `no_access_state` / `no_execution_state` / `no_trade_state` seals at the boundary of each eligible window
- A custody-sealed observer (independent of both parties) that attests to the absence over the disputed window
- An archive-finality contract that includes negative-state seals in the same append-only journal as positive events

Reconstructed-from-public-reporting elements: this case is the doctrine form of a class of disputes well-documented in legal and trading literature (the "prove the negative" gap). The scenario is generic; no specific real dispute is referenced.

Maturity: **REVIEW-READY**. The doctrine and the safeguard inventory are defined. AUDIT-READY status would require an operating host that emits negative-state seals and an independent observer's custody chain.
