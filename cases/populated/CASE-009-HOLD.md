# CASE-009 — HOLD

## 1. Case Title

**Crypto Theft — Legal Void From Unbound Ownership**

A wallet is drained; on-chain movement is visible to anyone; the claimant asserts loss. The court, the insurer, or the recovery process cannot proceed because the binding between the wallet, the claimant's identity, and the pre-loss control state was never sealed. Raw on-chain data is technical record, not legal evidence.

---

## 2. Surface Signal

```
wallet_address: 0x… (publicly visible)
state_at_T-0: balance present
state_at_T+1: balance drained
on_chain_movement_visible: TRUE
destination_address_visible: TRUE
claimant_assertion: "this wallet was mine; I did not authorize the movement; this is theft"
counterparty_assertion: none (no opposing party in many crypto-theft cases)
legal_forum_question: "prove this wallet was yours, and prove this movement was unauthorized"
```

The surface is rich with on-chain data and poor in legal evidence. Anyone can verify the movement. No one can verify, from on-chain data alone, **whose wallet it was** or **whether the movement was authorized by the owner.** **Surface status is not Root truth.** A blockchain entry is a positive event record; it is not a custody seal binding the event to a legal claimant.

---

## 3. Root Failure

The required Root for a wallet-loss PASS is:

```
ROOT(wallet_loss_claim) =
    wallet_loss_trace
  ∧ ownership_binding_to_claimant
  ∧ pre_loss_control_verification
  ∧ theft_causation_path
  ∧ sealed_artifact_for_legal_forum
```

In the standard crypto-theft scenario:

- `wallet_loss_trace` = **TRUE** — on-chain movement is publicly visible
- `ownership_binding_to_claimant` = **MISSING** — no sealed binding between the wallet's signing key and the claimant's legal identity exists prior to the loss
- `pre_loss_control_verification` = **MISSING** — no sealed observation that the claimant in fact controlled the wallet up to T-0
- `theft_causation_path` = **MISSING** — the movement is visible, but the causation (who initiated it, under what compromise, against whose authority) is not sealed
- `sealed_artifact_for_legal_forum` = **MISSING** — there is no admissible artifact that a court or insurer can ingest; only public chain data

Four required roots are **MISSING**. This is Missing Root, not False Root. The on-chain data is not contradicted; it is simply insufficient. The doctrine `MISSING ROOT → HOLD` applies.

---

## 4. Missing Safeguard

The safeguards that should have existed and did not:

- **`verify_wallet_owner_binding`** — at wallet creation or first use, a custody-sealed binding between the wallet's signing key and the claimant's verified legal identity (or pseudonymous identity bound to a sealed legal-identity escrow) should be produced.
- **`verify_pre_loss_control`** — periodic sealed observations (signed challenges, time-bounded proof-of-control transactions) should attest to continuous control up to the moment of loss.
- **`verify_post_loss_transfer_path`** — the destination address(es) and any subsequent hops should be sealed against a custody timeline, separating "movement the owner could have authorized" from "movement that occurred after credible loss-of-control."
- **`seal_loss_artifact`** — at the moment of credible loss (compromised key, phished signature, exchange breach), an artifact should be produced and sealed: who reported it, when, with what corroborating evidence.
- **`bind_wallet_event_to_claimant_identity`** — the entire chain (binding → pre-loss control → loss event → post-loss path) must be bundled into a single admissible artifact addressable by the legal forum.

These safeguards are not exotic; they exist in fragments (KYC at exchanges, time-stamped signed messages, custody attestations). What is absent is the **bundling and sealing into a single admissible artifact**.

---

## 5. VYRDON Root Map

```
ROOT      = MISSING (ownership binding and pre-loss control sealing are absent)
GATE      = N/A    (no gate evaluation can produce PASS without ROOT being TRUE)
VALID     = MISSING (the legal-forum fields have no sealed values)
CERTIFIED = N/A    (nothing can be certified without VALID being TRUE)

Contradiction = FALSE  (on-chain movement is uncontested; the gap is sealing, not contradiction)
```

### Math

```
RecoveryProof
  = WalletLossTrace
  × OwnershipBinding
  × PreLossControlVerification
  × TheftCausation
  × AdmissibleArtifact

If OwnershipBinding = 0 or PreLossControlVerification = 0 or TheftCausation = 0
or AdmissibleArtifact = 0, then RecoveryProof = 0.
```

### Code

