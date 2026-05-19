# Audit Request

## Summary

Independent technical review is requested for Citadel, a Rust-based
hybrid post-quantum key management system. The system combines X25519
with ML-KEM-768 in an operational governance architecture designed for
quantum-transition key lifecycle management.

The system is at evaluation stage. This review is sought prior to any
production deployment recommendation.

## What has already been done

- 190 automated tests, 0 failures (2 build environments)
- 50 NIST ACVP ML-KEM-768 vectors executed, byte-identical
- 9,264-position bit-flip adversarial testing
- Property-based replay governance proofs (proptest)
- 100-thread concurrency stress testing
- Bare-metal benchmarks on dedicated hardware (N=100,000)
- Comparative analysis against X25519-only and ML-KEM-only configurations
- cargo-audit: 0 production advisories
- cargo-deny: license compliance confirmed

## What has NOT been done

- Independent security audit (this request)
- Side-channel analysis
- Formal verification / theorem proving
- HSM integration testing
- Network partition testing of distributed replay backend
- Multi-node cluster validation
- FIPS 140-3 evaluation

## Specific areas for review

Priority order:

1. **Hybrid composition correctness.** Is the HKDF-SHA256 derivation
   from concatenated X25519 and ML-KEM shared secrets sound?

2. **Replay governance attack surface.** Can the fail-closed atomic
   replay model be bypassed through race conditions, timing attacks,
   or ciphertext poisoning?

3. **HKDF domain separation.** Does the context binding prevent
   cross-protocol or cross-application key reuse?

4. **Side-channel exposure.** Are there timing-observable differences
   in the encrypt/decrypt path? Is ml-kem 0.2.3 constant-time?

5. **Key hierarchy design.** Are there circular dependency, key
   commitment, or rollback attacks in the re-wrapping mechanism?

6. **Adaptive policy manipulation.** Can an attacker manipulate threat
   scoring to suppress response or trigger denial of service?

7. **Misuse resistance.** What happens when the API is used incorrectly?
   Where are the sharpest edges?

## Deliverable requested

A written technical assessment covering:
- Findings (critical / high / medium / low)
- Specific code locations where applicable
- Recommended mitigations
- Overall security posture assessment
- Production readiness opinion

## Materials provided in this package

Public (no restrictions):
- Paper manuscript (v14)
- citadel-envelope source (public GitHub)
- NIST ACVP vector execution results
- Bare-metal benchmark artifacts
- Comparative analysis results
- Property and concurrency test results
- Supply chain audit and license compliance
- Architecture, threat model, and limitation documentation

Not included (available under NDA):
- citadel-keystore (replay internals, hierarchy enforcement)
- citadel-core (capability token implementation)
- citadel-api (authentication, rate limiting)
- citadel-signer (ML-DSA-65 signing)
- citadel-ffi, citadel-cli
- Adaptive threat policy scoring logic
- KDF domain separation constants
- Wire format byte-level specification

## Access

Public core: https://github.com/mrcord77/rust_citadel
Full workspace: available under NDA (see SCOPE.md)

## Contact

Andre Cordero
Independent Research
