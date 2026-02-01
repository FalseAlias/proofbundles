## Verification Envelope
Inspected sources: github.com/FalseAlias/proofbundles@main (commit 4ef27183081c89063d475b16ffad43a9cd188518)
As-of timestamp: 2026-02-01
Known exclusions: identity, authorization, cryptography beyond hash presence
Falsification conditions: identical input stream yields non-identical outputs or mismatched failure enums

## Replay Contract
replay(input: bytes, nonce: bytes32, window: uint64) -> (hash: bytes32, status: ReplayStatus)

ReplayStatus enum:
- Accepted
- Replayed
- Expired
- OutOfOrder

## Normalization
norm := SHA256(input)

## Rules
- Accepted: first occurrence of (norm, nonce) within window
- Replayed: repeated (norm, nonce) within window
- Expired: timestamp < now - window
- OutOfOrder: sequence index decreases relative to last accepted index

## State
store key := (norm, nonce)
store value := {first_seen_ts: uint64, last_index: uint64}

## Determinism
Given identical ordered inputs and state snapshot, outputs are byte-for-byte identical or the same typed failure.

## Cache Interaction
Replay checks occur before resolution cache write.

## Offline
If state unavailable, status := OutOfOrder

## Tests
- Same stream → same outputs
- Permuted order → OutOfOrder where applicable
- Duplicate nonce → Replayed
- Window exceeded → Expired