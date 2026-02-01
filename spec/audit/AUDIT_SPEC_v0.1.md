## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit e93d712864841b9c467e981de7207cff1c483824)
As-of timestamp: 2026-02-01
Known exclusions: policy, authorization, cryptography beyond hash presence
Falsification conditions: identical inputs and state yield non-identical audit outputs or mismatched failure enums

## Audit Contract
audit(event: bytes, context: bytes32, seq: uint64) -> (record_id: bytes32, status: AuditStatus)

AuditStatus enum:
- Emitted
- Duplicate
- Rejected

verify_audit(record_id: bytes32) -> (status: VerifyStatus)

VerifyStatus enum:
- Confirmed
- Missing
- Tampered

## Normalization
record_id := SHA256(event + context + seq)

## Rules
- Emitted: first (record_id) in seq order
- Duplicate: repeat record_id
- Rejected: seq decreases or gap > threshold

## State
store key := record_id
store value := {seq: uint64, emit_ts: uint64}

## Determinism
Given identical inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Audit emission wraps all prior layers (identity→replay→revocation→resolution).

## Offline
If state unavailable, verify_audit returns Missing.

## Tests
- First event → Emitted
- Repeat → Duplicate
- Seq decrease → Rejected
- record_id present → Confirmed
- Absent → Missing
- Mismatch → Tampered