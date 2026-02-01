## Verification Envelope
Inspected sources: proofbundles main (`a65d4285f451529c6778fa9dc7f4d1acad373822`)[cite:42]
As-of timestamp: 2026-01-31
Known exclusions: Identity, crypto details, policy
Falsification conditions: Non-identical output on canonical input set, SHA mismatch on seal

## Resolution Contract

Signature: `resolve(canonical_input: bytes) -> (bundle_hash: bytes32, status: ResolutionStatus)`

ResolutionStatus enum:
- Valid
- PoisonedInput
- DowngradeAttempt
- CacheMiss
- OfflineFail

### Input Normalization
1. UTF-8 decode(input)
2. Prepend monotonic timestamp (ISO8601 UTC)
3. SHA256(norm) = canonical_key

### Cache Semantics
- Key: canonical_key
- TTL: 300s
- Invalidate: seal update or TTL expire

### Poison Resistance
- Reject if timestamp deviates >10% from monotonic chain
- Status: PoisonedInput

### Downgrade Handling
- Reject if input.version < current_seal.version
- Status: DowngradeAttempt

### Offline Guarantees
- Fallback: local seal cache
- Fail if absent: OfflineFail

### Acceptance Test
Same canonical input set → identical bytes or typed status.
Order/cache/partial loss agnostic.

References: CANONICAL_INDEX.md seal points.