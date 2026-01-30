This repository contains a verification-first provenance substrate.

This is not a claim of truth, correctness, safety, morality, or completeness.
This repository makes no promises and asks for no trust.

All contents are provisional constructs subject to falsification.

Verification:
Run `python verify.py` from the repository root.
The verifier recomputes SHA-256 hashes for all tracked files and compares them
against `manifest.sha256`.

Outcome:
VALID   — all files match the manifest exactly.
INVALID — any mismatch, omission, or alteration.

Falsification:
Any failure of verification voids all claims of integrity.
Unverifiable claims are non-claims.
