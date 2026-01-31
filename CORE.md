# ProofBundles Core

## What this is

A **cryptographically closed artifact** that binds:

- **Input state** (data, model configuration, prompt)
- **Execution record** (decisions, reasoning steps, intermediate outputs)
- **Output state** (final response)
- **Verification state** (manifest of all tracked files and their hashes)

into a single reproducible, falsifiable unit.

---

## Three primitives

### 1. Artifact closure

A ProofBundle is **closed** when all files referenced in the manifest are present and hash-verified.

- If any file is missing, altered, or added without updating the manifest, verification fails.
- Closure is binary: VALID or INVALID. There is no partial closure.

### 2. Verification inclusion

The manifest itself is **part of the bundle**, not external to it.

- The verifier recomputes hashes of all tracked files.
- Recomputed hashes are compared against the manifest.
- If they match exactly, the bundle is VALID. Any deviation produces INVALID.

### 3. Binding-point-forward

ProofBundles bind **from the moment of creation forward**.

- They do not claim ground-truth capture of external reality (e.g., "was this image real?").
- They claim: "From this point onward, these artifacts, this execution, and this output are bound together and have not been altered."
- Any falsification after binding is detectable. Any falsification before binding is out of scope.

---

## Failure semantics

**Verification outcomes:**

- **VALID** → All files match the manifest exactly. The bundle is intact.
- **INVALID** → Any mismatch, omission, or alteration. The bundle is compromised or incomplete.

**No trust required:**

- The verifier does not trust the bundle creator.
- The verifier does not trust the execution environment.
- The verifier trusts only: cryptographic hashes (SHA-256) and the mathematical properties of hash comparison.

**If verification fails, all claims void:**

- A failed ProofBundle makes no promises.
- Unverifiable claims are non-claims.

---

## What this is not

- **Not a claim of truth.** A valid bundle means "these artifacts are unaltered since binding," not "these artifacts are factually correct."
- **Not a timestamp proof.** ProofBundles do not currently include cryptographic timestamps (though they may in the future).
- **Not a supply-chain guarantee.** ProofBundles do not attest to the integrity of compilers, operating systems, or hardware.
- **Not a zero-knowledge proof.** The entire bundle must be disclosed to verify.

---

## Verification process

```
python verify.py
```

The verifier:

1. Reads `manifest.sha256`.
2. Computes SHA-256 hashes for all files listed in the manifest.
3. Compares recomputed hashes with manifest hashes.
4. Returns **VALID** if all match; **INVALID** otherwise.

---

## Why this matters

Existing provenance systems:

- **C2PA** (Content Provenance and Authenticity Coalition) → Metadata signing, easily stripped or forged in non-participating tools.
- **Codenotary / VeritasChain** → Supply-chain and event-log attestation, not decision-bearing artifacts.
- **NIST AI 100-4** → Conceptual guidance, no cryptographic binding of AI decisions.

ProofBundles introduces **artifact-closed, decision-bearing provenance**:

- The bundle is the unit of verification.
- Decisions, reasoning, and outputs are bound artifacts, not logs.
- Falsification is detectable by anyone with the bundle and the verifier.

---

## Security model

**Threat model:**

- An attacker with access to the bundle may attempt to alter files, inject new files, or remove files.
- An attacker may attempt to recompute the manifest to match altered files.

**Defenses:**

- Any alteration breaks the hash chain.
- Recomputing the manifest to match altered files produces a different manifest, which can be detected if the original manifest is archived or timestamped externally.
- Future versions may include Merkle-tree or blockchain anchoring for tamper-evident history.

**What we do not defend against (yet):**

- Timestamp forgery (no cryptographic timestamp authority).
- Insider attacks where the bundle creator is malicious from the start (garbage in, garbage bound).
- Physical or side-channel attacks on the verification environment.

---

## Design principles

1. **Simplicity over features.** The verifier is <100 lines of Python. No dependencies beyond `hashlib`.
2. **Falsifiability over claims.** A ProofBundle is designed to fail verification if compromised, not to make unfalsifiable assertions.
3. **Transparency over trust.** Everything is disclosed. The verifier trusts only math.

---

## Roadmap

- **Attack Bundle 001:** First adversarial test case (manifest alteration).
- **Merkle-tree extension:** Hierarchical hash structure for large bundles.
- **Timestamp anchoring:** Integration with blockchain or RFC 3161 timestamp authorities.
- **Replay hooks:** Standardized interfaces for re-executing AI decisions from closed bundles.

---

## License

Apache 2.0. See `LICENSE` for details.
