# Phase 1A — OQS / OpenSSL Qualification

## Objective

Build and qualify a reproducible post-quantum-capable TLS environment without yet treating any traffic measurements as scientific results.

## Qualified environment

The Phase 1A environment was built as an isolated/containerized cryptographic stack so it would not depend on the host OpenSSL installation.

| Component | Qualified value |
|---|---|
| Guest OS | Ubuntu 24.04.4 LTS |
| OpenSSL | 3.4.7 |
| oqs-provider | 0.11.0 |
| liboqs | 0.15.0 |
| Deployment pattern | Containerized / isolated from host OpenSSL |

## Qualification evidence

Phase 1A established that:

- the pinned stack can be built cleanly;
- the OQS provider is active in the qualified environment;
- ML-KEM algorithms are exposed;
- hybrid `X25519MLKEM768` is exposed;
- ML-DSA algorithms are exposed through the provider;
- the research stack remains distinct from the native host OpenSSL installation.

## Why this phase matters

The later scientific experiment compares cryptographic regimes. If the cryptographic implementation itself is not version-pinned and independently qualified, any observed traffic difference could be confounded by toolchain drift or host configuration.

Phase 1A therefore treats the cryptographic stack as experimental instrumentation: it must be qualified before it is used to generate evidence.

## Non-claims

Phase 1A does **not** by itself establish:

- successful classical, hybrid and PQC TLS handshakes;
- independently verified negotiated groups/signature algorithms from packet captures;
- handshake-size measurements;
- IDS performance differences;
- machine-learning results.

Those belong to later acceptance gates.

## Exit criterion

Phase 1A is complete once the pinned OQS/OpenSSL environment builds reproducibly and exposes the required algorithm families. TLS/PCAP/rebuild validation is Phase 1B.
