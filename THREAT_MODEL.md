# Threat Model

## In-scope adversaries

| Adversary | Capability | Citadel response |
|-----------|-----------|-----------------|
| Passive observer | Records all ciphertext | Hybrid envelope: confidentiality requires breaking both X25519 and ML-KEM |
| Active attacker | Injects, replays, modifies traffic | AES-256-GCM authentication + atomic replay governance |
| CRQC adversary | Shor's algorithm on ECC | ML-KEM-768 component survives; session key protected |
| HNDL adversary | Archives traffic for future quantum decryption | ML-KEM provides forward protection for archived data |

## Out-of-scope (acknowledged non-goals)

| Threat | Why excluded | Risk level |
|--------|-------------|-----------|
| Physical server access | Requires separate physical security | High if applicable |
| OS / hypervisor compromise | Kernel-level access bypasses all user-space crypto | High if applicable |
| Master key out-of-band exposure | Social engineering / insider threat | High if applicable |
| Hardware side channels (timing, power, cache, EM) | Requires specialized equipment and physical proximity | Medium |
| Supply chain compromise | Dependency injection, compiler tampering | Medium — partially mitigated by cargo-audit |
| Malicious dependency insertion | Trojan crates | Medium — partially mitigated by cargo-audit |

## Key security boundaries

### Boundary 1: Hybrid envelope
- X25519 failure alone does not break confidentiality
- ML-KEM failure alone does not break confidentiality
- Both must fail for an adversary to recover plaintext
- HKDF-SHA256 domain separation prevents cross-context key reuse

### Boundary 2: Replay store
- Fail-closed: store outage = denial, not bypass
- Atomic claim: TOCTOU prevention through single lock acquisition
- Domain isolation: cross-domain claims impossible
- Release semantics: failed decryption releases the claim (prevents ciphertext poisoning)

### Boundary 3: Capability enforcement
- Dual-layer: issuance and validation are separate steps
- Instance-bound: tokens from restarted instances are rejected
- Fail-closed: missing enforcer = all operations denied

### Boundary 4: Key hierarchy
- Role constraints: DEK cannot be created under Root (runtime validation)
- Domain boundaries: keys never traverse domains
- Re-wrapping: KEK revocation cascades to DEKs but does not require re-encryption

## Known weaknesses (honest disclosure)

1. No independent security audit has been conducted
2. ml-kem v0.2.3 carries "experimental" designation
3. ml-dsa v0.1.0-rc.9 is a release candidate
4. No HSM integration — master key in process memory
5. Side-channel analysis has not been performed
6. Single-node validation only — no cluster testing
7. Transport layer (TLS) is not PQ-protected
8. No formal proofs — properties verified by testing, not theorem proving

## Questions for reviewers

See REVIEW_QUESTIONS.md for specific areas where feedback is requested.
