## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit e93d712864841b9c467e981de7207cff1c483824)
As-of timestamp: 2026-02-01
Known exclusions: policy, authorization, cryptography beyond hash presence
Falsification conditions: identical inputs and state yield non-identical outputs or mismatched failure enums

## Downgrade Contract
downgrade(request: bytes, from_version: uint32, to_version: uint32) -> (decision: DowngradeDecision, reason: DowngradeReason)

DowngradeDecision enum:
- Allowed
- Denied
- Deferred

DowngradeReason enum:
- PolicyBlock
- SealBlock
- StateMissing

## Normalization
req_norm := SHA256(request)

## Rules
- Allowed: to_version >= seal.version AND downgrade_map permits (from_version, to_version)
- Denied: to_version < seal.version OR downgrade_map forbids pair
- Deferred: required state unavailable

## State
store key := (from_version, to_version)
store value := permission_flag

## Determinism
Given identical inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Downgrade checks occur after revocation and before resolution finalization.

## Offline
If state unavailable, decision := Deferred

## Tests
- Same inputs/state → same decision
- to_version < seal.version → Denied
- permitted pair → Allowed
- missing state → Deferred