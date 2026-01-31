# ProofBundles

This repository defines a minimal, verification-first substrate for bundling artifacts such that integrity and inclusion are mechanically verifiable without trust in authors, platforms, or narratives.

## What this is
A ProofBundle is a finite set of artifacts plus a manifest that allows any verifier to detect modification, omission, or substitution.

## What this is not
- Not a blockchain
- Not a governance system
- Not an ethical or safety framework
- Not a claim of truth, correctness, or completeness

## Invariant
Any modification, omission, reordering, or substitution of tracked artifacts is mechanically detectable.

## Verification
Run:

```bash
python tools/verify.py
```

The verifier recomputes SHA-256 hashes for all tracked files and compares them against `manifest.sha256`.

## Falsification
- Any hash mismatch falsifies integrity.
- Any untracked artifact is excluded.
- Integrity does not imply correctness, meaning, safety, or intent.

## Scope limit
This repository specifies integrity and provenance mechanics only. All semantic or normative interpretation is out of scope.
