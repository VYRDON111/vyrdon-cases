# Campaign Room Case: Conversion Tracking Gap

## Room: Campaign
## Domain: Outreach — reach, response, conversion, followup

---

## Scenario

A campaign reports high reach and high response rates. But conversion tracking shows zero conversions despite 200+ responses. The followup metric shows no followup actions were recorded.

## Pipeline Processing

### Stage 1: Ingest
- Campaign performance data ingested: 10,000 reach, 200 responses, 0 conversions

### Stage 2: Compute
- Reach metric: 10,000 (normal)
- Response metric: 200 (2% response rate — normal)
- Conversion metric: 0 (0% conversion — gap detected)
- Followup metric: 0 followup actions recorded (gap detected)

### Stage 3: Explain
- "Campaign reached 10,000 targets with 200 responses, but no conversions and no followup actions recorded. Either the conversion tracking is broken (technical failure) or responses were not followed up (process failure). Either way, the campaign cannot be evaluated as successful without conversion evidence."

### Stage 4: Transition
- State transition: `ACTIVE` → `TRACKING_GAP`

### Stage 5: Reconcile
- Cross-room check with Commercial room: no transactions traced to this campaign
- Reconciliation confirms: zero conversions is consistent across rooms

### Stage 8: Enforce
- VALID: FALSE (conversion evidence missing)
- CERTIFIED: FALSE (followup chain not recorded)
- Result: **PASS.FALSE**

## Decision

**NO_PASS** — Campaign cannot pass without conversion evidence and followup records.

---

## Metrics Computed

| Metric | Value | Status |
|--------|-------|--------|
| Reach | 10,000 | ✓ |
| Response | 200 | ✓ |
| Conversion | 0 | ✗ |
| Followup | 0 | ✗ |
