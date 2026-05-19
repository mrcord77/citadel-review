# Validation Evidence

All evidence was collected on May 19, 2026.

## Test Suite

190 tests across 4 crates. Zero failures on both build environments.

| Crate | Tests | Coverage |
|-------|-------|---------|
| citadel-envelope | 99 | Primitives, KATs, adversarial, streaming, wire format |
| citadel-keystore | 58 | Lifecycle, hierarchy, replay, policy, threat, properties, concurrency |
| citadel-api | 21 | HTTP integration, scoped auth, rate limiting |
| citadel-ffi | 12 | C, Python, Java bindings |

Build environments:
- Linux: rustc 1.75.0
- Windows: rustc 1.95.0 MSVC (AMD Ryzen 9700X)

## NIST ACVP Vector Validation

Source: https://github.com/usnistgov/ACVP-Server (ML-KEM-keyGen-FIPS203, ML-KEM-encapDecap-FIPS203)

| Operation | Vectors | Result |
|-----------|---------|--------|
| KeyGen (d,z → ek,dk) | 25 | 25/25 byte-identical to NIST reference |
| Encapsulation (ek,m → c,k) | 25 | 25/25 byte-identical to NIST reference |
| Total | 50 | 50/50 passed |

Method: deterministic RNG injection matching NIST-provided seeds.
Output compared byte-for-byte against NIST expected results.

## Adversarial Testing

| Category | Coverage | Result |
|----------|---------|--------|
| Bit-flip integrity | 9,264 bit positions | 0 undetected modifications |
| Byte-level auth | 1,183 ciphertext bytes | 0 malleabilities |
| Nonce uniqueness | 10,000 encryptions | 0 collisions |
| Panic safety | Arbitrary inputs to all parsers | 0 panics |
| Fuzz testing | 3 harness targets | 0 crashes, 0 false accepts |

## Property-Based Testing (proptest)

5 replay governance invariants verified across random inputs:

1. Fresh claim → always accepted
2. Duplicate claim → always rejected
3. Domain isolation → cross-domain claims independent
4. Release-and-reclaim → released slots reclaimable
5. No false rejections → N distinct IDs all accepted

3 hierarchy role tests:

1. All 5 valid parent-child relationships accepted
2. All 11 invalid parent-child relationships rejected
3. All 6 roles have defined leaf/parent behavior

## Concurrency Stress Testing

| Test | Setup | Result |
|------|-------|--------|
| Exactly-one-wins | 100 threads, same replay key | 1 accepted, 99 rejected, 0 errors |
| Distinct claims | 200 threads, unique keys | 0 false rejections |
| Claim-release churn | 10,000 cycles | No corruption, reclaim verified |
| Mixed traffic | 50 fresh + 50 replay | 50 accepted, 50 rejected |

## Bare-Metal Benchmarks

Hardware: AMD Ryzen 9700X (8 cores), Windows 11, High Performance power plan
Toolchain: rustc 1.95.0 MSVC
Crate: citadel-envelope with ml-kem 0.2.2
N=100,000, warmup=10,000

| Operation | Mean (µs) | Std (µs) | p50 (µs) | p95 (µs) |
|-----------|-----------|----------|----------|----------|
| X25519 keygen | 9.38 | 4.06 | 9.20 | 9.30 |
| X25519 DH | 28.33 | 2.04 | 28.20 | 28.40 |
| ML-KEM-768 keygen | 27.08 | 0.67 | 27.00 | 27.80 |
| ML-KEM-768 encaps | 24.93 | 1.12 | 24.70 | 26.10 |
| Hybrid keygen | 36.50 | 1.40 | 36.20 | 38.60 |
| Hybrid encrypt 1KB | 65.33 | 7.71 | 64.70 | 67.80 |
| Hybrid decrypt 1KB | 62.98 | 2.12 | 62.50 | 65.40 |

Internal consistency: hybrid keygen (36.5) = X25519 keygen (9.4) + ML-KEM keygen (27.1)

## Comparative Analysis

N=50,000, 1 KB payload, same hardware

| Configuration | Encrypt (µs) | Overhead (bytes) | Security |
|--------------|-------------|-----------------|----------|
| X25519 + AES-256-GCM | 38.6 | 60 | Classical (128-bit) |
| ML-KEM-768 + AES-256-GCM | 25.6 | 1,116 | PQ (~192-bit) |
| Citadel hybrid | 65.2 | 1,154 | Hybrid (either-or) |

Hybrid cost: +27 µs latency, +1,094 bytes vs classical-only.

## Supply Chain

- cargo-audit: 278 crates scanned, 0 production advisories
  (1 medium advisory in dev-only rsa crate, not used at runtime)
- cargo-deny: all dependency licenses permissive (MIT, Apache-2.0, ISC),
  compatible with AGPL-3.0 distribution
- Full dependency tree archived in dependency_tree.txt
