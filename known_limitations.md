# Known Limitations

Honest assessment of gaps as of May 19, 2026.

## Critical gaps (block production deployment)

**No independent security audit.** No external cryptographer or security
firm has reviewed the system. This is the primary deployment blocker.
All other evidence is self-generated.

## Significant gaps

**ml-kem v0.2.3 is experimental.** The crate has not received independent
security audit. ACVP vector execution (50/50 passed) confirms algorithmic
correctness but not implementation security (side channels, memory safety
in unsafe blocks, etc.). The hybrid construction mitigates this: X25519
provides a classical security floor independent of any ML-KEM flaw.

**ml-dsa v0.1.0-rc.9 is a release candidate.** The signing crate is
pre-stable. Version pinned; upgrade gated on explicit review.

**No side-channel analysis.** No timing, power, cache, or electromagnetic
side-channel testing has been performed. The ml-kem crate claims
constant-time operations but this has not been independently verified
for our usage.

**No HSM integration.** Master key material exists in process memory.
TPM/HSM integration is a planned future feature.

## Moderate gaps

**Single-node validation only.** The distributed Redis replay backend
has not been tested under network partition or split-brain conditions.
Multi-node cluster validation has not been performed.

**No transport-layer PQ protection.** The hybrid envelope operates at
the application layer. TLS connections to the API remain classically
protected. Post-quantum TLS integration is future work.

**No formal proofs.** Security properties are verified by testing
(190 tests, property-based proofs, stress tests) but not by formal
theorem proving or model checking.

**Supply chain partially addressed.** cargo-audit and cargo-deny pass,
but reproducible builds and build attestation are not yet implemented.

## Minor gaps

**Single hardware platform benchmarked.** Bare-metal measurements are
from AMD Ryzen 9700X / Windows 11 only. Linux and ARM measurements
are future work.

**ACVP vectors executed but not officially submitted.** 50 NIST vectors
passed byte-identical, but the system has not completed the formal
NIST ACVP submission process.

## What is NOT a gap

These items are sometimes assumed to be problems but are intentionally
out of scope or adequately addressed:

- Classical cryptographic confidence: X25519, AES-256-GCM, HKDF-SHA256,
  and SHA-3 are well-studied and standardized. No novel primitives.
- Algorithm selection: ML-KEM-768 is the NIST-specified parameter set
  for CNSA 2.0 software applications.
- Test coverage: 190 tests, 50 ACVP vectors, property proofs,
  concurrency stress, adversarial testing, fuzz testing.
- License compliance: all dependencies use permissive licenses.
- Replay governance: fail-closed, atomic, domain-scoped, property-tested,
  stress-tested under 100-thread contention.
