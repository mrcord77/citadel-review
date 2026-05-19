# Citadel v3 — Review Package

Prepared: May 19, 2026
Author: Andre Cordero
Repository: https://github.com/mrcord77/rust_citadel (public core)
Licensing:
  Public core (citadel-envelope): AGPL-3.0-or-later
  Operational layer (6 crates):    Proprietary, not included in this package

## What is Citadel

A self-hosted hybrid post-quantum key management system in Rust combining
X25519 (RFC 7748) with ML-KEM-768 (FIPS 203) via HKDF-SHA256 and AES-256-GCM.
Designed for quantum-transition key lifecycle management.

## Current status

- Evaluation-stage prototype
- 190 tests, 0 failures (Linux rustc 1.75.0, Windows rustc 1.95.0 MSVC)
- 50 NIST ACVP ML-KEM-768 vectors executed, 0 failures
- Bare-metal benchmarks on dedicated AMD Ryzen 9700X hardware
- Dependency audit: 0 production advisories (278 crates scanned)
- License compliance: all dependencies permissive, AGPL-compatible
- NOT independently audited
- NOT FIPS 140-3 certified
- NOT recommended for production without independent review

## What we are asking for

Technical review of the hybrid construction, operational governance model,
and implementation. See REVIEW_QUESTIONS.md for 15 specific areas where
feedback is requested.

## Package contents

```
README.md                  This file
ARCHITECTURE.md            System design: envelope, hierarchy, replay, caps, policy
THREAT_MODEL.md            Adversaries, security boundaries, honest weaknesses
VALIDATION.md              Complete evidence summary with all results
REVIEW_QUESTIONS.md        15 specific technical questions for reviewers
SCOPE.md                   Public vs confidential review boundaries
COMMANDS.md                Exact commands to reproduce all evidence
known_limitations.md       Honest gaps and future work

paper/
  Citadel_Hybrid_PQ_Paper_v14_FINAL.docx

benchmarks/
  benchmark_results.txt    Bare-metal, N=100,000 (AMD Ryzen 9700X)
  comparative_results.txt  X25519 vs ML-KEM vs Hybrid side-by-side

acvp/
  acvp_results.txt         50 NIST ACVP ML-KEM-768 vector results

test_results/
  property_results.txt     proptest replay + hierarchy proofs (8/8 passed)
  concurrency_results.txt  Multi-threaded stress tests (4/4 passed)
  supply_chain_audit.txt   cargo-audit report
  license_check.txt        cargo-deny license compliance
```

## Public / proprietary boundary

| Crate | Status | Role |
|-------|--------|------|
| citadel-envelope | Public | Hybrid KEM, HKDF, AES-256-GCM, wire format |
| citadel-keystore | Proprietary | Key lifecycle, hierarchy, replay, adaptive policy |
| citadel-core | Proprietary | Capability tokens, two-layer enforcement |
| citadel-api | Proprietary | HTTP interface, auth, rate limiting |
| citadel-signer | Proprietary | ML-DSA-65 signing (isolated from KEM) |
| citadel-ffi | Proprietary | C/Python/Java bindings |
| citadel-cli | Proprietary | Command-line interface |

Source access to proprietary crates is available under NDA for scoped
audit engagements. See SCOPE.md for details.
