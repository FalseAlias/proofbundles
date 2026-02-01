## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit e93d712864841b9c467e981de7207cff1c483824)
As-of timestamp: 2026-02-01
Known exclusions: authorization, policy, cryptography beyond hash presence
Falsification conditions: identical inputs yield non-identical outputs or mismatched failure enums

## Identity Contract
identify(subject: bytes, context: bytes32) -> (id: bytes32, status: IdentityStatus)

IdentityStatus enum:
- Resolved
- Ambiguous
- Unknown

## Normalization
subject_norm := SHA256(subject)

## Rules
- Resolved: subject_norm maps to a single id within context
- Ambiguous: subject_norm maps to multiple ids within context
- Unknown: subject_norm has no mapping within context

## State
store key := (subject_norm, context)
store value := {id: bytes32, resolution_ts: uint64}

## Determinism
Given identical inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Identity resolution occurs before revocation/replay checks.

## Offline
If state unavailable, status := Unknown.

## Tests
- Single mapping → Resolved
- Multiple mappings → Ambiguous
- No mapping → Unknown