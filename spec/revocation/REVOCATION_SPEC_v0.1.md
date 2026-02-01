## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit e93d712864841b9c467e981de7207cff1c483824)
As-of timestamp: 2026-02-01
Known exclusions: identity, authorization, cryptography beyond hash presence
Falsification conditions: revoked input resolves as non-revoked under identical inputs and state

## Revocation Contract
revoke(target: bytes32, reason: bytes32, effective_ts: uint64) -> (status: RevocationStatus)

RevocationStatus enum:
- Recorded
- Duplicate
- Invalid

check_revocation(target: bytes32, at_ts: uint64) -> (status: CheckStatus)

CheckStatus enum:
- Active
- Revoked
- Unknown

## Normalization
target_id := SHA256(target)

## Rules
- Recorded: first record of target_id with effective_ts
- Duplicate: same target_id with identical effective_ts
- Invalid: effective_ts > now + skew_limit

## State
store key := target_id
store value := {effective_ts: uint64, reason: bytes32}

## Determinism
Given identical inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Ordering
Revocation checks occur after replay checks and before resolution output finalization.

## Offline
If state unavailable, check_revocation returns Unknown.

## Tests
- Same target_id/effective_ts → Recorded then Duplicate
- at_ts < effective_ts → Active
- at_ts >= effective_ts → Revoked
- future effective_ts beyond skew_limit → Invalid