# Review Scope

## Public review (no NDA required)

Available to any reviewer via public repository and this package.

Materials:
- This review package (all files)
- Paper manuscript
- citadel-envelope source: https://github.com/mrcord77/rust_citadel
- Benchmark and test result artifacts
- Architecture and threat model documentation

Covers:
- Hybrid construction correctness (X25519 + ML-KEM-768 + HKDF + AES-256-GCM)
- Wire format design
- ACVP vector conformance
- Benchmark methodology
- Security argument review
- Threat model completeness
- Paper technical accuracy

## Confidential review (NDA required)

Available to engaged audit firms and academic collaborators under NDA.

Additional materials:
- Full workspace source (all 7 crates)
- Cargo.lock with exact dependency versions
- Tagged commit for reproducibility
- Internal test harnesses
- Deployment configuration

Covers (in addition to public scope):
- Replay governance internals and store implementations
- Capability token generation and validation logic
- Adaptive threat scoring formulas and thresholds
- Key hierarchy enforcement implementation
- KDF domain separation constants
- Wire format byte-level layout
- API authentication and rate limiting
- State machine transitions

## Engagement terms

Public review: open, no restrictions. Attribution appreciated.

Confidential review: standard mutual NDA. Findings may be referenced
in the paper as "independently reviewed by [firm/researcher]" with
reviewer approval.

For confidential review access, contact the author with:
- Reviewer credentials / firm name
- Intended scope (which crates / which questions)
- Timeline