```python
# What systems do today (insufficient)
def wallet_loss_pass(
    wallet_drained: bool,
    onchain_movement_visible: bool,
) -> bool:
    return wallet_drained and onchain_movement_visible
```

```python
# Missing safeguard (not attack code)
def verify_wallet_owner_binding():
    pass

def verify_pre_loss_control():
    pass

def verify_post_loss_transfer_path():
    pass

def seal_loss_artifact():
    pass

def bind_wallet_event_to_claimant_identity():
    pass
```

```python
# Right defensive control
def wallet_loss_pass(
    wallet_loss_traced: bool,
    owner_identity_bound: bool,
    pre_loss_control_verified: bool,
    theft_path_verified: bool,
    sealed_artifact_present: bool,
) -> bool:
    return (
        wallet_loss_traced
        and owner_identity_bound
        and pre_loss_control_verified
        and theft_path_verified
        and sealed_artifact_present
    )
```

Synthesis factor mapping:

| FinalExecutionPass factor | State in this case |
|---|---|
| MachineGreen | 1 — chain movement is visible |
| HumanRedSeal | 0 — no sealed identity binding from the human owner |
| ProofComplete | 0 — pre-loss control and theft causation are unsealed |
| ArchiveFinal | 0 — no admissible artifact has been bundled and sealed |
| CustodySeal | 0 — the on-chain record carries no custody binding to the legal claimant |

Four zeros → product is zero → not PASS. All four are MISSING, not contradicted → **HOLD** is the verdict per the missing-root rule.

---

## 6. Decision

```
DECISION = HOLD
DECISION_CODE = DEC-HOLD-MISSING_ROOT
```

**Reasoning.** Four required roots are MISSING, not FALSE. The on-chain movement is real; the binding between that movement and a legal claimant is absent. Per doctrine, MISSING ROOT → HOLD. The verdict is not NO_PASS because the on-chain data is not contradicted — it is only insufficient to support the claim alone. NO_PASS would be the verdict if, for example, a sealed pre-loss observation showed the claimant did not in fact control the wallet at T-0.

---

## 7. Output Packet

**Wallet Loss Evidence Packet** — emitted in HOLD state, carries:

- The wallet address and chain movement record
- The list of sealing artifacts that are MISSING (owner binding, pre-loss control, theft causation, admissible bundle)
- Any partial artifacts that do exist (exchange KYC excerpt, last signed proof-of-control if any, breach disclosure if any)
- A recommendation to the legal forum that the on-chain record alone is insufficient and that production of the missing seals is required for the case to advance to NO_PASS (denied claim) or PASS (claim accepted with recovery instruction)

The packet's role is not to recover funds. It is to make the gap auditable so that a court, an insurer, or an exchange's recovery process can see exactly which seals must be produced before any direction is taken.

---

## 8. System Boundary

VYRDON does **not** claim:

- that funds can be recovered — recovery is a legal/operational question downstream of HOLD
- that the claimant is the wallet's owner — only that the binding has not been sealed
- that retroactive seals can be produced — pre-loss control attestations must be made before the loss to be admissible
- that exchanges or chain-analysis firms can be substituted for custody-sealed evidence

VYRDON **does** claim:

- that on-chain visibility alone is technical record, not legal evidence
- that the binding from wallet → identity → pre-loss control → loss event → post-loss path must be sealed as one artifact, not five disjoint records
- that without the seal, a HOLD is more honest than a NO_PASS or a PASS

---

## 9. Audit Note

Evidence that would strengthen this review:

- An exchange or custody provider that issues signed, time-stamped ownership bindings at account creation
- Time-bounded proof-of-control transactions (signed messages with timestamps) issued by the owner at regular intervals
- A breach-disclosure artifact (from an exchange, a custodian, or the owner themselves) with a sealed timestamp
- A chain-analysis report sealed by an independent firm, ingestible by the legal forum

Reconstructed-from-public-reporting elements: this case is the doctrine form of a recurring pattern in crypto-theft litigation and insurance: chain data exists, ownership cannot be proved to a legal standard. UAE court rulings on the insufficiency of raw on-chain data as legal proof are a real public reference for this gap. The scenario is generic; no specific real wallet, court, or party is referenced.

Maturity: **REVIEW-READY**. The doctrine and the safeguard inventory are defined. AUDIT-READY status would require operating custodians or wallets that emit the sealing artifacts at creation and at periodic intervals, plus an independent observer's custody chain.
