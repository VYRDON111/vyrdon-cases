# Operations Room Case: Integrity Violation During Backup

## Room: Operations
## Domain: System health — uptime, latency, queues, backups, integrity

---

## Scenario

Scheduled backup completes successfully. But the post-backup integrity check detects that a registry snapshot hash does not match the expected value. The backup contains a registry that differs from the live registry.

## Pipeline Processing

### Stage 1: Ingest
- Backup completion event ingested
- Post-backup integrity check results ingested

### Stage 2: Compute
- Uptime metric: normal
- Latency metric: normal
- Queue metric: clear
- Backup metric: COMPLETED (but flagged)
- Integrity metric: FAIL (registry hash mismatch)

### Stage 3: Explain
- "Backup completed at timestamp T. Post-backup integrity check shows registry snapshot hash differs from live registry hash. Delta: 3 entries added to live registry after backup snapshot was taken but before integrity check ran. This is a timing gap, not corruption."

### Stage 4: Transition
- State transition: `BACKUP_COMPLETE` → `INTEGRITY_WARNING`

### Stage 5: Reconcile
- Cross-room check with Evidence room: evidence of the 3 new registry entries exists with timestamps after the backup snapshot time
- Reconciliation: timing gap confirmed, not corruption. But integrity warning remains until re-verified.

### Stage 8: Enforce
- Integrity metric FAIL triggers deployment law: no deployment from this backup until re-verified
- HOLD on backup promotion

## Decision

**HOLD** — Integrity warning requires re-verification before backup can be trusted for recovery.

---

## Metrics Computed

| Metric | Value | Status |
|--------|-------|--------|
| Uptime | Normal | ✓ |
| Latency | Normal | ✓ |
| Queue | Clear | ✓ |
| Backup | Completed | ⚠ |
| Integrity | Hash mismatch | ✗ |
