# Architecture Overview

## Hybrid Envelope (citadel-envelope)

The cryptographic core. Combines X25519 ephemeral Diffie-Hellman with
ML-KEM-768 key encapsulation. Both produce 32-byte shared secrets,
concatenated into 64 bytes of HKDF-SHA256 input key material with
domain-separated context binding. Output: 32-byte AES-256-GCM key.

Key sizes:
- Hybrid public key: 1,216 bytes (32 X25519 + 1,184 ML-KEM)
- Hybrid private key: 2,432 bytes (32 X25519 + 2,400 ML-KEM)
- Fixed ciphertext overhead: 1,154 bytes per record
- Wire format: self-describing with version/suite header

Security property: confidentiality holds if either X25519 OR ML-KEM
remains secure. Forward secrecy through ephemeral X25519 keygen per
operation.

Dependencies: ml-kem 0.2.3, x25519-dalek 2, aes-gcm 0.10, hkdf 0.12,
zeroize 1.7, rand_core 0.6

## Key Hierarchy (citadel-keystore)

Four levels aligned with NIST SP 800-57:

```
Root (offline, protects hierarchy)
  └── Domain (per-tenant / per-environment)
       └── KEK (key-encrypting key, wraps DEKs)
            └── DEK (data-encrypting key, encrypts user data)
```

KEK and DEK secrets are sealed under the parent's hybrid public key.
The hierarchy itself is post-quantum protected, not just the data layer.

Role validation is runtime-enforced at key generation time (not
compile-time). A DEK cannot be created under a Root key; the system
returns a structured error.

Key re-wrapping allows DEK migration after KEK revocation without
re-encrypting application data.

## Replay Governance (citadel-keystore)

Every decryption is gated by an atomic replay claim. Properties:

- Fail-closed: if the replay store is unavailable, decryption is denied
- Atomic: single lock acquisition prevents TOCTOU races
- Domain-scoped: claims in domain A cannot interfere with domain B
- Release on decrypt failure: prevents ciphertext-poisoning attacks

Backends: MemoryReplayStore (single instance), FileReplayStore,
RedisReplayStore (distributed).

Fail-closed holds for all backends including Redis: connection failure
produces denial, not unprotected access.

## Capability Authorization (citadel-core)

Two-layer model:

Layer 1 (StateEnforcer): issues cryptographically generated tokens
after checking key state, domain membership, and operation type.

Layer 2 (Keystore): re-validates each token at execution time against
the issuing instance. Tokens from a restarted instance are rejected.

Fail-closed: without a bound StateEnforcer, all operations error.

## Adaptive Threat Response (citadel-keystore)

Maps security events to a rolling threat score across five levels
(Low through Critical). As the score rises:
- Rotation intervals shorten
- Usage caps tighten
- Grace periods shrink

## Signing (citadel-signer)

ML-DSA-65 (FIPS 204) signing, structurally isolated from the KEM layer.
No key types shared between citadel-signer and citadel-envelope.

## Type Safety

Rust compile-time: KEM keys and signing keys are distinct types.
Algorithm-level key confusion is a compile error.

Rust runtime: hierarchy parent-child role constraints validated during
key generation.

Sensitive material wrapped in Zeroizing<T> for deterministic cleanup.
