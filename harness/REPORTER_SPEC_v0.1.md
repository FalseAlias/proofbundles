## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit ce53e20d101be61988aa1757d2acb7d24251d967)
As-of timestamp: 2026-02-01
Known exclusions: presentation formats, external sinks, policy interpretation
Falsification conditions: identical inputs and state yield non-identical report bytes or mismatched status enums

## Reporter Contract
report(verdict: HarnessVerdict, artifacts: bytes, state_id: bytes32, seq: uint64) -> (report_id: bytes32, status: ReportStatus)

HarnessVerdict enum:
- Pass
- Fail
- Inconclusive

ReportStatus enum:
- Emitted
- Duplicate
- Rejected

verify_report(report_id: bytes32) -> (status: VerifyStatus)

VerifyStatus enum:
- Present
- Absent

## Normalization
artifacts_norm := SHA256(artifacts)
report_id := SHA256(verdict + artifacts_norm + state_id + seq)

## Rules
- Emitted: first emission of (verdict, artifacts_norm, state_id, seq)
- Duplicate: repeated emission with identical tuple
- Rejected: seq < last_seq(state_id)

## State
store key := state_id
store value := {last_seq: uint64, emitted_norms: set[bytes32]}

## Determinism
Given identical ordered inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Reporter executes after harness verdict and before any export or persistence.

## Offline
If state unavailable, status := Rejected

## Tests
- Same inputs/state/seq → Emitted then Duplicate
- Decreasing seq → Rejected
- report_id present → Present
- Absent → Absent