# Market Room Case: Signal Contradiction

## Room: Market
## Domain: Market analysis — targets, momentum, volatility, conviction

---

## Scenario

A market thesis claims bullish momentum. The momentum metric confirms upward trend. But the volatility metric shows extreme instability, and the conviction metric shows declining confidence across analysts. The signal contradicts the thesis.

## Pipeline Processing

### Stage 1: Ingest
- Market data ingested: price history, volume, analyst ratings, sector comparatives

### Stage 2: Compute
- Target metric: thesis target still within range
- Momentum metric: positive (upward trend confirmed)
- Volatility metric: EXTREME (3x normal range)
- Conviction metric: DECLINING (analyst confidence dropping)

### Stage 3: Explain
- "Momentum is positive but volatility is extreme and conviction is declining. The bullish thesis is contradicted by instability indicators. Momentum alone does not confirm the thesis when volatility exceeds normal bounds and conviction is falling."

### Stage 4: Transition
- State transition: `ACTIVE_THESIS` → `SIGNAL_CONTRADICTION`

### Stage 5: Reconcile
- Cross-room check: No dependency from other rooms on this market thesis
- Self-reconciliation: momentum vs. volatility vs. conviction = contradiction

### Stage 8: Enforce
- VALID: FALSE (signal contradiction detected)
- Result: **PASS.FALSE** — thesis cannot pass with contradicting signals

## Decision

**NO_PASS** — Market thesis contradicted by volatility and conviction metrics.

---

## Metrics Computed

| Metric | Value | Status |
|--------|-------|--------|
| Target | In range | ✓ |
| Momentum | Positive | ✓ |
| Volatility | Extreme | ✗ |
| Conviction | Declining | ✗ |
