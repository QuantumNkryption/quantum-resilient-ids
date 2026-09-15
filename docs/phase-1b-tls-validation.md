# Phase 1B — TLS Handshake, PCAP & Reproducibility Validation

## Objective

Validate that the qualified Phase 1A cryptographic stack can perform the intended TLS operations, that negotiation can be independently observed in captured traffic, and that the environment can be destroyed and rebuilt without changing the result.

## Acceptance sequence

1. Generate suitable test credentials inside the qualified environment.
2. Establish a classical TLS baseline.
3. Establish a hybrid TLS session using the qualified hybrid group (including `X25519MLKEM768`).
4. Establish the PQC-oriented configuration supported by the qualified stack.
5. Capture each handshake using packet-capture tooling.
6. Inspect captures independently with `tshark` / Wireshark and, where useful, Zeek.
7. Record negotiated protocol, cipher suite, key-exchange/group and certificate/signature metadata that can be established from the client/server output or capture.
8. Preserve sanitized command output and capture-derived summaries.
9. Destroy the qualified environment.
10. Rebuild from the documented versions/configuration and reproduce the acceptance results.

## Verified evidence to date

Phase 1B has verified generation of a self-signed ML-DSA-65 server certificate with:

- Subject: `CN=oqs-phase1b-server`
- Issuer: `CN=oqs-phase1b-server`
- Signature algorithm: `mldsa65`
- Public-key algorithm: `mldsa65`
- Serial: `47FC538165C7C0E9596443BE2027FFCE96A226DA`
- Validity: 2026-09-15 01:36:49 UTC through 2026-10-15 01:36:49 UTC

A sanitized summary is stored at `artifacts/sanitized/phase1b_mldsa65_certificate_summary.txt`.

## Current status

**In progress.** Certificate generation is evidence that the qualified provider can perform an ML-DSA-65 certificate operation. It is not sufficient evidence that the required TLS handshakes, PCAP negotiation verification or destroy/rebuild/reproduce gate have all passed.

## Evidence required before Phase 1B is marked complete

For each required cryptographic regime, preserve:

- run ID and UTC timestamp;
- exact server/client invocation or scripted equivalent;
- server certificate/public certificate summary (never the private key);
- negotiated TLS version and cipher suite;
- negotiated group / key-establishment evidence where observable;
- certificate/signature algorithm evidence;
- sanitized PCAP-derived handshake summary;
- capture SHA-256 retained in the private research evidence store;
- tool versions;
- pass/fail result and exclusion reason if failed.

Then repeat the acceptance suite after a clean rebuild.

## Publication rule

No numerical handshake-size or latency claim should enter the canonical results table until the corresponding raw capture exists, its provenance is recorded and the extraction procedure is reproducible.
