# Phase 1B TLS Reproducibility Summary

## Supported test IDs

- `P1B-TLS-HYBRID-001`
- `P1B-PCAP-001`
- `P1B-REBUILD-001`

## Public artifact scope

This document records the sanitized public acceptance summary for Phase 1B. It intentionally omits private keys, raw packet captures, credentials and other sensitive runtime material.

## Acceptance result

**PASS — Phase 1B complete.**

Two independent acceptance runs were completed. The second run followed a clean destroy/rebuild of the qualified OQS/OpenSSL environment.

## Reproduced properties

Both accepted runs established the same target cryptographic behavior:

- TLS protocol: TLS 1.3
- Hybrid key-establishment group: `X25519MLKEM768`
- Observed group identifier: `0x11ec`
- Authentication/certificate algorithm: ML-DSA-65 (`mldsa65`)
- Packet count per accepted capture: 23
- Handshake sequence: consistent across Run 1 and Run 2
- Rebuild condition: Run 2 executed after clean environment destruction and rebuild
- Test artifacts: freshly generated for the rebuilt run

## Interpretation

This artifact supports the claim that the qualified environment reproducibly generated and captured the target TLS 1.3 handshake using `X25519MLKEM768` and ML-DSA-65 authentication.

It does **not** establish comparative classical-vs-hybrid-vs-PQC handshake size, latency, IDS performance or machine-learning generalization. Those comparisons belong to Phase 1C and later phases.

## Evidence handling

Raw captures and private-key material are retained outside the public repository. The public repository contains only sanitized evidence and research metadata suitable for release.
