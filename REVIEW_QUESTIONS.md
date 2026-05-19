# Review Questions

We are requesting feedback on specific technical areas. General impressions
are welcome, but targeted responses to these questions would be most valuable.

## Hybrid construction

1. Is the HKDF-SHA256 derivation from concatenated X25519 and ML-KEM shared
   secrets sound? Are there known attacks on this composition pattern?

2. Does the domain separation in the HKDF info string provide adequate
   context binding, or are there cross-protocol attack scenarios?

3. The security argument claims that compromise of either component alone
   does not reduce security below the surviving component. Are there
   interaction effects we have not considered?

## ML-KEM implementation

4. The ml-kem crate (v0.2.3) carries an "experimental" designation. Are
   there known issues with this specific version that affect our use case?

5. Our ACVP vector execution passed 50/50, but official ACVP submission
   has not been completed. Are there additional conformance concerns
   beyond vector matching?

## Replay governance

6. The fail-closed design denies decryption when the replay store is
   unavailable. Is this the correct security posture, or does it create
   denial-of-service vulnerabilities that an attacker could exploit?

7. The release-on-decrypt-failure mechanism prevents ciphertext poisoning.
   Are there race conditions or timing attacks that could bypass this?

## Key hierarchy

8. KEK and DEK secrets are sealed under the parent's hybrid public key.
   Does this create any circular dependency or key-wrapping depth concerns?

9. The re-wrapping mechanism allows DEK migration without re-encrypting
   data. Are there key-commitment or rollback attacks in this design?

## Operational governance

10. The adaptive threat response tightens policies based on a rolling
    event score. Could an attacker manipulate the scoring to cause
    either false calm (suppressing response) or denial of service
    (triggering maximum lockdown)?

11. Capability tokens are instance-bound and rejected after restart.
    Is the token generation entropy sufficient? Are there replay or
    forgery scenarios?

## Side channels

12. Are there timing-observable differences in the hybrid encrypt/decrypt
    path that could leak information about key material?

13. The ml-kem crate claims constant-time operations. Has this been
    independently verified for the version we use?

## General

14. What is the most likely attack vector against this system as designed?

15. What single change would most improve the security posture?
