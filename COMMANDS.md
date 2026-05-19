# Reproduction Commands

All commands below were executed on May 19, 2026.
Hardware: AMD Ryzen 9700X (8 cores), Windows 11, 64 GB RAM
Toolchain: rustc 1.95.0 (59807616e 2026-04-14) MSVC
Power plan: High Performance (fixed frequency)

## Prerequisites

```
rustc --version          # 1.81+ required; 1.95.0 used
cargo --version
cargo install cargo-audit
cargo install cargo-deny
```

## Test suite (190 tests)

```
cargo test --workspace
```

Expected: 190 passed, 0 failed across citadel-envelope (99),
citadel-keystore (58), citadel-api (21), citadel-ffi (12).

## Bare-metal benchmarks (N=100,000)

```powershell
# Set high performance power plan
powercfg /setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c

# Close all background applications

cargo build --release -p citadel-envelope --bin benchmark_baremetal
.\target\release\benchmark_baremetal.exe
```

## Comparative benchmarks (N=50,000)

```powershell
cargo build --release -p citadel-envelope --bin comparative_bench
.\target\release\comparative_bench.exe
```

## Property-based tests (proptest)

```
cargo test -p citadel-keystore --test property_tests -- --nocapture
```

Tests 5 replay governance invariants + 3 hierarchy role properties.

## Concurrency stress tests

```
cargo test -p citadel-keystore --test concurrency_stress -- --nocapture
```

Tests: 100-thread contention, 200-thread distinct claims,
10,000-cycle churn, mixed fresh/replay traffic.

## NIST ACVP vector validation

```powershell
# Download vectors
mkdir citadel-envelope\acvp-vectors
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/usnistgov/ACVP-Server/master/gen-val/json-files/ML-KEM-keyGen-FIPS203/internalProjection.json" -OutFile "citadel-envelope\acvp-vectors\keygen.json"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/usnistgov/ACVP-Server/master/gen-val/json-files/ML-KEM-encapDecap-FIPS203/internalProjection.json" -OutFile "citadel-envelope\acvp-vectors\encapdecap.json"

# Run
cargo test -p citadel-envelope --test acvp_vectors -- --nocapture
```

Expected: 25/25 keygen passed, 25/25 encapsulation passed.

## Supply chain audit

```
cargo audit
```

Expected: 0 production advisories. One medium advisory in dev-only
rsa crate (RUSTSEC-2023-0071, not used at runtime).

## License compliance

```
cargo deny check licenses
```

Requires deny.toml in workspace root:

```toml
[licenses]
allow = ["MIT", "Apache-2.0", "AGPL-3.0-or-later", "BSD-2-Clause", "BSD-3-Clause", "ISC", "Unicode-3.0", "Unicode-DFS-2016", "OpenSSL"]
confidence-threshold = 0.8
```

Expected: licenses ok.
