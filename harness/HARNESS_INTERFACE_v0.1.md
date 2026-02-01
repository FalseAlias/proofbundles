## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit e93d712864841b9c467e981de7207cff1c483824)
As-of timestamp: 2026-02-01
Known exclusions: runtime implementations, CI configuration, cryptography beyond hash presence
Falsification conditions: identical inputs and state yield non-identical harness verdicts or mismatched failure enums

## Harness Contract
verify(test_id: bytes32, inputs: bytes, state: bytes) -> (verdict: HarnessVerdict, detail: bytes32)

HarnessVerdict enum:
- Pass
- Fail
- Inconclusive

snapshot(state: bytes) -> (state_id: bytes32)

restore(state_id: bytes32) -> (status: RestoreStatus)

RestoreStatus enum:
- Restored
- Missing

## Normalization
inputs_norm := SHA256(inputs)
state_norm := SHA256(state)

## Rules
- Pass: all referenced specs return expected outputs for inputs/state
- Fail: any referenced spec returns unexpected output or failure enum
- Inconclusive: required state unavailable or exclusion encountered

## State
store key := state_id
store value := state_norm

## Determinism
Given identical inputs, state snapshot, and spec versions, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Harness execution occurs after spec selection and before report emission.

## Offline
If state snapshot unavailable, verdict := Inconclusive

## Tests
- Same inputs/state/specs → same verdict
- Modified state snapshot → Fail or Inconclusive
- Missing state_id → Inconclusive