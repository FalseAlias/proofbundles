# Attack Bundle 001: Manifest Alteration

## Attack classification

**Type:** Post-binding falsification  
**Target:** `manifest.sha256`  
**Goal:** Alter a tracked file without detection  
**Expected outcome:** Verification INVALID

---

## Attack scenario

An attacker with write access to a ProofBundle attempts to:

1. Modify the content of a tracked file (e.g., `README.md`).
2. Leave the manifest unchanged (hoping the verifier will not notice).
3. OR: Recompute the manifest to match the altered file.

Both variants should produce **INVALID** verification.

---

## Implementation

### Step 1: Create a valid baseline bundle

```bash
# Create a simple bundle with two files
mkdir attack_001
cd attack_001
echo "Original content" > data.txt
echo "Metadata" > metadata.txt

# Generate manifest
sha256sum data.txt metadata.txt > manifest.sha256

# Verify it's valid
python ../verify.py
# Expected: VALID
```

### Step 2: Attack variant A — File alteration without manifest update

```bash
# Alter a tracked file
echo "Tampered content" > data.txt

# Do NOT update manifest

# Run verifier
python ../verify.py
# Expected: INVALID
# Reason: Recomputed hash of data.txt does not match manifest entry
```

**Result:**  
The verifier recomputes the hash of `data.txt`, finds it differs from the manifest, and returns **INVALID**.

### Step 3: Attack variant B — File alteration with manifest recomputation

```bash
# Alter a tracked file
echo "Tampered content v2" > data.txt

# Recompute manifest to match altered file
sha256sum data.txt metadata.txt > manifest.sha256

# Run verifier
python ../verify.py
# Expected: VALID (but this is a NEW bundle, not the original)
```

**Result:**  
The verifier returns **VALID** because all current files match the current manifest.

**Critical observation:**  
This produces a *different* ProofBundle. If the original `manifest.sha256` was archived, timestamped, or shared externally, the attacker's recomputed manifest will not match the original.

**Defense:**  
- External archiving of `manifest.sha256` (e.g., blockchain anchor, timestamp authority, Git commit hash).
- Merkle-tree extension where the root hash is published separately.
- Verifier flag: `--check-against <external-manifest-hash>` to compare against a known-good baseline.

---

## Test cases

### Test case 1: Detect file alteration

```python
# In attack_001/ directory
import subprocess
import hashlib

# Create baseline
with open('data.txt', 'w') as f:
    f.write('Original content')
with open('metadata.txt', 'w') as f:
    f.write('Metadata')

# Generate manifest
subprocess.run(['sha256sum', 'data.txt', 'metadata.txt'], 
               stdout=open('manifest.sha256', 'w'))

# Verify baseline is valid
result = subprocess.run(['python', '../verify.py'], capture_output=True, text=True)
assert 'VALID' in result.stdout

# Attack: alter file
with open('data.txt', 'w') as f:
    f.write('Tampered content')

# Verify now fails
result = subprocess.run(['python', '../verify.py'], capture_output=True, text=True)
assert 'INVALID' in result.stdout
print("Test case 1 PASSED: File alteration detected")
```

### Test case 2: Detect manifest substitution (requires external reference)

```python
# Archive original manifest hash
with open('manifest.sha256', 'rb') as f:
    original_manifest_hash = hashlib.sha256(f.read()).hexdigest()

print(f"Original manifest hash: {original_manifest_hash}")

# Attack: alter file and recompute manifest
with open('data.txt', 'w') as f:
    f.write('Tampered content v2')
subprocess.run(['sha256sum', 'data.txt', 'metadata.txt'], 
               stdout=open('manifest.sha256', 'w'))

# Compute new manifest hash
with open('manifest.sha256', 'rb') as f:
    new_manifest_hash = hashlib.sha256(f.read()).hexdigest()

print(f"New manifest hash: {new_manifest_hash}")

# Verify the manifest itself has changed
assert original_manifest_hash != new_manifest_hash
print("Test case 2 PASSED: Manifest substitution detected via external reference")
```

---

## Lessons

1. **The verifier detects file alteration if the manifest is unchanged.**  
   This is the baseline defense: any tampering that doesn't update the manifest fails verification.

2. **The verifier cannot detect manifest substitution without an external reference.**  
   If an attacker can recompute and replace the manifest, the verifier will report VALID for the *new* bundle. External archiving (timestamp, blockchain, Git) is required to detect this.

3. **Future hardening: Merkle-tree extension.**  
   Publish a root hash externally. Even if the attacker recomputes the manifest, they cannot recompute the externally published root without detection.

---

## Next steps

- Implement `verify.py` --check-manifest-hash <expected-hash>` flag.
- Add test suite (`tests/attack_001_test.py`) that runs both variants.
- Document external archiving best practices in `HARDENING.md`.
- Design Attack Bundle 002: File injection (adding files not in the manifest).

---

## References

- NIST FIPS 180-4 (SHA-256 specification)
- Merkle, R. C. (1988). "A Digital Signature Based on a Conventional Encryption Function."
- ProofBundles CORE.md (artifact closure primitive)

---

## Status

**Attack Bundle 001:** Designed  
**Implementation:** Pending  
**Test suite:** Pending  
**Hardening:** Pending (external manifest anchoring)
